# cdforge

Préparation de données pour la **détection de changements 3D urbains** à partir de paires de nuages LiDAR multi-dates (MLS, ULS, ALS) : ingestion, recalage, tuilage, pré-label, annotation, découpage et export vers Pointcept.

Ce dépôt ne contient que les **versions construites** de l'application pour Linux. *English below.*

## Installer

1. Télécharger `cdforge-<version>-x86_64.AppImage` dans les [publications](https://github.com/betch130/cdforge-app/releases/latest).
2. Le rendre exécutable, une seule fois (le navigateur retire ce droit au téléchargement) :
   clic droit ▸ Propriétés ▸ « Exécuter comme un programme », ou
   ```bash
   chmod +x cdforge-*-x86_64.AppImage
   ```
3. Double-cliquer dessus : le studio s'ouvre. Rien d'autre à installer.

Pour l'ajouter au menu des applications (le lanceur suit l'emplacement du fichier, ne plus le déplacer ensuite) :
```bash
./cdforge-*-x86_64.AppImage install-launcher
```

En ligne de commande, l'AppImage accepte toutes les commandes de `cdforge` :
```bash
./cdforge-*-x86_64.AppImage --help
./cdforge-*-x86_64.AppImage ~/mon/projet     # ouvrir le studio sur un projet
```

Vérifier le téléchargement : `sha256sum -c cdforge-*-x86_64.AppImage.sha256`

## Guide de l'utilisateur

Le [guide](docs/README.md), en français et en anglais, est aussi dans l'application : **F1**, ou le bouton **?** de chaque étape.

## Configuration requise

- Linux x86_64 avec glibc 2.34 ou plus récente : Ubuntu 22.04+, Debian 12+, Fedora 35+, Linux Mint 21+…
- OpenGL pour la vue 3D. Sous Wayland, l'application passe par XWayland (présent par défaut sur GNOME et KDE).
- Pas besoin de libfuse2.

## Module IA (facultatif)

Les modèles sémantiques (PTv3) et l'assistance à l'annotation (SNAP) demandent un module à part, que l'application télécharge et installe elle-même : **Aide ▸ Module IA…** Elle vérifie d'abord la carte, le pilote et la place disque.

- Téléchargement 2,8 Go, 9 Go une fois installé, dans `~/.local/share/cdforge/ia`.
- Carte NVIDIA nécessaire :

| Carte | Fonctionne | Pilote minimal |
|---|---|---|
| GTX 10xx et plus anciennes | non | — |
| RTX 20xx, GTX 16xx | oui, sans flash-attn (plus lent, plus de mémoire vidéo) | 525.60 |
| RTX 30xx/40xx, A-series, A100, H100 | oui | 525.60 |
| RTX 50xx, Blackwell | oui | 570 |

Sans le module, tout le reste de l'application fonctionne.

## Signaler un problème

Ouvrir une [issue](https://github.com/betch130/cdforge-app/issues) en indiquant la distribution, la carte graphique et ce qui s'affiche dans le panneau « Tâches ».

---

## English

Data preparation for **urban 3D change detection** from multi-date LiDAR point cloud pairs: ingestion, registration, tiling, pre-labelling, annotation, splitting and export to Pointcept. This repository only holds the **built Linux releases**.

**Install.** Download `cdforge-<version>-x86_64.AppImage` from the [releases](https://github.com/betch130/cdforge-app/releases/latest), make it executable once (`chmod +x cdforge-*-x86_64.AppImage`, or Properties ▸ “Allow executing file as program”), then double-click it. `install-launcher` adds it to the application menu. Any `cdforge` command works: `./cdforge-*-x86_64.AppImage --help`.

**User guide.** The [guide](docs/README.md) (French and English) is also in the app: **F1**, or the **?** button on each stage.

**Requirements.** Linux x86_64, glibc 2.34+ (Ubuntu 22.04+, Debian 12+, Fedora 35+), OpenGL. On Wayland it runs through XWayland. No libfuse2 needed. The interface is available in French and English.

**AI module (optional).** Semantic models (PTv3) and annotation assistance (SNAP) need a separate module that the app downloads and installs itself: **Help ▸ AI module…** 2.8 GB download, 9 GB installed. Requires an NVIDIA GPU, Turing (RTX 20xx) or newer; driver 525.60+, 570+ for Blackwell (RTX 50xx).

**Issues.** Please open an [issue](https://github.com/betch130/cdforge-app/issues) with your distribution, GPU and what the “Tasks” panel shows.

## Licence

cdforge © 2026 Betchaleel — tous droits réservés / all rights reserved. Voir [LICENSE](LICENSE).
Les composants tiers inclus gardent leur propre licence : voir [THIRD_PARTY.md](THIRD_PARTY.md).
