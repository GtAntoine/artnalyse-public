# Architecture technique — Artnalyse

## Vue d'ensemble

Artnalyse est une app mobile qui s'appuie sur trois services externes : OpenAI pour l'analyse visuelle et l'audio, SerpAPI pour l'identification par image, et Supabase pour les fonctions serveur et le stockage temporaire. Côté client, l'app gère l'interface, la caméra, la lecture audio et l'historique local.

L'architecture a été pensée pour minimiser ce qu'il faut maintenir. Pas de base de données propriétaire, pas de backend applicatif complexe, pas d'infrastructure à scaler. Le gros du travail est délégué à des services spécialisés.

## Les grandes décisions

**Pourquoi les fonctions serveur plutôt qu'un appel direct depuis l'app ?**

Les clés API d'OpenAI et de SerpAPI ne peuvent pas être embarquées dans l'app. N'importe qui pourrait les extraire et les utiliser à nos frais. Toutes les clés sensibles vivent côté serveur, dans des variables d'environnement non exposées. L'app mobile n'a accès qu'à des clés anonymes limitées.

**Pourquoi Supabase plutôt qu'AWS ou un serveur dédié ?**

Supabase propose des fonctions serveur hébergées proches des services OpenAI, ce qui réduit la latence. La mise en place est rapide, la gestion est minimale, et le modèle de tarification est adapté à une app en croissance : on paie selon l'usage, pas un forfait fixe mensuel.

**Pourquoi React Native et pas une app native Swift/Kotlin ?**

Un seul codebase pour iOS et Android. Sur une app solo, écrire deux fois le même produit n'est pas envisageable. React Native offre des performances proches du natif pour ce type d'usage, avec un accès à la caméra et aux fichiers locaux sans friction.

## Les arbitrages par fonctionnalité

**L'identification par image**

Lancer une recherche d'image prend du temps. La décision : on ne la déclenche que quand les informations manquent vraiment (pas de photo de cartel, pas de saisie manuelle du titre et de l'artiste). Quand l'utilisateur a fourni les informations, on passe directement à l'analyse. Ça évite d'ajouter 3 à 5 secondes pour les utilisateurs qui savent ce qu'ils regardent.

Un timeout de 16 secondes a été fixé sur la recherche d'image. Au-delà, l'app continue sans le résultat de la recherche. L'analyse est légèrement moins précise mais l'utilisateur n'attend pas indéfiniment.

**L'affichage progressif**

L'analyse arrive en cinq sections dans l'ordre. La première (identification) est disponible en quelques secondes. Les suivantes arrivent dans la foulée. Ce n'est pas un simple effet visuel — ça change fondamentalement la perception de la vitesse. Un utilisateur qui attend 10 secondes sans rien voir abandonne. Un utilisateur qui voit quelque chose apparaître au bout de 3 secondes attend volontiers la suite.

**Le cache audio local**

Le fichier audio du récit est sauvegardé sur l'appareil après la première génération. Réécouter une analyse depuis l'historique ne génère pas un nouvel appel à l'API — l'audio est déjà là. C'est un gain en coût et en vitesse, et ça permet la réécoute hors connexion.

**L'historique limité à 100 entrées**

Stocker des centaines d'images et d'analyses en base64 sur un téléphone finirait par poser un problème d'espace. La limite de 100 œuvres est un choix pragmatique : c'est déjà beaucoup plus que ce qu'un utilisateur moyen analysera, et ça évite de gérer une infrastructure de stockage cloud pour l'historique.

## Ce qui a été écarté

**Une base de données d'œuvres propriétaire.** Tenter de référencer des millions d'œuvres d'art était hors de portée en termes de coût et de maintenance. L'approche retenue — analyse visuelle directe enrichie par la recherche d'image — couvre un spectre bien plus large sans avoir à maintenir une base de données.

**Le fine-tuning du modèle.** Entraîner un modèle spécifique à l'histoire de l'art est un projet de plusieurs mois et de plusieurs dizaines de milliers d'euros. Le modèle de vision généraliste d'OpenAI, bien prompté, donne des résultats suffisamment précis pour l'usage visé, avec une couverture qui s'améliore automatiquement à chaque nouvelle version du modèle.

**Les fonctionnalités sociales.** Partager des collections, commenter les analyses d'autres utilisateurs, suivre des profils — tout ça a été écarté en V1. Ça ajoute de la complexité sans résoudre le problème principal. La valeur de l'app est dans l'analyse, pas dans le réseau.

## Ce que ça coûte à maintenir

Artnalyse n'a pas de serveur à surveiller, pas de base de données à sauvegarder, pas d'infrastructure à scaler manuellement. Les fonctions serveur s'exécutent à la demande et s'arrêtent quand elles ont fini. Le coût principal est proportionnel à l'usage — les appels à l'API OpenAI et à SerpAPI sont facturés par requête.

C'est un modèle économique adapté à une app en croissance : les coûts augmentent avec l'usage, mais les revenus aussi. Il n'y a pas de frais fixes importants à couvrir avant d'avoir des utilisateurs payants.
