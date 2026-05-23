# <img src="./public/images/icon.png" alt="Artnalyse" width="50" style="vertical-align: middle;"/> Artnalyse

> Photographiez une œuvre d'art, obtenez son analyse complète et son récit audio en quelques secondes

<div align="center">

[![App Store](https://img.shields.io/badge/App_Store-0D96F6?style=for-the-badge&logo=app-store&logoColor=white)](https://apps.apple.com/fr/app/artnalyse/id6762506288)
[![Google Play](https://img.shields.io/badge/Google_Play-414141?style=for-the-badge&logo=google-play&logoColor=white)](https://play.google.com/store/apps/details?id=fr.goethals.artnalyse)
[![React Native](https://img.shields.io/badge/React_Native-20232A?style=for-the-badge&logo=react&logoColor=61DAFB)](https://reactnative.dev/)
[![Expo](https://img.shields.io/badge/Expo-000020?style=for-the-badge&logo=expo&logoColor=white)](https://expo.dev/)
[![OpenAI](https://img.shields.io/badge/OpenAI-412991?style=for-the-badge&logo=openai&logoColor=white)](https://openai.com/)

**[📱 App Store](https://apps.apple.com/fr/app/artnalyse/id6762506288) · [🤖 Google Play](https://play.google.com/store/apps/details?id=fr.goethals.artnalyse) · [🌐 Site Web](https://goethals.fr/artnalyse/)**

</div>

---

<div align="center">
  <img src="./public/images/screenshot-1.jpg" alt="Artnalyse - Accueil" width="23%" />
  <img src="./public/images/screenshot-2.jpg" alt="Artnalyse - Récit audio" width="23%" />
  <img src="./public/images/screenshot-3.jpg" alt="Artnalyse - Détails" width="23%" />
  <img src="./public/images/screenshot-4.jpg" alt="Artnalyse - Découvrir" width="23%" />
</div>

---

## 📚 Documentation détaillée

- **[01 — Vision & marché](./docs/01-vision-produit.md)** — Positionnement, personas, opportunité
- **[02 — Pipeline IA](./docs/02-pipeline-ia.md)** — Comment l'analyse se construit, de la photo au son
- **[03 — Design system](./docs/03-design-system.md)** — "Le Vernissage" — couleurs, typographie, composants
- **[04 — Architecture technique](./docs/04-architecture-technique.md)** — Choix techniques et arbitrages

---

## C'est quoi

Artnalyse est une application mobile disponible sur iOS et Android. Elle permet d'analyser n'importe quelle œuvre d'art en la photographiant.

L'idée de départ : les audioguides de musée sont chers (3–5€ à l'entrée), couvrent 10% des œuvres, et leur contenu date de dix ans. Artnalyse fait ça pour n'importe quelle œuvre, dans n'importe quel musée du monde, gratuitement.

**Ce que l'utilisateur voit :**
- Il prend une photo — ou importe une image de sa galerie
- L'analyse arrive progressivement, section après section, sans écran de chargement : identification, récit, points clés, anecdote, œuvres similaires
- Il peut écouter le récit en audio (généré à la volée) avec un player complet : pause, avance rapide, vitesse variable
- Il retrouve toutes ses œuvres analysées dans un historique personnel

---

## Ce que ça démontre

**Construire un produit de A à Z, seul**

Artnalyse n'est pas un wrapper autour d'une API. C'est un produit complet, publié sur deux stores, avec une expérience utilisateur pensée pour qu'on oublie la technique derrière.

| Compétence | Ce qui le prouve |
|-----------|-----------------|
| Vision produit | Identifier un vrai problème (les audioguides ne couvrent pas tout), trouver un angle clair |
| UX mobile native | L'analyse s'affiche en temps réel, section par section — pas de spinner de 10 secondes |
| Architecture IA | Identifier l'œuvre même sans nom, via une recherche d'image inversée automatique |
| Design premium | Thème "Le Vernissage" — noir profond, or champagne, Playfair Display — cohérent sur chaque écran |
| Audio à la volée | Le texte est converti en audio et synchronisé pendant que l'analyse se construit |
| Publication | iOS App Store + Google Play, gestion des permissions caméra/galerie, cycles de release |

---

## Comment ça fonctionne

La photo est envoyée à une fonction serveur (Supabase Edge Functions). Si l'utilisateur n'a pas saisi le nom de l'œuvre, l'app lance d'abord une recherche d'image sur Google Lens pour l'identifier. Ensuite, le modèle de vision d'OpenAI analyse l'image et génère cinq sections dans l'ordre : identification → récit → points clés → anecdote → œuvres similaires. Chaque section arrive dès qu'elle est prête — l'utilisateur lit pendant que la suivante se génère.

L'audio du récit est généré en parallèle sur le même serveur et mis en cache localement. Il est disponible pour réécoute même hors connexion.

```
Photo → recherche d'image (si besoin) → analyse IA → 5 sections progressives
                                                    ↘ audio généré en parallèle
```

---

## Stack

| Couche | Choix |
|--------|-------|
| Framework | React Native + Expo |
| Langage | TypeScript |
| Backend | Supabase Edge Functions (Deno) |
| Vision | OpenAI GPT-4o Vision |
| Audio | OpenAI TTS + expo-audio |
| Identification | SerpAPI Google Lens |
| State | Zustand persisté (AsyncStorage) |
| Caméra | expo-camera + expo-image-picker |

---

## Design

Le thème s'appelle "Le Vernissage". L'idée : l'app doit ressembler à une galerie privée, pas à une app utilitaire. Fond quasi-noir (#080808), accents or champagne (#c9a55c), typographie Playfair Display pour les titres d'œuvres. Chaque écran applique les mêmes tokens — rien ne détonne.

---

## Roadmap

**Fait ✅**
- Analyse photo avec identification automatique
- Récit audio généré à la volée
- 5 sections d'analyse progressives
- Historique personnel + favoris
- Publication iOS + Android

**Prévu**
- [ ] Mode visite : naviguer entre œuvres d'un même musée
- [ ] Partage d'une œuvre analysée (lien public)
- [ ] Traductions EN, ES, DE, IT
- [ ] B2B : intégration dans les apps de musées
