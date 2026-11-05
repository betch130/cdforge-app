# Annotation

L'annotateur ouvre une tuile — ses deux époques — pour corriger les pré-labels et compléter ce qui manque, point par point. On l'ouvre avec **Ouvrir l'éditeur d'annotation**, sous la colonne quand l'étape Annotation est sélectionnée ou dans son onglet **Éditeur**, par un double-clic sur la carte des tuiles, ou avec `cdforge gui mon_projet`.

Chaque modification est enregistrée tuile par tuile au fil de l'édition : un arrêt brutal ne perd que le geste en cours. Chaque action est aussi écrite dans le journal de la tuile (qui, quand, quoi, combien de points), et chaque label posé à la main prend la provenance *manuel*.

## La fenêtre

![L'annotateur](img/annotator.png)

- **File de tuiles**, à gauche : toutes, à faire, terminées, récentes ; une recherche (Ctrl+F) ; un tri (par score, pour commencer par les tuiles les plus douteuses).
- **Les vues**, au centre : la vue **3D**, le **plan** (XY) et la **coupe verticale**. Les mêmes outils de sélection marchent dans les trois. En 3D, Ctrl + glisser fait tourner la vue quel que soit l'outil ; la molette zoome ; le bouton du milieu déplace.
- **Le panneau**, à droite, en trois onglets : **Étiqueter**, **Vue**, **Révision**.

## Étiqueter

**Outil de sélection** — polygone (Ctrl+1), lasso (Ctrl+2), pinceau (Ctrl+3), objet connexe (Ctrl+4), même classe (Ctrl+5), aucun — déplacer la vue (Ctrl+6). Le **mode**, dans la barre au-dessus des vues, dit ce que fait un geste : **Remplacer**, **Ajouter** ou **Retirer** la sélection ; **E** bascule en retrait et revient.

**Cible de l'édition** — la **couche** (sémantique ou changement, **L** pour basculer) et l'**époque**.

**Classes** — la liste de la couche active ; chaque classe affiche sa touche entre crochets, **A** applique la classe choisie à la sélection. **Touche de classe** règle ce que fait la touche : **Choisir, puis A** (par défaut) ou **Appliquer**, qui applique la classe dès la touche ; A applique dans les deux cas.

Les touches se règlent par projet. **Clic droit** sur une classe ▸ **Choisir une touche…** : un chiffre, Alt+chiffre, une lettre libre ou une touche de fonction, enregistrée dans `annotation_keys.yaml` à la racine du projet ; une touche déjà prise par la fenêtre (A, S, G…) est refusée. Les autres classes reçoivent une touche de l'appli, **1** à **0** puis **Alt+1** à **Alt+0** : d'abord les classes les plus posées à la main dans le projet, puis, à égalité, les classes de base avant leurs sous-classes. Sur un projet neuf, les dix classes de base ont donc les dix chiffres. Une classe garde sa touche tant qu'elle reste parmi les premières, et les touches ne bougent pas pendant qu'on annote : la distribution se refait à l'ouverture. **Touches des classes…** (même menu) les montre toutes, avec le nombre de gestes. Le filtre ne change pas les touches ; **Entrée** y prend la première classe trouvée. Une classe ajoutée ou modifiée dans **Projet ▸ Classes du projet** apparaît dans la liste sans rouvrir l'annotateur.

**Sur la sélection** — appliquer la classe (A), sélectionner toute la classe (S), élargir ou rétrécir (+ / −), propager à l'autre époque (T), changement de l'objet (K), valider auto → manuel (V), incertain (I), non étiqueté (U), écarter des points du jeu de données (Suppr) et les restaurer (Maj+Suppr), vider la sélection (Échap), annuler et rétablir.

Un point **écarté** (bruit, oiseau, point fantôme) n'est pas une classe de plus : il reste sur le disque, hors du jeu de données, et se restaure.

### Annoter les deux dates

Une tuile a deux époques, et un objet qui a changé se lit sur les deux. **T** reporte la sélection et sa classe sur les points correspondants de l'autre date ; **K** montre le changement de l'objet sélectionné ; **M** sélectionne le miroir sur l'autre date. Le détail : **Aide ▸ [Annoter les deux dates](../deux_dates.md)**.

### L'assistance SNAP

Avec le module IA (voir [Modèles sémantiques](07_ia.md#le-module-ia)), **Démarrer** lance le service SNAP. Ensuite, dans la vue 3D :

- **G** segmente l'objet sous le curseur et propose sa classe ;
- **Maj+G** ajoute un clic au même objet, pour l'affiner ;
- **Alt+G** retire du masque l'objet sous le curseur ;
- **Ctrl+G** cherche une catégorie par son nom (« pole », « vegetation »…) ;
- **[** et **]** élargissent ou resserrent le masque.

Huit modèles, pour trois domaines — extérieur, intérieur (avec la couleur), aérien. Le détail : **Aide ▸ [Assistance sémantique SNAP](../assistance_snap.md)**.

### Le sol par simulation de tissu

**Classer le sol — CSF…** (Ctrl+Maj+G) ouvre une fenêtre où un drap tombe sur une coupe du nuage : on y voit l'effet de chaque réglage, puis on l'applique à la tuile. Le réglage retenu sert ensuite au pré-label des autres tuiles.

**P** fait croître une région depuis le point sous le curseur.

## Vue

- **Époques et couleurs** — t0, t1 ou les deux ; Espace bascule, Ctrl+B fait clignoter t0 ↔ t1. Colorer par changement, sémantique (fine ou en grandes classes), RGB du relevé, intensité, hauteur, distance t0 ↔ t1, provenance des labels, points incertains, ou points requalifiés par les rayons.
- **Classes affichées** — **H** masque ou réaffiche la classe active, **Maj+H** réaffiche tout.
- **Clipping** — bornes basse et haute, par exemple pour voir sous les arbres.
- **Capteur et rayons** — la trajectoire du capteur, et les rayons rejoués par **R**.
- **Points** — taille (Ctrl++ / Ctrl+−), détail, réinitialiser la caméra.

La **coupe** se trace avec **C**, puis en glissant dans le plan ; **,** et **.** la décalent d'une largeur. La vue de coupe n'existe pas avant : elle s'ouvre à côté du plan au premier tracé, puis suit d'une tuile à l'autre, à la même place dans la tuile.

## Révision

- **Progression de la tuile** — ce qui est décidé, ce qui reste.
- **À vérifier** — sélectionner les points auto-remplis, non étiquetés, incertains, ou requalifiés par les rayons.
- **Corrections d'ensemble** :
  - **Changements temporaires…** — requalifie en *transient* les points ajoutés ou retirés dont la classe est temporaire (véhicule, personne…) ; c'est ici, et non au pré-label, que cela se décide ;
  - **Remplacer une classe…** — sur toute la tuile (pour tout le projet : **Projet ▸ Classes du projet ▸ Fusionner dans…**) ;
  - **Transférer le tampon aux voisines…** — ce qui a été étiqueté à la main dans le tampon rejoint la tuile où ces points sont au cœur, puisque l'export ne garde que les cœurs.
- **Afficher seulement** — des filtres : non étiquetés, labels automatiques, incertains, classe active, cœur seulement, changements mesurés ; **Réinitialiser les filtres**.
- **Note sur la tuile**.
- **Fin de tuile** — enregistrer (Ctrl+S), marquer la tuile terminée, **terminé et tuile suivante** (Ctrl+D), rouvrir, rejeter.

Une tuile passe par les statuts *tuilée → pré-labellisée → en cours → terminée* (ou *rejetée*). Le pré-label ne retouche jamais une tuile en cours, terminée ou rejetée : un rejet est une décision, comme une validation.

## Comprendre un label : la touche R

Sur un point de la vue 3D, **R** rejoue la décision de changement : la mesure, le seuil, la couverture, les rayons du capteur dessinés dans la 3D et le plan, et l'étape qui a tranché.

## Ce que sait un point : Ctrl+I

**Ctrl+I** ouvre le panneau **Point sous le curseur**. Il suit la souris dans la 3D, le plan ou la coupe, et dit ce que la tuile sait du point visé :
- l'époque, les coordonnées X, Y, Z, et si le point est dans le cœur ou le tampon ;
- sa classe sémantique et sa classe de changement, chacune avec sa source (pré-label, manuel, vérité terrain), sa confiance, et s'il est à revoir ;
- la distance à l'autre époque, le seuil de détection, si l'écart est significatif, et ce qu'en ont dit les rayons ;
- tous les autres attributs du relevé : intensité, couleur, numéro de bande, temps GPS…

Le point visé est cerclé. Ctrl+I referme le panneau.

## Les raccourcis

**F1** dans l'annotateur affiche la table complète, lue dans le programme lui-même.

| Touche | Action |
|---|---|
| 1…9, 0 | choisir la classe |
| A | appliquer la classe |
| S | sélectionner toute la classe |
| V | valider auto → manuel |
| U | non étiqueté |
| I | marquer incertain |
| T | propager à l'autre époque |
| K | changement de l'objet |
| M | miroir sur l'autre date |
| + / − | élargir / rétrécir la sélection |
| Échap | vider la sélection |
| E | mode retrait |
| Suppr / Maj+Suppr | écarter / restaurer les points |
| Ctrl+1…6 | outil de sélection |
| L | changer de couche |
| Espace / Ctrl+B | basculer d'époque / clignotement |
| H / Maj+H | masquer la classe active / tout réafficher |
| C | tracer la coupe |
| , / . | décaler la coupe |
| P | croissance de région |
| R | expliquer la décision de changement |
| Ctrl+I | informations du point sous le curseur |
| G, Maj+G, Alt+G, Ctrl+G | assistance SNAP |
| [ / ] | seuil SNAP |
| Ctrl+Maj+G | sol par simulation de tissu |
| Ctrl++ / Ctrl+− | taille des points |
| Ctrl+Z / Ctrl+Y | annuler / rétablir |
| Ctrl+S | enregistrer |
| Ctrl+D | terminer la tuile et ouvrir la suivante |
| N / B | tuile suivante / précédente |
| Ctrl+F | rechercher une tuile |
| Entrée | fermer le polygone |
| F1 | l'aide |

Étape suivante : [Splits](09_splits.md).
