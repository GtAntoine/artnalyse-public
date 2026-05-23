# 04 — Architecture Technique : Artnalyse

## Vue d'Ensemble

Artnalyse suit une architecture **client-lourd / serverless**. La logique métier sensible (clés API, IA) vit dans des Supabase Edge Functions (Deno). Le client React Native orchestre l'UX et le streaming.

```
┌─────────────────────────────────────────────────────────┐
│                    Client iOS (React Native)              │
│                                                           │
│  CaptureScreen → useStreamingAnalysis → ResultScreen     │
│       ↑                   ↑                  ↓           │
│  expo-camera          XMLHttpRequest     expo-audio       │
│  expo-image-picker    streaming NDJSON   AudioPlayer      │
└────────────────────────────┬────────────────────────────┘
                             │ HTTPS
┌────────────────────────────▼────────────────────────────┐
│               Supabase Edge Functions (Deno)             │
│                                                           │
│  artwork-analysis-stream      openai-tts-stream          │
│  ├── SerpAPI Google Lens      └── OpenAI TTS API          │
│  └── OpenAI GPT-5 Vision                                  │
└────────────────────────────────────────────────────────┘
                             │
┌────────────────────────────▼────────────────────────────┐
│                    Supabase Storage                       │
│           (images temporaires pour Google Lens)          │
└─────────────────────────────────────────────────────────┘
```

## Choix Techniques Majeurs

### 1. Supabase Edge Functions vs API Routes Next.js

**Choix** : Supabase Edge Functions (Deno)

**Raison** :
- Proche géographiquement des APIs OpenAI/SerpAPI → latence réduite
- Deno natif → streaming HTTP facilité
- Intégration Supabase Auth + Storage sans configuration supplémentaire
- Cold start < 50ms vs ~200ms pour Lambda@Edge

### 2. XMLHttpRequest vs fetch pour le streaming

**Choix** : XMLHttpRequest

**Raison** :
- React Native (au moment du développement) ne supportait pas `ReadableStream` sur fetch
- `xhr.onprogress` permet la lecture progressive de `responseText`
- Compatible Android et iOS sans polyfill

```typescript
// Pattern adopté
xhr.onprogress = () => {
  const chunk = xhr.responseText.slice(lastProcessedLength);
  lastProcessedLength = xhr.responseText.length;
  processNdjsonChunk(chunk);
};
```

### 3. Zustand vs Redux pour le state management

**Choix** : Zustand

**Raison** :
- API minimaliste → moins de boilerplate
- Persistance AsyncStorage triviale avec `zustand/middleware`
- Compatible TypeScript sans configuration complexe
- Performance : subscriptions granulaires (pas de re-renders inutiles)

### 4. expo-audio vs expo-av

**Choix** : expo-audio (nouveau package Expo)

**Raison** :
- expo-av est déprécié (legacy)
- expo-audio offre une API plus claire pour les use cases simples
- Support natif des contrôles système iOS (AirPlay, Control Center)

### 5. NDJSON vs JSON complet vs SSE

**Choix** : NDJSON (Newline Delimited JSON)

**Raison** :
- SSE (Server-Sent Events) : mal supporté dans React Native sans polyfill
- JSON complet : pas de streaming possible, UX dégradée (attente 10s+)
- NDJSON : compatible XMLHttpRequest, parsable ligne par ligne, format simple

```
// SSE format (écarté)
data: {"type": "section", "content": "..."}

// NDJSON format (adopté)
{"type": "identification", "data": {...}}\n
{"type": "narrative", "data": {...}}\n
```

## Navigation et Routing

```typescript
// src/navigation/RootNavigator.tsx

type RootStackParamList = {
  Home: undefined;
  Capture: undefined;
  Result: {
    resultId: string;
    imageBase64: string;
    imageUri: string;
    artistName?: string;
    artworkTitle?: string;
    cartelBase64?: string;
  };
  History: undefined;
  Settings: undefined;
};
```

**Note importante** : Il n'existe pas de AnalysisScreen séparé. `ResultScreen` gère à la fois le streaming en cours (avec indicateurs de progression) et l'affichage final. Ce choix simplifie la navigation et évite une transition qui briserait le flux immersif.

## State Management — Détail

```typescript
// src/stores/app-store.ts

interface AppStore {
  // Historique (max 100 items, LIFO)
  history: HistoryItem[];
  addToHistory: (item: HistoryItem) => void;
  removeFromHistory: (id: string) => void;
  toggleFavorite: (id: string) => void;
  updateAudioPath: (id: string, path: string) => void;
  clearHistory: () => void;

  // Préférences
  hapticsEnabled: boolean;
  toggleHaptics: () => void;

  // Onboarding
  hasSeenOnboarding: boolean;
  setHasSeenOnboarding: () => void;
}

// Persisté dans AsyncStorage 'artnalyse-storage-v2'
// Clé versionnée pour permettre des migrations futures
```

## Sécurité

### Secrets côté serveur uniquement

```
Variables d'environnement (Supabase Edge Functions) :
├── OPENAI_API_KEY      → Jamais exposé côté client
├── SERPAPI_KEY         → Jamais exposé côté client
└── SUPABASE_SERVICE_KEY → Jamais exposé côté client

Variables côté client (préfixe EXPO_PUBLIC_) :
├── EXPO_PUBLIC_SUPABASE_URL     → Domaine public Supabase
└── EXPO_PUBLIC_SUPABASE_ANON_KEY → Clé anonyme (limitée par RLS)
```

### Rate limiting (prévu V2)

- Supabase Auth → limite d'appels par utilisateur authentifié
- Freemium : quota 5 analyses/mois géré côté Edge Function

## Publication App Store

### Expo EAS (Expo Application Services)

```json
// eas.json
{
  "build": {
    "production": {
      "ios": {
        "resourceClass": "m-medium",
        "distribution": "store"
      }
    }
  }
}
```

### Configuration iOS spécifique

```javascript
// app.config.js
ios: {
  bundleIdentifier: 'com.artnalyse.app',
  infoPlist: {
    ITSAppUsesNonExemptEncryption: false,     // Validation App Store simplifiée
    CFBundleLocalizations: ['fr', 'en'],
    NSCameraUsageDescription: 'Artnalyse utilise la caméra pour photographier les œuvres.',
    NSPhotoLibraryUsageDescription: 'Artnalyse accède à vos photos pour importer des images.'
  }
}
```

### Processus de release

```
1. Bump version dans app.config.js
2. eas build --platform ios --profile production
3. eas submit --platform ios  (upload vers App Store Connect)
4. Review Apple (~24-48h)
5. Publication manuelle ou automatique
```

## Performance

### Optimisations implémentées

- **Images** : compression avant upload (1024px max, qualité 0.8)
- **Historique** : limite hard 100 items avec rotation LIFO
- **Audio cache** : MP3 stocké localement, pas re-téléchargé
- **Base64** : thumbnails compressées (200px) pour l'historique, image originale uniquement pour l'analyse

### Points d'amélioration identifiés (V2)

- Mise en cache des analyses récentes (même oeuvre re-photographiée)
- Lazy loading de l'historique (pagination)
- Background download TTS pour les livres précédents
