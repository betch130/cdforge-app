# Annoter les deux dates

Une paire, ce sont deux passages au même endroit. L'annotateur montre les
deux nuages superposés. Il peut prendre un objet aux deux dates d'un seul
geste, y reporter les labels et en déduire le changement.

La règle de fond est la même partout : **rien ne traite t0 et t1
différemment**. Chaque opération vaut dans les deux sens. Un point de t1
sans correspondant en t0 est « ajouté » exactement comme un point de t0
sans correspondant en t1 est « supprimé ».

---

## 1. Les touches

| Touche | Effet |
|---|---|
| **G**, **Maj+G** | avec la cible « t0 + t1 », SNAP prend l'objet aux deux dates, au même point 3D |
| **T** | reporte la sélection sur l'autre date, l'objet entier si SNAP est prêt, et ses labels avec |
| **K** | propose le changement de l'objet sélectionné, d'après les deux dates (aperçu, puis Appliquer) |
| **M** | active ou coupe le miroir : chaque classe posée l'est aussi sur l'autre date |
| **Ctrl+Z** | annule chacune de ces opérations, les deux dates d'un coup |

La liste complète des raccourcis est dans **F1**.

---

## 2. Correspondants

Toutes ces opérations reposent sur la même question : **ce point a-t-il un
correspondant sur l'autre date ?**

- **Le correspondant** est le point le plus proche de l'autre nuage, cherché
  dans le nuage **entier**, et non parmi les seuls points sélectionnés. Au
  pied d'un mur, le sol de l'autre date est plus près du mur qu'un point du
  mur lui-même : le chercher parmi la sélection le ferait passer pour du mur.
- **Le rayon** est le « Rayon croissance » des outils de sélection. Il est
  relevé à **1,5 fois l'espacement des points** du nuage le moins dense. Sans
  ce relèvement, un passage deux fois moins dense laisserait des trous
  entre ses points, et ces trous passeraient pour du changement.
  L'aperçu de **K** signale quand le rayon a été relevé.
- **Les filtres actifs** (non labellisés, une classe, emprise, hauteur sur
  le sol, coupe…) valent aussi sur l'autre date.

---

## 3. T : reporter sur l'autre date

T se déroule en trois temps :

1. **La sélection passe aux correspondants** sur l'autre date.
2. **L'objet entier**, si le service SNAP a répondu et si la sélection tient
   dans son bloc. SNAP repart de trois clics bien répartis sur les
   correspondants et prend l'objet sur l'autre date. Les deux passages n'ont
   ni la même densité ni les mêmes occlusions, et la seule proximité en
   laisserait des morceaux. Une sélection au polygone sur tout un îlot n'est
   pas « un objet » : elle s'arrête à l'étape 1.
3. **Les labels suivent.** Chaque correspondant reçoit la classe du point le
   plus proche de la sélection. Ce que SNAP a trouvé en plus (la partie
   vue seulement sur l'autre date) reçoit la classe majoritaire de l'objet.

Ne sont jamais écrasés :

- ce qui est **déjà manuel** sur l'autre date ;
- ce qui y était **déjà sélectionné** : c'est vous qui l'y avez mis.

Sur la couche changement, seuls « inchangé », « transitoire » et « non
observé » se reportent. Un « ajouté » n'existe que sur sa date.

---

## 4. K : le changement de l'objet

Sélectionnez l'objet aux deux dates (G avec « t0 + t1 », ou une date puis
T), puis appuyez sur **K**.

| Point | a un correspondant | n'en a pas |
|---|---|---|
| de **t0** | inchangé | supprimé |
| de **t1** | inchangé | ajouté |

Ce qui était « non observé » le reste : l'autre date ne voyait pas cet
endroit, et l'absence de correspondant n'y prouve rien.

**L'aperçu** donne les comptes par date avant toute écriture. Il prévient
dans trois cas :

- **le rayon a été relevé** à l'espacement des points ;
- **aucun pré-label de changement** ne couvre l'objet. Sans les rayons du
  pré-label, rien ne dit où l'autre date était aveugle : un point sans
  correspondant peut y être caché (occlusion, emprise) plutôt que supprimé ;
- **des labels de changement manuels** vont être remplacés (leur nombre est
  donné).

Il dit aussi quand rien n'est sélectionné sur une des dates. « Appliquer »
écrit une décision manuelle, annulable par Ctrl+Z.

---

## 5. M : le miroir

Avec **« Miroir sur l'autre date (M) »** coché (panneau « Cible de
l'édition »), chaque classe appliquée (A, ou un clic dans la liste) l'est
aussi aux correspondants de la sélection sur l'autre date.

- **Le miroir vaut même quand on ne cible qu'une date.** C'est d'ailleurs
  son intérêt : on travaille sur t0, et t1 suit.
- **Il n'écrase jamais un label manuel** de l'autre date.
- **Sur la couche changement**, il ne reflète que « inchangé »,
  « transitoire » et « non observé ».
- La barre d'état donne le nombre de points reflétés. Le réglage est
  mémorisé d'une session à l'autre.

---

## 6. Le tampon : transférer aux voisines

Chaque tuile est entourée d'un **tampon**, une bande qui appartient au cœur
d'une tuile voisine. On y annote souvent pour finir un objet coupé par la
limite. Mais l'export ne garde que le cœur, et ce travail serait perdu.

**Révision ▸ Corrections d'ensemble ▸ « Transférer le tampon aux
voisines… »** l'envoie dans la tuile dont le cœur contient chaque point :

- **Ce qui part** : les labels **manuels** du tampon et les points écartés.
  Les pré-labels automatiques restent, et le cœur de la tuile ne part
  jamais.
- **La correspondance** se fait par identifiant de point (attribué à
  l'ingestion). Chaque point a exactement un cœur propriétaire.
- **Les voisines sont verrouillées** pendant la lecture et l'écriture. Une
  voisine ouverte ailleurs est laissée de côté, et le message la nomme.
- **Les conflits** : là où la voisine porte déjà, à la main, une autre
  classe, une fenêtre liste chaque désaccord (tuile, date, couche, ici,
  là-bas, nombre de points). Rien n'est coché d'office. Cochez les lignes où
  le tampon doit l'emporter ; les autres gardent leur label.

En ligne de commande, pour tout le projet ou une tuile :

```
cdforge transfer-buffer <projet> [--pair P] [--tile T] [--conflicts keep|overwrite]
```

Par défaut, `keep` laisse aux voisines leurs labels manuels.
