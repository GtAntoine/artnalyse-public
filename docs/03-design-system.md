# Design system "Le Vernissage" — Artnalyse

## Le parti pris

Artnalyse aurait pu ressembler à une app utilitaire — fond blanc, police système, boutons bleus. Ce choix aurait été plus rapide à produire. Il aurait aussi produit une app que les utilisateurs utilisent une fois et oublient.

Le parti pris inverse : l'app doit ressembler à une galerie privée. Pas un musée institutionnel, pas une startup qui fait dans la culture — une galerie tamisée, avec des murs noirs, un éclairage doré, des typographies qui ont de la présence. Le nom du thème, "Le Vernissage", résume l'intention.

Cette décision a une conséquence directe sur la rétention. Une app qui ressemble à quelque chose qu'on a envie de montrer à ses amis génère du bouche-à-oreille. Une app qui fonctionne mais ne ressemble à rien d'autre, non.

## Les couleurs

Trois niveaux de fond, du plus sombre au moins sombre, pour créer de la profondeur sans aplatir l'interface. Un seul accent chromatique : l'or champagne. Il signale les éléments importants — le bouton d'analyse, les titres d'œuvres, les séparateurs décoratifs — sans être omniprésent. Le texte principal est ivoire, pas blanc pur : plus doux sur fond noir, moins fatigant à lire dans un musée.

Le choix de l'or n'est pas arbitraire. L'or évoque le cadre d'un tableau, l'étiquette d'une galerie, la reliure d'un catalogue d'exposition. C'est un signal culturel que les utilisateurs décodent immédiatement, même inconsciemment.

## La typographie

Deux familles, deux rôles distincts.

Playfair Display pour les titres d'œuvres, les noms d'artistes, les en-têtes de section. C'est une typographie sérif avec du caractère — elle appartient au monde de l'édition culturelle et du catalogue d'art. Elle donne du poids à ce qu'elle compose.

La police système pour le corps de texte, les labels, les métadonnées. Lisible, rapide à rendre, cohérente avec les conventions iOS. Le mélange des deux crée une hiérarchie claire : ce qui est important a une police qui dit que c'est important.

## Les animations

Trois principes ont guidé les choix d'animation.

**Rien qui distrait.** Dans un musée, l'attention est déjà sollicitée. Les animations ne doivent pas compétitionner avec les œuvres. Elles doivent guider, pas divertir.

**Les entrées sont directionnelles.** Quand une section d'analyse arrive, elle glisse depuis le bas. C'est cohérent avec le sens de lecture et donne une sensation de flux continu plutôt que d'apparition soudaine.

**La "respiration".** Certains éléments ont une légère oscillation lente quand ils sont en attente — pendant que l'analyse se charge, pendant que l'audio se génère. C'est subtil mais ça donne une impression de vie, pas d'écran figé.

## Le player audio

Le player a été pensé pour un usage debout, dans un lieu public, souvent avec un seul pouce disponible.

Les zones de tap sont larges. Les boutons sont en bas de l'écran, accessibles au pouce. La vitesse se change en un tap. La barre de progression est interactive — on peut se déplacer dans l'audio sans avoir à précisément viser une zone de quelques pixels.

Le design évite le skeuomorphisme (imiter un vrai lecteur CD) et le minimalisme excessif (juste un triangle de lecture). Il vise un équilibre : les contrôles sont reconnaissables, mais intégrés dans l'esthétique générale de l'app.

## La cohérence comme exigence

Chaque écran applique les mêmes tokens de couleur, les mêmes marges, les mêmes tailles de texte. Cette cohérence n'est pas un détail esthétique — elle réduit la charge cognitive de l'utilisateur. Quand tout se ressemble, on apprend l'interface une seule fois et on peut se concentrer sur le contenu.

La règle appliquée : si un élément casse la cohérence visuelle sans raison fonctionnelle, il est revu. Un bouton qui jure avec le reste n'est pas acceptable même s'il fonctionne correctement.

## Ce que ce système dit sur l'app

Un design soigné dans une app culturelle n'est pas superficiel. Il dit à l'utilisateur : les gens qui ont fait ça prennent la culture au sérieux. Ils ont réfléchi à l'expérience, pas seulement à la fonctionnalité. C'est le signal de confiance qui pousse quelqu'un à sortir l'app dans un musée plutôt que de la cacher parce qu'elle fait cheap.
