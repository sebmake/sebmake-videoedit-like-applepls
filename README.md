# sebmake-videoedit-like-applepls

Une skill Claude Code qui monte vos rushes, les vidéos brutes de votre tournage, à la manière des films projetés pendant les keynotes d'Apple. Elle assemble le film elle-même dans FilmCraft, une app de montage qu'elle pilote, ou vous remet un fichier à importer dans DaVinci Resolve ou Premiere Pro. Elle sait aussi critiquer un montage déjà fait, plan par plan, et dire quoi corriger, le plus souvent au timecode près (la position exacte dans le film, à l'image près). Ses règles ne sortent pas d'un manuel : on a découpé les films de huit keynotes, puis on a compté.

*English below.*

## Ce que les mesures ont donné

Le corpus réunit les films de huit keynotes, de 2019 à 2026, rangés en quatre familles : 16 films de produit, 20 films d'ouverture et d'ambiance, 8 documentaires, et une dernière famille qui regroupe 6 bandes-annonces et 2 films d'usage, où l'on voit des gens se servir d'un objet. Chaque règle de la skill porte son compte, « N sur M » : on l'a vue dans N cas sur les M mesurés (des films, des coupes ou des plans, selon la règle). Ce qu'on n'a pas mesuré est marqué « convention ».

Les comptes disent d'abord ce que ces films ne font pas. La musique n'y commande pas les coupes : dans les documentaires, 363 coupes sur 597 tombent au début d'un son, à peu près ce que donnerait le hasard (361,5). Le fondu enchaîné reste une exception : 594 coupes franches sur 606 changements de plan dans les documentaires. Un film de produit n'affiche ni carton explicatif ni chiffre (0 sur 16), et aucun ne finit sur un logo seul ou sur un slogan (0 sur 16).

La révélation, elle, tombe tard et presque à heure fixe. Dans les 10 films qui gardent l'objet pour la fin, il apparaît en entier entre 69 et 82 % de la durée (médiane 77 %), jamais dans le premier plan. Quand le nom est écrit (7 films sur 16), il arrive entre 83 et 96 % ; dans 8 des 9 autres, c'est l'orateur qui le prononce, juste après la projection. Les bandes-annonces accélèrent : tous films réunis, la durée médiane des plans baisse à chaque tiers (1,20 s, puis 0,90 s, puis 0,77 s), et le passage le plus rapide tombe entre 78 et 88 % de la durée dans les 6. Dans un documentaire, enfin, on ne voit la personne parler que pendant un tiers de ce qu'elle dit (médiane 33 %) ; le reste du temps, l'image montre ce dont elle parle.

## Comment elle travaille

Elle vous demande ce que vous avez (rushes, musique, montage déjà fait), la famille du film, sa durée, son format (16:9, vertical ou carré) et votre logiciel. Un script découpe chaque rush en prises et repère le flou, le tremblement, le noir et l'image brûlée, à des seuils de convention qui peuvent se tromper ; Claude regarde les vignettes et note ce que montre chaque prise. Avant de construire quoi que ce soit, la skill vous montre le déroulé du film en tableau, une ligne par plan, avec la règle que suit chacun.

Dans FilmCraft, elle monte la séquence dans un projet neuf qu'elle n'enregistre pas, et peut en exporter la vidéo. Pour Premiere Pro et DaVinci Resolve, elle écrit un fichier XML FCP7, vérifie image par image qu'il reproduit le montage, et vous dit comment l'importer ; une EDL CMX3600, plus sommaire, sert de secours. Pour changer le montage, demandez-le à Claude, qui refait l'export : une retouche faite à la main dans le XML serait perdue au prochain export.

La critique lit une vidéo, un XML FCP7 ou une EDL et compare le montage aux films de sa famille. Claude regarde une image de chaque plan pour ce que le calcul ne voit pas : la valeur des plans, la révélation, le nom, les textes. Chaque écart sort avec sa règle, son compte dans le corpus et, le plus souvent, un correctif du type « remplacez le fondu à 00:00:18:13 par une coupe ».

Claude n'invente ni nom, ni chiffre, ni citation, ni la fonction d'une personne filmée : ce qui manque reste absent, et le compte rendu le signale. La musique, c'est vous qui la fournissez ; sinon, le film n'en a pas. Rien d'Apple ne passe dans vos films, ni image, ni son, ni logo, ni phrase ; les titres prennent votre police (son fichier, pour des titres en images), ou une police courante si vous n'en donnez pas.

![Douze images du film de test de révélation, rendu dans FilmCraft à partir des rushes dessinés d'une lampe fictive](skills/sebmake-videoedit-like-applepls/assets/apercu-test.jpg)

## Installation

En plugin Claude Code :

```bash
claude plugin marketplace add sebmake/sebmake-videoedit-like-applepls
claude plugin install sebmake-videoedit-like-applepls@sebmake-videoedit
```

À la main, copiez `skills/sebmake-videoedit-like-applepls` dans `~/.claude/skills/`.

Il vous faut Python 3.9 ou plus, avec ffmpeg et ffprobe dans le PATH. L'analyse des rushes et la critique demandent aussi numpy et Pillow, que la skill cherche dans `~/.venv-videoedit` :

```bash
python3 -m venv ~/.venv-videoedit && ~/.venv-videoedit/bin/pip install numpy pillow
```

Le reste (assemblage, exports, pilotage de FilmCraft) n'utilise que la bibliothèque standard, sauf les titres du XML en images PNG, qui demandent Pillow. Pour Premiere Pro et DaVinci Resolve, rien de plus : vous importez le fichier. Pour FilmCraft, il faut Claude Code sur la machine où tourne l'app. Si elle est fermée, la skill la lance avec son canal de contrôle, la connexion locale par laquelle Claude la pilote ; si elle tourne déjà sans ce canal, quittez-la puis relancez-la avec `open -a FilmCraft --args --control 9876`, car la skill ne ferme jamais votre projet.

## Ce qui reste à éprouver

Lancée sur 46 films du corpus, la critique en déclare 42 conformes, sans une alerte, même si presque tous gardent une remarque ou un point à vérifier à l'œil. Sur un montage raté exprès (l'objet entier dès le premier plan, des plans de 0,6 s, six fondus, un carton chiffré, un slogan pour finir), elle lève 4 alertes en lisant le XML seul, et 9 avec les notes que Claude prend à l'œil. Aucun de ces deux essais ne prouve qu'elle trouve toutes les fautes.

Le cas de test est une lampe de bureau qui n'existe pas, « Orée » : 16 rushes dessinés puis filmés par une caméra simulée. La skill en a tiré une révélation de 45 s et une ouverture de 30 s, construites dans FilmCraft, exportées en XML et en EDL, rendues puis critiquées : zéro alerte. Ce zéro ne dit pas tout : la lampe se voit déjà en entier, en petit, à 9,3 s (image 4 ci-dessus), bien avant sa révélation, et ni l'assemblage ni la critique ne l'ont relevé ; l'ouverture, elle, est plus courte que toutes celles du corpus (65 à 143 s). La release v1.0.0 contient ces deux films, sans son, et le paquet .skill.

Personne n'a encore ouvert un fichier de la skill dans Premiere Pro ou DaVinci Resolve. Le XML et l'EDL sont relus par l'export lui-même, par un second script, et par l'importeur de FilmCraft, qui a retrouvé chaque plan à l'image près sur trois montages de test (en XML, et l'un d'eux aussi en EDL) ; les pas à pas d'import suivent la documentation des deux logiciels sans avoir été rejoués.

Une vidéo seule en dit moins qu'une timeline. Sur les rendus de test, la critique a manqué une coupe entre deux plans du même objet sur fond sombre, n'a vu aucun fondu vers ou depuis le noir, et a lu le nom, un petit texte clair sur noir, comme du noir. Qu'elle lise une vidéo ou un XML, elle prend un zoom ou une rotation pour un plan fixe. La critique reste aussi indulgente sur la place de la révélation : entre 60 et 66 % de la durée, celle-ci ne vaut qu'une remarque, et entre 66 et 69 % (ou entre 82 et 87 %), elle passe sans un mot. Les formats vertical et carré suivent les règles du 16:9 par convention, et aucun export ne recadre les plans. Le détail est dans [`critique.md`](skills/sebmake-videoedit-like-applepls/references/critique.md#8-ce-que-la-critique-ne-voit-pas) et [`logiciels.md`](skills/sebmake-videoedit-like-applepls/references/logiciels.md#7-limites-connues).

Un piège trouvé en route : dans FilmCraft, l'export de la séquence entière s'arrête au dernier plan et perd le noir final (44,04 s au lieu de 45,00 s). `filmcraft_build.py --rendre` exporte le montage de sa première à sa dernière image, noir compris.

---

## English

A Claude Code skill that edits your footage the way the films shown during Apple keynotes are cut, or reviews an edit you've already made and tells you what to fix, usually down to the timecode. It builds the timeline in FilmCraft, an editing app it drives directly, or hands you an FCP7 XML file for DaVinci Resolve and Premiere Pro, with an EDL as a fallback. The rules come from the films of eight Apple keynotes (2019 to 2026), broken down shot by shot and counted: 16 product films, 20 openers and mood pieces, 8 documentaries, 6 trailers and 2 product-in-use films.

A few of the counts: music doesn't drive the cuts (in documentaries, 363 of 597 cuts land on a sound onset, about what chance would give). No product film shows a number or ends on a slogan (0 of 16). When a film saves the product for the end, it shows up in full between 69% and 82% of the running time, never in the first shot. Trailers speed up: taken together, their median shot gets shorter with each third of the film (1.20 s, 0.90 s, 0.77 s).

It asks what you have and what you're making, shows you a shot-by-shot plan, then builds or exports it. It won't make up a name, a job title, a number or a quote, and it only uses music you supply. Install with `claude plugin marketplace add sebmake/sebmake-videoedit-like-applepls`, then `claude plugin install sebmake-videoedit-like-applepls@sebmake-videoedit`. You'll need Python 3.9+, ffmpeg and ffprobe, plus numpy and Pillow in `~/.venv-videoedit` (`python3 -m venv ~/.venv-videoedit && ~/.venv-videoedit/bin/pip install numpy pillow`).

On 46 corpus films, the review raised no alert on 42. On a deliberately botched edit, it raised 4 alerts from the XML alone and 9 with Claude's by-eye notes; neither test proves it catches every mistake. Nobody has opened its files in Premiere Pro or DaVinci Resolve yet; they've been checked by the skill, by a second script and by FilmCraft's importer. Reviewing a rendered video alone catches less than reviewing the timeline: on the test renders it missed a cut between two similar dark shots and didn't catch a single fade to or from black. Whether it reads a video or an XML file, it takes zooms and rotations for static shots, and a reveal anywhere from 66% to 87% passes without comment. Vertical and square formats simply reuse the 16:9 rules, since no corpus film is vertical or square, and no export reframes the shots.

---

Not affiliated with, endorsed or sponsored by Apple Inc. Apple, Keynote and Final Cut Pro are trademarks of Apple Inc. DaVinci Resolve is a trademark of Blackmagic Design. Premiere Pro is a trademark of Adobe. MIT License.
