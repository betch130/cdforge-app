# Questions fréquentes

## Installation

### L'AppImage ne démarre pas

Elle doit être exécutable : `chmod +x cdforge-<version>-x86_64.AppImage`. Sur une distribution sans FUSE 2 (le paquet `libfuse2`), lancez-la avec `--appimage-extract-and-run`.

### « certificat du serveur refusé »

Un téléchargement (poids d'un modèle, module IA) n'a pas pu vérifier le serveur. cdforge ne télécharge jamais sans vérification : un point de contrôle est du code que torch exécute. Installez le paquet `ca-certificates` de votre distribution, ou désignez un fichier d'autorités par la variable `SSL_CERT_FILE`. Derrière un proxy d'entreprise qui réécrit les certificats, c'est le certificat de ce proxy qu'il faut fournir.

### Les noyaux Rust ne sont pas installés

Le recalage, le tuilage et le pré-label ont deux moteurs, Rust et Python, qui donnent le même résultat. Sans le module `cdforge_native`, les menus affichent « Auto — Python (Rust non installé) » et tout fonctionne, plus lentement.

### Lire un fichier E57

L'AppImage le lit directement. Installé depuis les sources, cdforge a besoin du paquet `pye57` (`pip install pye57`) ; sans lui, l'ingestion échoue en le disant.

## Données

### Tout est marqué changé, ou presque

Le plus souvent, les deux époques ne sont pas recalées : regardez la NMAD et la rotation dans l'étape Recalage. Avec C2C, vérifiez aussi le seuil — en automatique, il inclut l'espacement des points, mais un seuil fixé trop bas marque tout.

### Une bande de « supprimé » le long du bord du relevé

Au bord de sa couverture, l'autre époque n'a de points que d'un côté. La portion **Côtés couverts** de la couverture (`obs_quadrants`, 3 par défaut) évite que ces points passent pour disparus ; ne la coupez pas sans raison. Une trajectoire capteur permet aux rayons de trancher.

### Beaucoup de points « non observé »

Ils sont là où l'autre époque n'avait pas de points en face, et où rien ne dit qu'elle regardait : derrière une voiture garée, sous un arbre. Sans trajectoire, la couverture ne peut pas en dire plus. Avec une trajectoire associée à la source (Ingestion ▸ Catalogue ▸ Trajectoire…), les rayons départagent « caché » et « disparu ».

### Une tuile manque sur la carte

Elle a été **écartée** au tuilage : une des deux époques y a moins de points que **Points minimum**. La carte la marque « écartée au tuilage », avec son motif ; abaissez le seuil et retuilez la paire.

### Retuiler perd-il mes annotations ?

Non. Chaque point garde l'identifiant attribué à l'ingestion, et les labels posés à la main le suivent dans les nouvelles tuiles. **Effacer toutes les tuiles**, en revanche, supprime aussi les annotations.

## Performances

### Le tuilage ou le pré-label manque de mémoire

Baissez le nombre de **tuiles en parallèle**. Le tuilage ne lance une tuile que si la mémoire libre la tient, mais un autre programme qui grossit en même temps peut le surprendre.

### Le disque se remplit pendant le tuilage

Le tuilage écrit des fragments intermédiaires, effacés à la fin — une trentaine d'octets par point pour les deux époques d'une paire. Choisissez un **dossier du spill** sur un disque qui a la place, de préférence un SSD.

### La vue 3D de l'annotateur est lente

Réduisez le **Détail** (onglet Vue ▸ Points) : la vue tire alors un sous-ensemble des points visibles. Sur une machine sans OpenGL, `cdforge gui --no-3d` ouvre l'annotateur avec le plan et la coupe seulement.

### L'inférence est très lente

Elle tourne sur le processeur : vérifiez que **Calcul** est sur GPU, et que le module IA a réussi son essai sur la carte (Aide ▸ Module IA… ▸ Vérifier).

## Annotation

### Mes modifications sont-elles enregistrées ?

Oui, à chaque édition, tuile par tuile : un arrêt brutal ne perd que le geste en cours. Ctrl+S force l'enregistrement.

### « Tuile verrouillée »

Une tuile ouverte dans un annotateur est verrouillée, pour que deux personnes ne l'éditent pas en même temps. Après un arrêt brutal, le verrou peut rester : `cdforge locks` les liste, `cdforge locks clean` retire ceux dont le processus n'existe plus.

### Pourquoi ce point est-il « ajouté » ?

Touche **R** sur le point, dans la vue 3D : la décision est rejouée étape par étape, avec les rayons dessinés.

## Et encore

- Le détail de la décision de changement : **Aide ▸ Comment le pré-label décide d'un changement**.
- Les démonstrations guidées : **Aide ▸ Démos interactives**.
- Un projet d'essai complet : `cdforge demo`.
