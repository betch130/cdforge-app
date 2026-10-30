# Comment le pré-label décide d'un changement

Ce document décrit ce que fait réellement le code (`cdforge/labels/prelabel.py`,
`cdforge/labels/visibility.py`, `cdforge/core/trajectory.py`). Chaque réglage
cité porte son nom dans `project.yaml`, section `prelabel:`.

Le pré-label ne remplace pas l'annotation : il propose, avec une confiance, et
l'annotateur valide. Les labels de vérité terrain et les labels posés à la main
ne sont **jamais** écrasés.

---

## 1. Vue d'ensemble

Une tuile contient deux époques, **t0** et **t1**, déjà recalées l'une sur
l'autre (t1 a reçu la matrice du recalage au tuilage). Le pré-label traite
chaque époque à son tour, contre l'autre, **avec exactement le même code** :

- les points de t0 sont comparés à t1 → « inchangé », « retiré », « non observé » ;
- les points de t1 sont comparés à t0 → « inchangé », « ajouté », « non observé ».

Dans la suite, **A** est l'époque dont on étiquette les points et **B** l'autre.

Pour chaque point **p** de A :

```
                       ┌───────────────────────────────┐
                       │ 1. Mesurer : B a-t-il assez   │
                       │    de points en face de p ?   │
                       └──────────────┬────────────────┘
                     oui (n_B ≥ min_pts)    non (n_B < min_pts)
                 ┌─────────────┘                 └──────────────┐
   ┌─────────────▼─────────────┐          ┌───────────────────▼──────────────────┐
   │ 2. Distance |d| > seuil ? │          │ 3. B regardait-il cet endroit ?      │
   │  oui → ajouté / retiré    │          │  (couverture, puis rayons si une     │
   │  non → inchangé           │          │   trajectoire existe)                │
   └───────────────────────────┘          │  oui → ajouté / retiré (disparu)     │
                                          │  non → non observé (angle mort)      │
                                          └──────────────────────────────────────┘
                       ┌───────────────────────────────┐
                       │ 4. Petits amas → inchangé     │
                       └───────────────────────────────┘
   « transitoire » n'est jamais décidé ici : l'annotateur le propose.
```

Le lancer de rayons n'intervient **qu'à l'étape 3**, et seulement pour les points
qui n'ont pas de support dans l'autre époque. Partout ailleurs, il ne change rien.

---

## 2. Étape 1 — la mesure (M3C2 simplifié, ou C2C)

### M3C2 (`change_method: m3c2`, par défaut)

1. **Orientation de la surface** : la normale de p est estimée par ACP sur ses
   `k_normals` plus proches voisins dans A.
2. **Cylindre** : un cylindre de rayon `cyl_radius`, de demi-longueur `cyl_depth`,
   centré sur p et orienté selon la normale. On y compte les points de A (n_A) et
   de B (n_B), `k_cyl` au plus de chaque.
3. **Même surface** (`same_surface_angle`, `same_surface_planarity`) : autour
   d'un point dont le voisinage est franchement plan, seuls comptent les voisins
   dont la normale fait moins de `same_surface_angle` degrés avec celle de p.
   Sans cette règle, le cylindre d'un point de sol attrapait le mur d'un bâtiment
   neuf ou la voiture voisine, et déclarait changé un sol qui n'avait pas bougé.
4. **Distance** : d = moyenne de B − moyenne de A, le long de la normale.
5. **Seuil de détection (LoD95)**, d'après Lague et al. (2013) :

   ```
   LoD95 = 1,96 × ( √(σ_A²/n_A + σ_B²/n_B) + erreur de recalage )
   ```

   σ_A et σ_B sont les rugosités dans le cylindre. L'erreur de recalage
   (`reg_error`) est par défaut le résidu NMAD du recalage de la paire.

### C2C (`change_method: c2c`)

Distance au plus proche voisin dans B. Pas de normale, plus rapide, mais pas de
rugosité par point : le seuil vaut l'erreur de recalage **plus l'espacement
médian des points de B** (deux échantillonnages d'une même surface ne
coïncident jamais, la distance vaut déjà l'ordre de l'espacement).

### Ce qui sort de l'étape 1

- **n_B ≥ `min_pts`** : B a assez de points en face de p pour mesurer. On passe à
  l'étape 2.
