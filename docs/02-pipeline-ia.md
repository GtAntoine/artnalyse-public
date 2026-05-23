# 02 — Pipeline IA : Artnalyse

## Vue d'Ensemble

Le pipeline IA d'Artnalyse combine trois technologies complémentaires pour produire une analyse complète d'une oeuvre d'art en temps réel :

```
Photo oeuvre
     ↓
[Optionnel] Reverse Image Search (Google Lens / SerpAPI)
     ↓
OpenAI GPT-5 Vision — Streaming NDJSON 5 sections
     ↓
OpenAI TTS — Audio en parallèle
     ↓
Affichage progressif + lecture audio
```

## Étape 1 : Capture et Préparation

### Modes de capture
- **Caméra native** : expo-camera, capture temps réel
- **Galerie** : expo-image-picker, import depuis la photothèque
- **Double photo** : oeuvre principale + cartel (pancarte muséale) optionnel
- **Saisie manuelle** : artiste + titre optionnels (enrichissement direct sans reverse search)

### Préparation de l'image
```typescript
// Compression avant envoi (< 1MB recommandé pour latence optimale)
const result = await ImageManipulator.manipulateAsync(
  uri,
  [{ resize: { width: 1024 } }],
  { compress: 0.8, format: ImageManipulator.SaveFormat.JPEG, base64: true }
);
```

## Étape 2 : Reverse Image Search (conditionnel)

### Déclenchement
La recherche Google Lens est activée seulement si :
- Pas de photo de cartel fournie
- ET artiste + titre ne sont PAS tous les deux saisis manuellement

### Architecture SerpAPI Google Lens

```typescript
// supabase/artwork-analysis-stream/reverse-image-search.ts

async function searchWithGoogleLens(imageBase64: string): Promise<LensResult[]> {
  // 1. Upload temporaire vers Supabase Storage
  const { data } = await supabase.storage
    .from('temp-images')
    .upload(`lens-${uuid}.jpg`, base64ToBlob(imageBase64));

  // 2. Recherche Google Lens via SerpAPI
  const response = await fetch(`https://serpapi.com/search?engine=google_lens&url=${publicUrl}&api_key=${SERPAPI_KEY}`);

  // 3. Timeout 16 secondes (expérience utilisateur vs précision)
  return withTimeout(response.json(), 16000);
}
```

### Algorithme de Scoring

```typescript
function scoreResult(result: LensResult): number {
  let score = 0;

  // Correspondance titre (most important)
  if (exactTitleMatch(result.title)) score += 0.4;
  else if (partialTitleMatch(result.title)) score += 0.2;

  // Correspondance artiste
  if (artistMatch(result.snippet)) score += 0.3;

  // Source institutionnelle (musée, encyclopédie)
  if (isInstitutionalSource(result.source)) score += 0.2;

  // Cohérence temporelle (date mentionnée)
  if (dateCoherence(result.snippet)) score += 0.1;

  return Math.min(score, 1.0);
}

// Niveaux de confiance → wording dans le prompt
// High (≥0.8)   : "J'ai identifié avec certitude..."
// Medium (≥0.5) : "Il s'agit probablement de..."
// Low (<0.5)    : "Cette oeuvre pourrait être..."
```

### Enrichissement du prompt LLM

Le résultat Google Lens est transmis comme "hint secondaire" au LLM. Il ne force jamais l'identification — le LLM peut le contredire si son analyse visuelle diverge.

```typescript
const hint = highConfidenceResult
  ? `Hint (Google Lens, confiance élevée): "${result.title}" par ${result.artist}. Utilise cette information mais confirme par ton analyse visuelle.`
  : `Hint (confiance faible): Potentiellement lié à "${result.title}". Analyse l'image de façon indépendante.`;
```

## Étape 3 : Streaming OpenAI GPT-5 Vision

### Format NDJSON 5 sections

Chaque section est émise comme une ligne JSON dès qu'elle est complète. Le mobile peut commencer l'affichage section 1 pendant que les sections 2-5 se génèrent.

```typescript
// Format de chaque ligne NDJSON
type NdjsonLine =
  | { type: 'identification'; data: ArtworkIdentification }
  | { type: 'narrative';      data: { text: string } }
  | { type: 'key_points';    data: KeyPoint[] }
  | { type: 'anecdote';      data: Anecdote }
  | { type: 'related_artworks'; data: RelatedArtwork[] }

