# Comment l'analyse se construit — Artnalyse

## Le parcours d'une photo à l'analyse

L'utilisateur prend une photo. Trente secondes plus tard, il écoute un récit de 2 minutes sur l'œuvre qu'il a devant lui. Voici ce qui se passe entre les deux.

## L'identification : résoudre le problème de la page blanche

Le premier défi est d'identifier l'œuvre. Si l'utilisateur saisit lui-même le nom de l'artiste et du titre, le travail est simplifié. Mais dans la majorité des cas, il ne sait pas ce qu'il regarde — c'est justement pour ça qu'il utilise l'app.

La solution retenue : avant même d'envoyer la photo au modèle d'analyse, l'app interroge Google Lens pour tenter une identification par image. Le résultat de cette recherche est transmis au modèle comme un indice, pas comme une vérité absolue. Si Google Lens retourne "Victoire de Samothrace avec une confiance élevée", le modèle en tient compte. Si la confiance est faible, il analyse l'image de façon indépendante.

Cette approche en deux temps — recherche d'image puis analyse contextuelle — améliore significativement la précision sur les œuvres célèbres, sans dégrader les résultats sur les œuvres inconnues.

## L'analyse : une structure en cinq sections

L'analyse produit systématiquement cinq sections dans le même ordre :

**Identification** : titre, artiste, date, mouvement, technique, dimensions, lieu de conservation. Ce sont les informations qu'on trouve sur un cartel de musée, en plus complet.

**Récit** : 200 à 250 mots écrits à la deuxième personne, style audioguide professionnel. Ce texte est celui qui sera lu à voix haute. Il est rédigé pour être écouté debout devant une œuvre, pas pour être lu sur un écran.

**Points clés** : quatre à cinq éléments essentiels pour comprendre l'œuvre — le contexte historique, la technique utilisée, le symbolisme, ce qui est remarquable dans l'exécution.

**Anecdote** : une seule, choisie pour sa capacité à faire vivre l'œuvre. Le critère : est-ce que l'utilisateur va le raconter à quelqu'un ce soir ?

**Œuvres similaires** : quatre suggestions avec une phrase d'explication pour chacune. Pas une liste de l'artiste, mais des œuvres qui partagent quelque chose de précis avec celle analysée — une période, une technique, un thème.

## Pourquoi l'analyse s'affiche progressivement

Un spinner de 10 secondes tue l'expérience dans un musée. L'utilisateur est debout, peut-être entouré d'autres visiteurs, l'attention d'une œuvre d'art est fugace.

L'affichage progressif change la perception du temps d'attente. L'identification apparaît en quelques secondes, le récit suit, puis les autres sections. L'utilisateur commence à lire pendant que l'analyse se termine. En pratique, il n'attend jamais — il lit ou écoute en continu.

## L'audio : généré pendant l'affichage

Le récit textuel de la section 2 est transformé en audio dès qu'il est disponible, en parallèle de la génération des sections suivantes. Quand l'utilisateur appuie sur "Écouter le récit", le fichier audio est déjà prêt sur son appareil. Il est conservé localement : l'utilisateur peut réécouter une analyse passée sans connexion.

## La photo du cartel

Si l'utilisateur prend une photo du cartel en même temps que l'œuvre (l'étiquette avec le titre, l'artiste, la date), l'analyse est plus précise et la recherche d'image n'est pas déclenchée. C'est une fonctionnalité discrète mais utile pour les musées qui exposent leurs cartels de façon lisible.

## Les arbitrages

**Vitesse vs précision.** La recherche d'image ajoute quelques secondes. Elle est déclenchée uniquement quand les informations manquent — sinon, l'analyse démarre immédiatement. Ce choix préserve la rapidité pour les utilisateurs qui saisissent manuellement, et améliore la précision pour ceux qui ne savent pas ce qu'ils regardent.

**Généricité vs spécialisation.** Le modèle d'analyse n'est pas fine-tuné sur un corpus d'art. Il s'appuie sur sa connaissance générale, enrichie par les indices de recherche. Cette approche couvre un spectre bien plus large qu'un modèle spécialisé, au prix d'une précision légèrement inférieure sur les œuvres très obscures — un compromis acceptable pour l'usage visé.
