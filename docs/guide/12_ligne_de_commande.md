# La ligne de commande

Tout ce que fait le studio passe par la commande `cdforge` — le studio lui-même la lance pour chaque étape. Elle sert aussi seule : sur un serveur, dans un script, pour reprendre un traitement.

Le dossier du projet (celui qui contient `project.yaml`) est le dernier argument, et vaut `.` par défaut. `cdforge <commande> --help` donne toutes les options.

## Le pipeline

| Commande | Ce qu'elle fait | Options utiles |
|---|---|---|
| `cdforge init DOSSIER` | crée un projet : `project.yaml` commenté, dossiers, 10 classes de base | |
| `cdforge validate` | vérifie `project.yaml`, les fichiers sources et les schémas de labels | |
| `cdforge ingest` | lit, rogne, uniformise les labels, écrit `work/ingested/*.laz` | `--source`, `--overwrite` |
| `cdforge align` | recale t1 sur t0 | `--pair`, `--overwrite`, `--implementation` |
| `cdforge tile` | découpe chaque paire en tuiles | `--pair`, `--size`, `--overlap`, `--min-points`, `--workers`, `--overwrite` |
| `cdforge prelabel` | pré-labels de changement et de sémantique | `--pair`, `--tile`, `--workers`, `--implementation` |
| `cdforge split` | répartition train/val/test sans fuite | |
| `cdforge export` | écrit le jeu de données | `--format laz` ou `pointcept` |
| `cdforge report` | écrit `reports/report.html` | `--fast` (sans relire les tuiles) |
| `cdforge run` | enchaîne tout, de l'ingestion au rapport, en sautant ce qui est à jour | `--workers`, `--resume`, `--overwrite` |

`--pair` et `--tile` se répètent pour en traiter plusieurs. `--implementation` choisit les noyaux Rust (`rust`), Python (`python`) ou Rust s'il est installé (`auto`, le défaut) ; le résultat n'en dépend pas.

`cdforge run --resume` reprend une exécution interrompue (plantage, Ctrl+C) sans refaire les tuiles déjà terminées.

## Les modèles sémantiques

| Commande | Ce qu'elle fait |
|---|---|
| `cdforge models` | le catalogue ; `--model NOM` pour le détail d'un modèle |
| `cdforge fetch-weights MODÈLE` | télécharge les poids et vérifie leur empreinte |
| `cdforge stage-inference MODÈLE` | prépare les tuiles pour le modèle et affiche la commande d'inférence (`--python` : l'interpréteur où torch est installé) |
| `cdforge import-predictions DOSSIER` | importe des prédictions `<tuile>/<époque>_semantic.npy` comme pré-labels (`--scheme`, `--min-conf`, `--keep-low-conf`, `--reset-auto`) |

## L'annotation

| Commande | Ce qu'elle fait |
|---|---|
| `cdforge studio` | ouvre cdforge Studio |
| `cdforge gui` | ouvre l'annotateur directement (`--pair`, `--tile`, `--no-3d` sur une machine sans OpenGL) |
| `cdforge transfer-buffer` | rend au cœur des tuiles voisines ce qui a été étiqueté à la main dans les tampons (`--conflicts keep` ou `overwrite`) |
| `cdforge locks` | liste les verrous de tuiles (une tuile ouverte dans un annotateur est verrouillée) ; `cdforge locks clean` retire ceux qu'un arrêt brutal a laissés |

## Suivre et vérifier

| Commande | Ce qu'elle fait |
|---|---|
| `cdforge status` | l'avancement des tuiles, par paire |
| `cdforge dag` | l'enchaînement des étapes (`--show` pour le graphe) |
| `cdforge history FICHIER -p PROJET` | la chaîne de traitements qui a produit un fichier |
| `cdforge evaluate` | compare un champ à une vérité terrain (`--gt-field`, `--pred-field`) |
| `cdforge merge` | réassemble les tuiles d'une paire en un nuage t0 et un nuage t1 complets (`--no-core-only` pour garder les tampons) |
| `cdforge migrate` | met un projet au format courant (`--dry-run`, sauvegarde par défaut) |

## Autres

| Commande | Ce qu'elle fait |
|---|---|
| `cdforge demo [DOSSIER]` | crée un projet synthétique aux changements connus et lance tout le pipeline |
| `cdforge install-launcher` | ajoute cdforge Studio au menu des applications (`--uninstall` pour le retirer) |

## Variables d'environnement

| Variable | Rôle |
|---|---|
| `CDFORGE_LANG` | la langue (`fr`, `en`), avant le réglage du studio |
| `CDFORGE_TORCH_PYTHON` | l'interpréteur avec torch à utiliser pour les modèles, avant le module IA |
| `CDFORGE_ALIGN_TMPDIR` | un dossier temporaire pour le recalage, sur un disque qui a la place |
| `CDFORGE_ALIGN_RAM_MB` | la mémoire retenue par magasin de voxels du recalage (0 : tout sur disque) |
| `CDFORGE_ALIGN_IMPLEMENTATION`, `CDFORGE_TILING_IMPLEMENTATION`, `CDFORGE_PRELABEL_IMPLEMENTATION` | le moteur par défaut de chaque étape |
| `CDFORGE_PRELABEL_WORKERS` | les tuiles pré-labellisées en parallèle |

Pour finir : [Questions fréquentes](13_faq.md).