// Exemple de ligne 1 (identification)
{
  "type": "identification",
  "data": {
    "title": "La Victoire de Samothrace",
    "artist": "Artiste inconnu",
    "year": "Vers 200-190 av. J.-C.",
    "period": "Hellénistique",
    "movement": "Art grec",
    "medium": "Marbre de Paros",
    "dimensions": "244 cm (hauteur)",
    "location": "Musée du Louvre, Paris"
  }
}
```

### Prompt Engineering

```
System: Tu es un guide culturel expert, spécialisé dans l'histoire de l'art.
Tu analyses des oeuvres d'art et produis des analyses structurées en JSON.
Réponds UNIQUEMENT avec des lignes JSON valides (NDJSON), une section par ligne.

Sections dans l'ordre :
1. identification : titre, artiste, année, période, mouvement, technique, dimensions, localisation
2. narrative : texte audioguide fluide 200-250 mots à la 2ème personne du singulier
3. key_points : 4-5 points clés { label, value }
4. anecdote : UNE anecdote captivante { titre, contenu }
5. related_artworks : 4 oeuvres similaires { titre, artiste, raison }

[Context: hint reverse search si disponible]
[Images: oeuvre principale + cartel optionnel]
```

### Streaming via XMLHttpRequest

React Native ne supporte pas `fetch` avec streaming natif. Solution : XMLHttpRequest avec lecture progressive de `responseText`.

```typescript
// src/lib/openai/secure-client.ts

function streamOpenAIChat(
  payload: ChatPayload,
  onLine: (line: NdjsonLine) => void,
  onComplete: () => void
): void {
  const xhr = new XMLHttpRequest();
  let processedLength = 0;

  xhr.onprogress = () => {
    const newText = xhr.responseText.slice(processedLength);
    processedLength = xhr.responseText.length;

    // Traitement ligne par ligne
    const lines = newText.split('\n').filter(Boolean);
    for (const line of lines) {
      try {
        const parsed = JSON.parse(line) as NdjsonLine;
        onLine(parsed);

        // Si section narrative → déclencher TTS en parallèle
        if (parsed.type === 'narrative') {
          triggerTTSDownload(parsed.data.text);
        }
      } catch (e) {
        // Ligne incomplète — buffer pour la prochaine itération
      }
    }
  };

  xhr.onload = onComplete;
  xhr.open('POST', SUPABASE_EDGE_FUNCTION_URL);
  xhr.send(JSON.stringify(payload));
}
```

## Étape 4 : Text-to-Speech (parallèle)

### Pipeline TTS

Dès que la section `narrative` est reçue, le téléchargement audio démarre en parallèle avec l'affichage des sections suivantes.

```typescript
// src/lib/tts-stream-client.ts

async function downloadTTS(text: string, resultId: string): Promise<string> {
  // 1. Appel Edge Function TTS
  const response = await fetch(`${SUPABASE_URL}/functions/v1/openai-tts-stream`, {
    method: 'POST',
    body: JSON.stringify({ text, voice: 'nova', model: 'tts-1-hd' })
  });

  // 2. Stream vers fichier local (FileSystem)
  const audioPath = `${FileSystem.documentDirectory}tts/${resultId}.mp3`;
  await FileSystem.downloadAsync(response.url, audioPath);

  return audioPath;
}
```

### Contrôles audio (expo-audio)

```
AudioPlayer
├── Play / Pause
├── Skip -5s / Skip +15s
├── Seek slider (interactif)
├── Vitesse : 0.75x | 1x | 1.25x | 1.5x | 2x
└── Restart depuis le début
```

## Gestion des Erreurs

| Scénario | Comportement |
|---------|-------------|
| Google Lens timeout (>16s) | Continue sans hint, analyse directe |
| Oeuvre non identifiée | Analyse stylistique générale ("oeuvre impressionniste...") |
| Réseau lent | Affichage progressif des sections déjà reçues |
| TTS échoue | Texte affiché, bouton audio désactivé gracieusement |
| Image floue | GPT-5 Vision produit quand même une analyse approximative |

## Performance & Latences

| Étape | Latence typique |
|------|----------------|
| Upload image → Edge Function | ~500ms |
| Google Lens (si activé) | 2-8s |
| Première section NDJSON | 1-3s après démarrage streaming |
| Analyse complète (5 sections) | 5-15s |
| TTS disponible | 3-8s après section narrative |
| **Total ressenti utilisateur** | **~3-5s avant premier contenu** |
