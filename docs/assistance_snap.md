# Assistance sémantique SNAP

SNAP segmente l'objet que l'on désigne d'un clic **et propose ce qu'il est**.
Dans l'annotateur, ce nom passe par la liste de classes du projet : la classe
correspondante est présélectionnée, et il ne reste qu'à appuyer sur **A**.

Le nom est une **proposition**. Rien n'est appliqué sans geste de
l'annotateur.

---

## 1. Les gestes

| Touche | Effet |
|---|---|
| **G** | nouvel objet : segmente l'objet sous le curseur (vue 3D) |
| **Maj+G** | ajoute un clic au **même** objet : le masque est recalculé avec tous les clics (dix au plus) |
| **Alt+G** | retire du masque l'objet sous le curseur (le voisin sur lequel il déborde) |
| **[** et **]** | baisse ou monte le seuil du masque, sans nouvelle requête |
| **Ctrl+G** | cherche une catégorie par son nom (« Pole », « Vegetation »…) dans le bloc autour du curseur |
| **A** | applique la classe présélectionnée |

**Avec la cible « t0 + t1 »** (panneau « Cible de l'édition », le défaut),
G et Maj+G segmentent l'objet **aux deux dates**, au même point 3D : chaque
clic est reporté sur le point le plus proche de l'autre nuage, s'il est à
moins de 0,5 m (ou de deux rayons de sélection). Plus loin, l'objet n'y est
pas — déplacé, démoli, pas encore construit — et la ligne d'état le dit
(« absent en t1 »). Cela coûte deux requêtes au lieu d'une ; avec une seule
date ciblée, l'autre n'est pas touchée. La suite — reporter, décider du
changement, refléter les classes — est dans le document « Annoter les deux
dates ».

Le masque d'un nouveau clic **remplace** celui du clic précédent. La
sélection qui existait avant le premier clic est conservée. Les clics de
l'objet en cours sont dessinés dans la vue 3D : **cyan** pour l'objet,
**orange** pour les retraits.

**La vue reste libre pendant le calcul.** La requête part hors du fil de
l'interface : on tourne la scène, on vise la suite. Un nouveau G remplace
celui qui attendait encore son tour ; seul le dernier est montré. La ligne
« ⏳ segmentation… » du panneau dit qu'une réponse est en route.

Un masque qui s'arrête trop tôt se reprend avec **Maj+G** sur la partie
manquante. Mesuré sur un arbre de Boreas, le second clic fait passer l'IoU
prédit de 0,26 à 0,96.

Un masque qui **déborde** se reprend de trois façons :

- **le seuil** (panneau, ou **[** **]**) : le service renvoie la probabilité
  de chaque point, et le masque se resserre (au-dessus de 0,5) ou s'élargit
  (en dessous) sous les yeux ;
- **« seulement ce qui touche les clics »** (coché par défaut) : les
  morceaux détachés de l'objet cliqué partent — un autre poteau, la haie
  d'en face, que le modèle allume parfois parce qu'ils lui ressemblent ;
- **Alt+G** sur le voisin. SNAP n'a pas de clic négatif : il n'a été
  entraîné qu'avec des clics *sur* l'objet. On segmente donc le voisin, et
  on l'ôte du masque.

---

## 2. Huit modèles, trois domaines

Les huit points de contrôle publiés par les auteurs. Chacun se télécharge au
premier démarrage du service qui l'emploie, et son empreinte SHA-256 est
vérifiée avant d'être chargé.

| | Entraîné sur | Domaines |
|---|---|---|
| **SNAP-C** | ScanNet, HM3D, SemanticKITTI, nuScenes, Pandaset, STPLS3D, DALES | les trois — modèle principal, recommandé par les auteurs |
| **SNAP-Outdoor** | SemanticKITTI, nuScenes, Pandaset | route |
| **SNAP-C, grille variable** | les 7 jeux de SNAP-C, voxels de taille tirée au hasard | les trois — recommandé quand la grille diffère |
| **SNAP-C, sans HM3D** | les jeux de SNAP-C moins HM3D | les trois |
| **SNAP-KITTI** | SemanticKITTI seul | route |
| **SNAP-Aerial** | STPLS3D (photogrammétrie de drone), DALES (lidar aéroporté) | aérien |
| **SNAP-Indoor** | ScanNet, HM3D | intérieur |
| **SNAP-ScanNet** | ScanNet seul | intérieur |

**Le domaine** se choisit sous le modèle. Il décide de trois choses, reprises
du code des auteurs :

| Domaine | Voxels | Ce que le modèle reçoit | Noms proposés |
|---|---|---|---|
| **Route** (MLS) | 5 cm | xyz, sol local à −1,73 m comme un lidar de voiture | les 19 classes de SemanticKITTI |
| **Intérieur** | 2 cm | xyz, couleur, normales calculées sur le bloc | les 200 classes de ScanNet200 |
| **Aérien** (ALS, ULS) | 33 cm | xyz, couleur | les classes de STPLS3D et de DALES |

L'intérieur et l'aérien ont appris **avec la couleur**. Une tuile sans RVB
leur est envoyée en gris neutre, et la ligne d'état le signale (« sans
couleur »). Seuls les noms de la route ont une traduction vérifiée vers les
classes du projet ; ailleurs, le nom proposé passe par la ressemblance des
noms (voir la section 4).

