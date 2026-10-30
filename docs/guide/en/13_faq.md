# Frequently asked questions

## Installation

### The AppImage does not start

It must be executable: `chmod +x cdforge-<version>-x86_64.AppImage`. On a distribution without FUSE 2 (the `libfuse2` package), run it with `--appimage-extract-and-run`.

### "The server certificate was refused"

A download (model weights, AI module) could not verify the server. cdforge never downloads without verification: a checkpoint is code that torch executes. Install your distribution's `ca-certificates` package, or point the `SSL_CERT_FILE` variable to a CA bundle. Behind a corporate proxy that rewrites certificates, it is that proxy's certificate you must provide.

### The Rust kernels are not installed

Registration, tiling and pre-labelling have two engines, Rust and Python, which give the same result. Without the `cdforge_native` module, the menus show "Auto — Python (Rust not installed)" and everything works, more slowly.

### Reading an E57 file

The AppImage reads it directly. Installed from source, cdforge needs the `pye57` package (`pip install pye57`); without it, ingestion fails with a message saying so.

## Data

### Everything, or nearly, is marked changed

Most often the two epochs are not registered: look at the NMAD and the rotation in the Registration stage. With C2C, also check the threshold — in automatic mode it includes the point spacing, but a fixed threshold set too low marks everything.

### A strip of "removed" along the edge of the survey

At the edge of its coverage, the other epoch has points on one side only. The **Covered sides** part of coverage (`obs_quadrants`, 3 by default) keeps those points from passing for removed; do not turn it off without reason. A sensor trajectory lets the rays decide.

### Many "unobserved" points

They are where the other epoch had no points facing them, and where nothing says it was looking: behind a parked car, under a tree. Without a trajectory, coverage cannot say more. With a trajectory attached to the source (Ingestion ▸ Catalogue ▸ Trajectory…), the rays tell "hidden" from "gone".

### A tile is missing from the map

It was **dropped** at tiling: one of the two epochs has fewer points there than **Minimum points**. The map marks it "Dropped at tiling", with its reason; lower the threshold and tile the pair again.

### Does tiling again lose my annotations?

No. Every point keeps the identifier given at ingestion, and labels set by hand follow it into the new tiles. **Delete all tiles**, however, also deletes the annotations.

## Performance

### Tiling or pre-labelling runs out of memory

Lower the number of **Tiles in parallel**. Tiling only starts a tile if free memory can hold it, but another program growing at the same time can catch it out.

### The disk fills up during tiling

Tiling writes intermediate fragments, erased at the end — about thirty bytes per point for both epochs of a pair. Choose a **Spill folder** on a disk with room, preferably an SSD.

### The annotator's 3D view is slow

Lower the **Detail** (View tab ▸ Points): the view then draws a subset of the visible points. On a machine without OpenGL, `cdforge gui --no-3d` opens the annotator with the plan and the section only.

### Inference is very slow

It is running on the processor: check that **Compute** is set to GPU, and that the AI module passed its trial on the card (Help ▸ AI module… ▸ Check).

## Annotation

### Are my changes saved?

Yes, at every edit, tile by tile: a crash only loses the gesture in progress. Ctrl+S forces a save.

### "Tile locked"

A tile open in an annotator is locked, so that two people do not edit it at the same time. After a crash the lock can remain: `cdforge locks` lists them, `cdforge locks clean` removes those whose process no longer exists.

### Why is this point "added"?

The **R** key on the point, in the 3D view: the decision is replayed step by step, with the rays drawn.

## And also

- The detail of the change decision: **Help ▸ How the pre-label decides on a change**.
- The guided demonstrations: **Help ▸ Interactive demos**.
- A complete test project: `cdforge demo`.
