# SNAP semantic assistance

SNAP segments the object you point at with a click **and proposes what it is**.
In the annotator, that name goes through the project's class list: the
matching class is preselected, and all that is left is to press **A**.

The name is a **proposal**. Nothing is applied without a gesture from the
annotator.

---

## 1. Gestures

| Key | Effect |
|---|---|
| **G** | new object: segments the object under the cursor (3D view) |
| **Shift+G** | adds a click to the **same** object: the mask is recomputed with all the clicks (ten at most) |
| **Alt+G** | removes the object under the cursor from the mask (the neighbour it spills onto) |
| **[** and **]** | lowers or raises the mask threshold, without a new request |
| **Ctrl+G** | searches a category by its name (“Pole”, “Vegetation”…) in the block around the cursor |
| **A** | applies the preselected class |

**With the “t0 + t1” target** (“Edit target” panel, the default), G and
Shift+G segment the object **on both dates**, at the same 3D point: each
click is carried to the nearest point of the other cloud, if it is closer
than 0.5 m (or two selection radii). Further away, the object is not there —
moved, demolished, not built yet — and the status line says so (“absent in
t1”). This costs two requests instead of one; with a single date targeted,
the other is left alone. What comes next — carrying over, deciding the
change, mirroring classes — is in the document “Annotating both dates”.

The mask of a new click **replaces** that of the previous click. The
selection that existed before the first click is kept. The clicks of the
current object are drawn in the 3D view: **cyan** for the object,
**orange** for removals.

**The view stays free while computing.** The request leaves the interface
thread: you rotate the scene, you aim at what comes next. A new G replaces
the one still waiting for its turn; only the last one is shown. The
“⏳ segmentation…” line in the panel says an answer is on its way.

A mask that stops too early is completed with **Shift+G** on the missing
part. Measured on a Boreas tree, the second click raises the predicted IoU
from 0.26 to 0.96.

A mask that **spills over** can be fixed in three ways:

- **the threshold** (panel, or **[** **]**): the service returns the
  probability of each point, and the mask tightens (above 0.5) or widens
  (below) before your eyes;
- **“only what touches the clicks”** (checked by default): pieces detached
  from the clicked object go away — another pole, the hedge across the
  street, which the model sometimes lights up because they look alike;
- **Alt+G** on the neighbour. SNAP has no negative click: it was only
  trained with clicks *on* the object. So the neighbour is segmented, and
  removed from the mask.

---

## 2. Eight models, three domains

The eight checkpoints published by the authors. Each is downloaded the first
time the service that uses it starts, and its SHA-256 fingerprint is checked
before it is loaded.

| | Trained on | Domains |
|---|---|---|
| **SNAP-C** | ScanNet, HM3D, SemanticKITTI, nuScenes, Pandaset, STPLS3D, DALES | all three — main model, recommended by the authors |
| **SNAP-Outdoor** | SemanticKITTI, nuScenes, Pandaset | road |
| **SNAP-C, variable grid** | the 7 datasets of SNAP-C, randomly drawn voxel sizes | all three — recommended when the grid differs |
| **SNAP-C, without HM3D** | the datasets of SNAP-C minus HM3D | all three |
| **SNAP-KITTI** | SemanticKITTI only | road |
| **SNAP-Aerial** | STPLS3D (drone photogrammetry), DALES (airborne lidar) | aerial |
| **SNAP-Indoor** | ScanNet, HM3D | indoor |
| **SNAP-ScanNet** | ScanNet only | indoor |

**The domain** is chosen under the model. It decides three things, taken from
the authors' code:

| Domain | Voxels | What the model receives | Proposed names |
|---|---|---|---|
| **Road** (MLS) | 5 cm | xyz, local ground at −1.73 m like a car lidar | the 19 SemanticKITTI classes |
| **Indoor** | 2 cm | xyz, colour, normals computed on the block | the 200 ScanNet200 classes |
| **Aerial** (ALS, ULS) | 33 cm | xyz, colour | the STPLS3D and DALES classes |

Indoor and aerial were trained **with colour**. A tile without RGB is sent to
them in neutral grey, and the status line says so (“without colour”). Only
the road names have a verified translation to the project's classes;
elsewhere, the proposed name goes through name similarity (see section 4).

**Compare with…** runs the same object again, with the same clicks, on
another model of the same domain. Both lines are shown side by side, each with its mask, and
**Take** keeps the one you prefer.

What was observed on two Boreas tiles:

- **Masks of isolated objects** (trees, furniture) are clean with both
  models.
- **Names diverge**, both ways:
  - SNAP-C called a mass of tree crowns “Building” that SNAP-Outdoor
    named “Vegetation”;
  - SNAP-Outdoor called the road “Vegetation” or “Sidewalk”, where SNAP-C
    said “Road”.

Hence the comparison, and hence the rule: a name is never applied on its own.

---

## 3. What a result line shows

`SNAP-C: Vegetation 0.43 → vegetation`
`13,225 points · predicted IoU 0.26 · same name`