**Comparer avec…** relance le même objet, avec les mêmes clics, sur un autre
modèle du même domaine. Les deux lignes s'affichent côte à côte, chacune avec son masque, et
**Prendre** garde celui qu'on préfère.

Ce qu'on a observé sur deux tuiles Boreas :

- **Les masques des objets isolés** (arbres, mobilier) sont nets avec les
  deux modèles.
- **Les noms divergent**, dans les deux sens :
  - SNAP-C a appelé « Building » une masse de houppiers que SNAP-Outdoor
    nommait « Vegetation » ;
  - SNAP-Outdoor a appelé la chaussée « Vegetation » ou « Sidewalk », quand
    SNAP-C disait « Road ».

D'où la comparaison, et d'où la règle : un nom n'est jamais appliqué seul.

---

## 3. Ce que montre une ligne de résultat

`SNAP-C : Vegetation 0.43 → vegetation`
`13 225 points · IoU prédit 0.26 · même nom`

- **Vegetation 0.43** : le nom dans le vocabulaire du modèle, et sa
  confiance (cosinus CLIP).
  - **Sous 0,30, aucune classe n'est proposée.** Sur Boreas, les noms
    parasites se regroupent entre 0,18 et 0,23 (presque toujours
    « Other_Ground »), les noms sensés entre 0,33 et 0,46.
- **→ vegetation** : la classe du projet retenue.
- **IoU prédit** : la qualité que le modèle prête à son propre masque. Sous
  0,5, un second clic (Maj+G) vaut souvent la peine.
- **La dernière mention** dit comment le nom a été traduit (voir section 4).

---

## 4. Du nom du modèle à la classe du projet

Le vocabulaire de SNAP en extérieur, ce sont les 19 classes de SemanticKITTI,
dans l'ordre exact du schéma `semantickitti19` de cdforge. La traduction suit
trois étapes :

1. **Le nom lui-même**, s'il est une classe, un alias ou un groupe du projet.
   C'est le cas d'un projet qui a gardé les classes d'un modèle.
2. **Le schéma vérifié** `semantickitti19`. C'est lui qui sait que « Trunk »
   est un arbre (`high_vegetation`), qu'un « Bicyclist » est une personne et
   que « Terrain » est un sol naturel.
3. **Le repli** : une classe absente du projet va à la base de son groupe.
   Sur les 10 classes de base, « Car » devient **vehicle**, et la ligne
   l'indique (« repliée sur… »).

Si aucune étape n'aboutit, aucune classe n'est choisie et la ligne le
signale.

---

## 5. Démarrer le service

Le modèle tourne dans **un autre interpréteur Python**, celui de Pointcept
(torch, spconv, flash-attn, clip). cdforge n'en dépend pas.

Le panneau « Assistance sémantique (SNAP) » de l'onglet Étiqueter :

- **Démarrer** lance le service et l'arrête avec la fenêtre. Le premier
  chargement prend 5 à 10 s.
- **Réglages…** désigne l'interpréteur et le dépôt SNAP cloné
  (`git clone https://github.com/neu-vi/SNAP`). Les deux sont proposés
  automatiquement quand on les trouve.
