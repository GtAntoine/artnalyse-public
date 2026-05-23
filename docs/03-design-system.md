# 03 — Design System "Le Vernissage" : Artnalyse

## Philosophie

Le design d'Artnalyse est pensé comme une expérience muséale premium. Chaque interaction doit évoquer l'atmosphère d'un vernissage — l'intimité tamisée d'une galerie privée, la chaleur dorée des éclairages muséaux, la qualité d'un catalogue d'exposition haut de gamme.

**Principes directeurs :**
1. **L'obscurité comme toile** : le fond noir profond met en valeur les oeuvres, comme les murs d'un musée sombre
2. **L'or comme accent** : utilisé avec parcimonie, l'or champagne signale les éléments importants sans ostentation
3. **La sérif pour l'autorité** : Playfair Display ancre l'app dans la tradition éditoriale artistique
4. **L'animation comme respiration** : pas d'animations de distraction, seulement des transitions qui guident l'attention

## Palette de Couleurs

```
Backgrounds
├── bg-primary:    #080808  ← Fond principal (quasi-noir)
├── bg-surface:    #111113  ← Cartes, bottom sheets
└── bg-alt:        #1a1a1c  ← Éléments tertiaires, separators

Or Champagne (accent)
├── gold:          #c9a55c  ← Actions principales, highlights
├── gold-light:    #d4b46a  ← États survol, actif
├── gold-dark:     #a88845  ← États pressé, ombres
└── gold-glow:     rgba(201, 165, 92, 0.2)  ← Halos lumineux

Texte
├── ivory:         #f0efe9  ← Texte principal
├── warm-gray-1:   #b8b5af  ← Texte secondaire
├── warm-gray-2:   #7a7570  ← Texte tertiaire, captions
└── muted:         #4a4540  ← Texte désactivé

Feedback
├── success:       #4a9960  ← Favoris ajoutés
└── error:         #c94040  ← Erreurs
```

## Typographie

### Familles

```
Playfair Display (Google Fonts, bold/regular/italic)
— Usage: Titres oeuvres, noms d'artistes, section headers
— Caractère: autorité, tradition, élégance culturelle

System Font (San Francisco sur iOS)
— Usage: Corps de texte, labels, metadata, UI
— Caractère: lisibilité, modernité, cohérence système
```

### Échelle typographique

| Rôle | Famille | Taille | Poids |
|------|---------|--------|-------|
| Titre oeuvre | Playfair Display | 28px | Bold |
| Nom artiste | Playfair Display | 18px | Regular Italic |
| Section header | Playfair Display | 16px | Bold |
| Corps narratif | System | 16px | Regular |
| Points clés label | System | 11px | SemiBold Uppercase |
| Points clés valeur | System | 15px | Regular |
| Metadata | System | 13px | Regular |
| Caption | System | 11px | Regular |

## Composants Signature

### GoldDivider
Séparateur décoratif horizontal avec losange central :
```
──────────── ◆ ────────────
```
Utilisé entre chaque section majeure sur ResultScreen.

### AmbientParticles
20-30 particules flottantes animées en arrière-plan de HomeScreen. Mouvement aléatoire lent (durée 8-15s par cycle), opacité 0.1-0.3, couleur gold. Effet "poussière de musée".

### FadeInView
```typescript
// Entrée animée directionnelle
<FadeInView direction="up" delay={200}>
  <ArtworkTitle />
</FadeInView>

// Props: direction (up|down|left|right), delay (ms), duration (ms)
// Distance de slide: 20px, opacity 0→1
```

### AnalysisProgress Badge
Badge en ligne unique qui affiche l'étape courante du streaming :
```
● Recherche en cours...
● Identification...
● Récit en cours...
● Points clés...
● Oeuvres similaires...
● Analyse complète ✓
```
Couleur: gold, animation pulse breathing, disparaît à la fin du streaming.

### AudioPlayer
```
┌──────────────────────────────────────┐
│  ▶  Écouter le récit                 │
│                                      │
│  ◁◁  [═══════════●──────]  ▷▷       │
│  -5s                          +15s  │
│                                      │
│  [0.75×] [1×] [1.25×] [1.5×] [2×] │
└──────────────────────────────────────┘
```

### Tab Bar Dynamique
La barre d'onglets (RÉCIT | DÉTAILS | DÉCOUVRIR) est positionnée dynamiquement via `onLayout`. Elle se repositionne après la fin du streaming, en tenant compte du scroll offset pour rester visible sans masquer le contenu.

### ImageView
Visionneuse plein écran :
- Double-tap → zoom 2×
- Pinch → zoom libre
- Swipe vertical (haut ou bas) → fermeture avec animation de scale + opacity

## Effets & Animations

### Gold Glow Shadow
```typescript
// Utilisé sur les CTA, boutons principaux, accents
shadowColor: '#c9a55c',
shadowOffset: { width: 0, height: 0 },
shadowOpacity: 0.4,
shadowRadius: 12,
elevation: 8,  // Android
```

### Breathing Animation
```typescript
// Animation oscillante lente — éléments "en attente"
const breathe = useRef(new Animated.Value(1)).current;

Animated.loop(
  Animated.sequence([
    Animated.timing(breathe, { toValue: 1.02, duration: 1500 }),
    Animated.timing(breathe, { toValue: 0.98, duration: 1500 }),
  ])
).start();
```

### Ornamental Borders
Bordures dorées fines (0.5px, couleur `gold-dark`) sur les cartes premium. Évoquent les cadres de tableaux.

## Dark Mode

L'app est exclusivement en dark mode. Pas de light mode prévu — cela contredirait la philosophie "galerie tamisée". La décision est assumée pour maintenir la cohérence de l'expérience.

## Accessibility

- Contraste texte/fond : ratio ≥ 4.5:1 pour tout texte principal (ivory sur #080808)
- Taille de touche minimale : 44×44pt (guidelines Apple HIG)
- Support Dynamic Type : textes system scalables
- Haptic feedback : actions importantes avec retour tactile `impactLight/Medium`