- **Vegetation 0.43**: the name in the model's vocabulary, and its
  confidence (CLIP cosine).
  - **Below 0.30, no class is proposed.** On Boreas, spurious names cluster
    between 0.18 and 0.23 (almost always “Other_Ground”), sensible names
    between 0.33 and 0.46.
- **→ vegetation**: the project class chosen.
- **Predicted IoU**: the quality the model gives its own mask. Below 0.5, a
  second click (Shift+G) is often worth it.
- **The last mention** says how the name was translated (see section 4).

---

## 4. From the model's name to the project's class

SNAP's outdoor vocabulary is the 19 SemanticKITTI classes, in the exact
order of cdforge's `semantickitti19` scheme. The translation follows three
steps:

1. **The name itself**, if it is a class, an alias or a group of the
   project. This is the case for a project that kept a model's classes.
2. **The verified scheme** `semantickitti19`. It knows that “Trunk” is a
   tree (`high_vegetation`), that a “Bicyclist” is a person and that
   “Terrain” is natural ground.
3. **The fallback**: a class missing from the project goes to the base of
   its group. With the 10 base classes, “Car” becomes **vehicle**, and the
   line says so (“folded onto…”).

If no step succeeds, no class is chosen and the line says so.

---

## 5. Starting the service

The model runs in **another Python interpreter**, Pointcept's (torch,
spconv, flash-attn, clip). cdforge does not depend on it.

The “Semantic assistance (SNAP)” panel of the Label tab:

- **Start** launches the service and stops it with the window. The first
  load takes 5 to 10 s.
- **Settings…** sets the interpreter and the cloned SNAP repository
  (`git clone https://github.com/neu-vi/SNAP`). Both are suggested
  automatically when they are found.
- **Missing checkpoints** are downloaded from the official repository links
  (about 200 MB each), into `~/.cache/cdforge/models/snap/`. Their
  fingerprint is checked: a checkpoint is a pickle, and only the authors'
  file is loaded.
- **The service log** is `~/.cache/cdforge/models/snap/service.log`.

You can also start the service yourself. The annotator finds it at the
address `CDFORGE_ASSIST_URL`, by default `http://127.0.0.1:8765`:

```
<Pointcept python> cdforge/annotate/snap_server.py --repo ~/SNAP \
    --checkpoint snap-c=SNAP_C.pth --checkpoint snap-outdoor=SNAP_Outdoor.pth
```

---

## 6. How the model sees the tile

The settings below come from the SNAP paper and its training files, not
from intuition.

- **A block around the click, not the whole tile.**
  - On accumulated KITTI-360 (the dataset closest to an MLS tile), SNAP-C
    drops to 23.1 IoU at the first click on the whole scene, against 65.6 on
    crops centred on the object.
  - The **radius** is set in the panel (20 m by default).
  - The service caps the block at 400,000 voxels of 5 cm, which gives 13 to
    15 m on a dense Boreas tile: beyond that, up to 40 GB of graphics memory
    were needed.
  - **A mask touching the edge of the block** (more than 2 % of its voxels
    within one metre of the edge) is extended once: a second block, centred
    on the mask, is encoded, and each point keeps the higher of the two
    probabilities. The status line says so (“extended beyond the block”).
    On three dense Boreas tiles, 7 clicks out of 24 needed it.
  - **An object much larger than the block stays cut**: a road, a long
    façade. Region growing finishes them better.
- **A sensor's frame** (road domain). The block is recentred on the click,
  the local ground placed at −1.73 m, like the KITTI lidar. With the ground at 0,
  names more often became “Other_Ground”.
- **Indoor and aerial**: the block is recentred as in the authors' code
  (centre of the footprint, lowest point at zero). A trial on a synthetic
  6 × 6 m room with a table: indoors, SNAP-C takes the whole table (IoU 0.99)
  and names it “table”; without colour, the IoU drops to 0.93.
- **No intensity.** Every outdoor training configuration removes it.
- **Working memory.**
  - The tile is sent to the service **as soon as it opens**, in the
    background: the first click no longer carries its millions of points.
  - A new block is encoded in 0.4 to 0.6 s (median 0.59 s over 24 clicks on
    three dense Boreas tiles; it was about 1.1 s before voxelisation went
    from a row-by-row sort to an integer key).
  - Following clicks in the same block take about 0.05 s.
- **Search by name** (Ctrl+G) seeds clicks every 2 m in the block. It keeps
  the masks whose name is the one asked for and no longer seeds where a
  mask has already passed. It takes about 4 s.

---

## 7. Limitations

- **SNAP has never seen dense MLS**: neither Boreas nor any comparable
  survey. The paper's measurements are on other datasets.
- **The observations in this document** come from two Boreas tiles, without
  ground truth: no labelled MLS survey was available for a quantified
  measurement.
- **It only knows 19 names** on the road. An object outside this vocabulary
  (street furniture, construction site) gets the closest name, or none if it
  is below the threshold.
- **It only names semantics.** For change, it takes the object on both
  dates (G with “t0 + t1”); the decision then comes from comparing the two
  (key **K**, document “Annotating both dates”).