- **Les points de contrôle absents** sont téléchargés depuis les liens du
  dépôt officiel (environ 200 Mo chacun), vers `~/.cache/cdforge/models/snap/`.
  Leur empreinte est vérifiée : un point de contrôle est un pickle, et l'on
  ne charge que le fichier des auteurs.
- **Le journal** du service est `~/.cache/cdforge/models/snap/service.log`.

On peut aussi lancer le service soi-même. L'annotateur le trouve à
l'adresse `CDFORGE_ASSIST_URL`, par défaut `http://127.0.0.1:8765` :

```
<python de Pointcept> cdforge/annotate/snap_server.py --repo ~/SNAP \
    --checkpoint snap-c=SNAP_C.pth --checkpoint snap-outdoor=SNAP_Outdoor.pth
```

---

## 6. Comment le modèle voit la tuile

Les réglages ci-dessous viennent de l'article de SNAP et de ses fichiers
d'entraînement, pas de l'intuition.

- **Un bloc autour du clic, pas la tuile entière.**
  - Sur KITTI-360 accumulé (le jeu le plus proche d'une tuile MLS), SNAP-C
    tombe à 23,1 d'IoU au premier clic sur la scène entière, contre 65,6 sur
    des extraits centrés sur l'objet.
  - Le **rayon** se règle dans le panneau (20 m par défaut).
  - Le service plafonne le bloc à 400 000 voxels de 5 cm, ce qui donne 13 à
    15 m sur une tuile Boreas dense : au-delà, il fallait jusqu'à 40 Go de
    mémoire graphique.
  - **Un masque qui touche le bord du bloc** (plus de 2 % de ses voxels à
    moins d'un mètre du bord) est étendu une fois : un second bloc, centré
    sur le masque, est encodé, et chaque point garde la plus forte des deux
    probabilités. La ligne d'état le signale (« étendu au-delà du bloc »).
    Sur trois tuiles Boreas denses, 7 clics sur 24 en ont eu besoin.
  - **Un objet beaucoup plus grand que le bloc reste coupé** : une chaussée,
    une longue façade. La croissance de région les finit mieux.
- **Le repère d'un capteur** (domaine route). Le bloc est recentré sur le
  clic, le sol local placé à −1,73 m, comme le lidar de KITTI. Avec le sol à 0, les noms
  devenaient plus souvent « Other_Ground ».
- **Intérieur et aérien** : le bloc est recentré comme dans le code des
  auteurs (centre de l'emprise, point le plus bas à zéro). Un essai sur une
  pièce synthétique de 6 × 6 m avec une table : en intérieur, SNAP-C prend
  la table entière (IoU 0,99) et la nomme « table » ; sans la couleur,
  l'IoU tombe à 0,93.
- **Pas d'intensité.** Toutes les configurations d'entraînement en extérieur
  la retirent.
- **Mémoire de travail.**
  - La tuile est envoyée au service **dès qu'elle s'ouvre**, en arrière-plan :
    le premier clic ne transporte plus ses millions de points.
  - Un nouveau bloc s'encode en 0,4 à 0,6 s (médiane 0,59 s sur 24 clics de
    trois tuiles Boreas denses ; c'était environ 1,1 s avant que la
    voxélisation ne passe d'un tri ligne à ligne à une clé entière).
  - Les clics suivants dans le même bloc prennent environ 0,05 s.
- **La recherche par nom** (Ctrl+G) sème des clics tous les 2 m dans le
  bloc. Elle garde les masques dont le nom est celui demandé et ne sème plus
  là où un masque est déjà passé. Elle prend environ 4 s.

---

## 7. Limites

- **SNAP n'a jamais vu de MLS dense** : ni Boreas ni aucun relevé
  comparable. Les mesures de l'article portent sur d'autres jeux de données.
- **Les observations de ce document** viennent de deux tuiles Boreas, sans
  vérité terrain : aucun relevé MLS étiqueté n'était disponible pour une
  mesure chiffrée.
- **Il ne connaît que 19 noms** sur la route. Un objet hors de ce
  vocabulaire (mobilier urbain, chantier) reçoit le nom le plus proche, ou
  aucun s'il est sous le seuil.
- **Il ne nomme que la sémantique.** Pour le changement, il prend l'objet
  aux deux dates (G avec « t0 + t1 ») ; la décision vient ensuite de la
  comparaison des deux (touche **K**, document « Annoter les deux dates »).