- **n_B < `min_pts`** : B n'a (presque) rien en face de p. Deux explications
  incompatibles sont possibles, et c'est l'étape 3 qui tranche.

---

## 3. Étape 2 — le point a-t-il bougé ?

| Condition | Label | Confiance |
|---|---|---|
| \|d\| ≤ LoD95 | inchangé | 0,5 à 1 selon la marge |
| \|d\| > LoD95 | ajouté (point de t1) / retiré (point de t0) | 0,5 à 1 selon la marge |

---

## 4. Étape 3 — B regardait-il cet endroit ?

C'est **la** question difficile. Un point de A sans rien en face dans B peut
signifier :

- **l'objet a disparu** : B regardait là et n'a rien vu → vrai changement ;
- **l'objet était caché** : un camion garé, un arbre, un mur masquait l'endroit
  au passage de B → ce n'est pas un changement, c'est un angle mort.

Deux méthodes répondent, et peuvent se combiner.

### 4.1 La couverture (toujours calculée)

Elle ne regarde que **où** B a des points, pas d'où il regardait.

| Portion | Interrupteur | Réglage | Ce qu'elle teste |
|---|---|---|---|
| Même case au sol | `obs_column` | `obs_cell` | B a un point dans la même case d'une grille au sol de `obs_cell` mètres |
| Point proche | `obs_nearby` | `obs_radius` | B a un point à moins de `obs_radius` mètres de p (en 3D) |
| Côtés couverts | `obs_sides` | `obs_quadrants`, `obs_radius` | B a des points dans au moins `obs_quadrants` des 4 quadrants autour de p |

```
couverture = (même case  OU  point proche)   — parmi les portions actives
             ET côtés couverts               — si la portion est active
```

Si les deux premières portions sont coupées, la couverture répond toujours
« non observé ».

**Pourquoi les côtés couverts.** Au bord de ce que B a relevé, B n'a de points
que d'un côté. Sans cette règle, une bande de faux « retiré » large de
`obs_radius` suivait chaque bord du relevé (61 000 points sur la scène de
mesure, aucun vrai changement perdu en l'ajoutant). Le test se fait par quatre
petites requêtes : un disque de rayon `obs_radius/2` au centre de chaque
quadrant.

**La limite de la couverture.** Un camion garé masque une façade, mais le
camion, lui, est bien là : B a des points tout près, la couverture conclut
« observé », donc « disparu ». C'est la raison d'être des rayons.

### 4.2 Les rayons (quand B a une trajectoire)

La question devient géométrique : **la ligne entre le capteur de B et
l'emplacement de p était-elle libre ?**

#### a) La trajectoire

- Le fichier de trajectoire de la source de B (`trajectory` dans la source) est
  un CSV. Colonnes reconnues : `x / easting / x_utm…`, `y / northing / y_utm…`,
  et une altitude.
- **Plusieurs altitudes** (Boreas : `h_ellipsoidale` et `H_cgvd2013`, à 37 m
  d'écart). Il faut celle **du même système que le nuage**. Si `trajectory_z`
  est donné dans la source, c'est elle. Sinon cdforge la reconnaît sur le
  nuage : la bonne place le capteur entre 0,3 et 6 m au-dessus du sol mesuré
  sous lui (5ᵉ centile des points à moins de 2 m). Si aucune, ou plus d'une,
  n'est plausible, **aucun rayon n'est lancé** sur cette tuile plutôt qu'un
  rayon faux. Sur Boreas : `h_ellipsoidale`, capteur à 2,1 m du sol.
- **Points de vue** : les poses sont éclaircies, on n'en garde une que si elle
  est à plus de `ray_pose_spacing` mètres de la précédente gardée. À 200 Hz,
  les poses consécutives sont à quelques centimètres : « les trois plus
  proches » désigneraient sinon trois fois le même endroit.
- **Recalage** : t1 a été déplacé par la matrice du recalage au tuilage. Sa
  trajectoire subit **la même** transformation, sinon chaque rayon viserait à
  côté.

#### b) La grille d'occupation

Les points de B sont rangés dans une grille de voxels (cubes) de `ray_cell`
mètres. Un voxel est « occupé » s'il contient au moins un point de B. Pas de
maillage : un nuage n'a pas de surfaces.

#### c) Le rayon

Pour le point p :

1. On prend les `ray_viewpoints` points de vue les plus proches de p, à moins de
   `ray_max_range` mètres.
2. Pour chacun, on avance sur le segment capteur → p, par pas de `ray_cell`.
3. On **saute** les `ray_min_range` premiers mètres (portion « ignorer le
   véhicule », interrupteur `ray_skip_vehicle`) : la plateforme se voit
   toujours elle-même.
4. On **saute** les `ray_clearance` derniers mètres (portion « ignorer la
   surface du point », interrupteur `ray_skip_target`) : une surface *au* point
   ne doit pas compter comme un obstacle *devant* lui.
5. Si un échantillon tombe dans un voxel occupé, la ligne est **coupée**.

```
 capteur ●──────────────────────────────●──────────────● p
         │← ray_min_range →│     pas de ray_cell     │← ray_clearance →│
         (ignoré)           ■ voxel occupé ?           (ignoré)
```

- **Au moins une ligne libre** → B regardait là → **observé**.
- **Toutes les lignes coupées** → p était caché → **non observé**.
- **Aucun point de vue à portée** → les rayons ne savent rien (« non tranché »).

#### d) Ce qui est approché

- Un voxel occupé n'est pas forcément opaque : un feuillage clairsemé compte
  comme un mur.
- Avec un pas fini, un obstacle très mince peut passer entre deux échantillons.

Les deux erreurs vont vers « observé », donc « disparu » — le même sens que la
couverture seule. Les rayons corrigent une partie des cas, ils n'en dégradent
aucun.

Les implémentations Python et Rust donnent **les mêmes réponses au bit près**
(même échantillonnage) ; Rust arrête seulement chaque rayon à son premier
obstacle.

### 4.3 Combiner les deux

`visibility` choisit le principe :

- `coverage` : la couverture seule, aucun rayon ;
- `auto` (défaut) : les rayons là où B a une trajectoire, la couverture
  ailleurs. (`rays` est accepté et se comporte comme `auto`.)

Quand les rayons sont actifs, quatre portions décident de leur poids :

| Situation | Interrupteur | Activé (défaut) | Coupé |
|---|---|---|---|
| Rayons : libre, couverture : non | `ray_reveal` | observé → **disparu** | la couverture garde « non observé » |
| Rayons : coupé, couverture : oui | `ray_hide` | **non observé** (caché) | la couverture garde « observé » |
| Rayons et couverture d'accord | — | leur réponse | leur réponse |
| Aucun point de vue à portée | `ray_fallback` | la couverture répond | **non observé** |

Couper `ray_reveal` et `ray_hide` ensemble revient à calculer les rayons pour
les montrer dans l'annotateur sans qu'ils changent aucun label.

### 4.4 Le label qui en sort

| B regardait là ? | Label | Confiance |
|---|---|---|
| oui | ajouté / retiré (disparition) | 0,8 |
| non | non observé | 0,5 |

---

## 5. Étape 4 — nettoyage, et ce qui est laissé à l'annotateur

- **Petits amas** : les points changés sont regroupés (deux points à moins de
  `cluster_radius` sont dans le même amas). Un amas de moins de `min_cluster`
  points est remis à « inchangé » (confiance 0,3) : un vrai changement a une
  étendue, un point isolé est du bruit. `min_cluster: 1` désactive ce filtre.
- **Transitoires** : le pré-label ne les décide pas. Un véhicule garé à t0 et
  parti à t1 sort donc « retiré ». Dans l'annotateur, **« Changements
  temporaires… »** liste les groupes temporaires du projet (`transient_classes`,
  réglés dans l'éditeur de classes : véhicule, personne par défaut), compte les
  points ajoutés/retirés qui en portent la classe sur la tuile, et les
  requalifie en « transitoire » si vous le confirmez (annulable).

---

## 6. Voir une décision dans l'annotateur

Viser un point et appuyer sur **R** (« Expliquer la décision »). L'annotateur
**rejoue** la décision sur la tuile ouverte avec **la même fonction** que le
pipeline (`change_prelabel`, entrées préparées par `change_inputs`) et les
réglages actuels de `project.yaml`. Le premier R sur une tuile prend le temps
d'un pré-label de cette tuile (les normales des deux époques) ; les suivants
sont immédiats.

Le panneau **« Pourquoi ce label ? »** (en onglet à côté d'« Annotation »)
détaille chaque étape pour ce point, l'étape qui a tranché en gras :

- 1 — combien de points de l'autre époque en face (`n_B`, seuil `min_pts`) ;
- 2 — la distance `|d|` contre le seuil LoD95, si on a pu mesurer ;
- 3a/3b/3c — chaque portion de la couverture : oui, non, ou « portion
  coupée » si vous l'avez désactivée ;
- 3d — les rayons : combien testés, libre ou coupé ; puis la règle qui a
  combiné rayons et couverture (`ray_reveal`, `ray_hide`, `ray_fallback`) ;
- 4 — petit amas remis à « inchangé ».

La vue 3D montre, **par-dessus le nuage** (un rayon coupé traverse justement
les points qui le coupent) :

| Dessin | Sens |
|---|---|
| sphère jaune | le point expliqué |
| sphère bleue | une position du capteur de l'autre époque (un point de vue) |
| trait vert épais | rayon libre : l'autre époque voyait le point de là |
| trait rouge épais puis fin, sphère rouge | rayon coupé : épais jusqu'au premier voxel occupé (l'obstacle), fin au-delà |
| trait gris | tronçon ignoré (`ray_min_range` côté capteur, `ray_clearance` côté point) ; point de vue non essayé parce qu'un précédent voyait déjà le point ; ou rayon plus court que les deux tronçons réunis (rien à tester : compté libre, le panneau le signale) |
| carré jaune | la case au sol (`obs_cell`) de la portion « même case » |
| cercle jaune | le disque `obs_radius` de la portion « point proche » |
| quatre petits cercles | les côtés testés : vert, l'autre époque y a des points ; gris, vide |

Couverture et rayons ne sont dessinés que pour les points **sans mesure** :
ailleurs ils n'ont pas pesé. Le dessin suit les changements d'affichage
(t0, t1, t0 + t1) ; « Effacer le dessin » le retire. En mode « côte à côte »
les époques sont décalées à l'écran : repasser en « t0 + t1 ».

Les rayons sont aussi projetés dans le **plan** (mêmes couleurs), où l'on voit
de quel côté ils arrivent et ce qu'ils traversent.

**Trajectoires du capteur** (onglet Vue, groupe « Capteur et rayons », cochée
par défaut) : le passage du capteur de chaque époque autour de la tuile — t0
en orange, t1 en bleu — dans la 3D et le plan, avec les points de vue d'où
partent les rayons (un tous les `ray_pose_spacing` mètres). Seule la partie à
moins de `ray_max_range` de la tuile est tracée : c'est celle que les rayons
utilisent. La trajectoire de t1 est recalée comme son nuage. Le groupe dit,
par époque, combien de points de vue sont à portée, ou pourquoi il n'y en a
pas (source sans trajectoire, fichier illisible, trajectoire trop loin).
Le cadrage ne bouge pas : la vue reste sur la tuile. Dans la **coupe**, un
triangle de la couleur de l'époque marque chaque passage du capteur à travers
la ligne de coupe, à son altitude : on y voit s'il roule bien au-dessus de la
chaussée, et ce qu'il a autour de lui.

**Faisceaux du lidar** (même groupe) : d'où chaque point a **réellement** été
mesuré — un trait de la position du lidar au moment du tir jusqu'au point,
pour tous les points visibles, de la couleur de l'époque — resserrer une
tranche (clipping) pour y voir clair. Les exports
Boreas voxelisés n'ont plus de temps GPS (`gps_time` à 0), mais chaque point
garde dans `point_source_id` le numéro du scan qui l'a mesuré, et la
trajectoire a une ligne par scan conservé (un sur deux) : la pose du scan est
la ligne `point_source_id / 2`. Vérifié : chaque point est alors à moins de
160 m de sa pose, la portée réglée à l'export ; la moitié vient de plus de
27 m. Un nuage voxelisé ne garde qu'un scan par voxel : beaucoup de points ont
été vus de loin, le long de la route.

**Lignes de visée du pré-label** (même groupe) : ce ne sont **pas** des
faisceaux mesurés. C'est la ligne que le pré-label teste entre un point et les
points de vue de l'**autre** époque les plus proches, pour savoir si l'autre
époque pouvait le voir. Pour tous les points sans mesure visibles — les seuls
que les rayons tranchent.
Vert : libre ; rouge : coupée, tracée jusqu'à l'obstacle. « Montrer » garde
toutes les lignes, les coupées ou les libres. Elles suivent le clipping et les
classes masquées. Les verdicts sont ceux de la décision rejouée (comme
R).

Si le label enregistré dans la tuile diffère du label rejoué, le panneau le
dit. Trois raisons possibles : la tuile a été pré-étiquetée avec d'autres
réglages ; un label a été posé à la main ; la classe sémantique a changé
depuis (transfert sémantique, correction), ce qui change l'étape 5.

---

### Repérer les points que les rayons ont requalifiés

Le pré-label écrit, pour chaque point, si les rayons ont **changé** la réponse
de la couverture (champ `chg_ray` de la tuile) :

- **vu** (orange) : la couverture disait « non observé », une ligne libre dit
  le contraire → le point devient ajouté/retiré ;
- **caché** (bleu) : la couverture disait « observé », toutes les lignes sont
  coupées → le point devient non observé.

Dans l'annotateur :

- couleur **« rays »** : gris rien, orange vu, bleu caché ;
- Révision ▸ **Progression** : « rayons : N requalifiés », par époque ;
- Révision ▸ **Sélectionner les requalifiés (rayons)** ;
- dans la file de tuiles, le badge **« rayons »** marque les tuiles où ça arrive ;
- R sur un de ces points montre pourquoi.

Un label protégé (posé à la main, vérité terrain) n'est jamais marqué : les
rayons ne l'ont pas décidé.

---

## 7. Ce que dit le rapport

Pour chaque tuile et chaque époque, `visibility` dans le rapport de
pré-labellisation donne :

- `method` : `rays` ou `coverage` ;
- `n_rays` : points tranchés par les rayons ;
- `n_coverage` : points tranchés par la couverture ;
- `n_disagree` : points où les rayons ont **changé** la réponse de la
  couverture — c'est le chiffre qui dit si les rayons ont servi ;
- `n_revealed`, `n_hidden` : ce même nombre, séparé en « vus » et « cachés ».

Une trajectoire illisible est signalée une fois par source, dans le journal du
run (« trajectoire de … illisible, rayons désactivés pour cette source »).
L'écran Pré-labels ▸ Configurer affiche « Trajectoires lisibles : n sur N
sources ».

---

## 8. Récapitulatif des réglages

| Nom | Défaut | Rôle |
|---|---|---|
| `change_method` | m3c2 | M3C2 simplifié ou C2C |
| `k_normals` | 20 | voisins pour la normale |
| `cyl_radius`, `cyl_depth` | 0,5 m, 3 m | cylindre de mesure |
| `k_cyl` | 48 | points au plus par cylindre |
| `same_surface_angle`, `same_surface_planarity` | 45°, 0,5 | règle « même surface » |
| `min_pts` | 3 | en dessous, B « n'a rien en face » |
| `reg_error` | auto | incertitude du recalage dans le seuil |
| `obs_column`, `obs_cell` | oui, 1 m | portion « même case au sol » |
| `obs_nearby`, `obs_radius` | oui, 2 m | portion « point proche » |
| `obs_sides`, `obs_quadrants` | oui, 3/4 | portion « côtés couverts » |
| `visibility` | auto | rayons quand une trajectoire existe |
| `ray_cell` | 0,5 m | voxels et pas du rayon |
| `ray_viewpoints` | 3 | points de vue essayés |
| `ray_pose_spacing` | 2 m | espacement des points de vue |
| `ray_max_range` | 40 m | portée d'un point de vue |
| `ray_skip_vehicle`, `ray_min_range` | oui, 2 m | ignorer le véhicule |
| `ray_skip_target`, `ray_clearance` | oui, 0,5 m | ignorer la surface du point |
| `ray_reveal` | oui | les rayons peuvent révéler une disparition |
| `ray_hide` | oui | les rayons peuvent requalifier en caché |
| `ray_fallback` | oui | sans point de vue, la couverture répond |
| `min_cluster`, `cluster_radius` | 30, 0,6 m | filtre des petits amas |
| `transient_classes` | vehicle, person | groupes temporaires, utilisés par l'annotateur |

Source : `trajectory` (chemin du CSV), `trajectory_z` (colonne d'altitude, si
le fichier en a plusieurs).
