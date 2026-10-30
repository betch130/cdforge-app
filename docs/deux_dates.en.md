# Annotating both dates

A pair is two passes over the same place. The annotator shows both clouds
overlaid. It can take an object on both dates with a single gesture, carry
its labels over, and derive the change from them.

The underlying rule is the same everywhere: **nothing treats t0 and t1
differently**. Every operation works both ways. A t1 point with no
counterpart in t0 is “added” exactly as a t0 point with no counterpart in
t1 is “removed”.

---

## 1. Keys

| Key | Effect |
|---|---|
| **G**, **Shift+G** | with the “t0 + t1” target, SNAP takes the object on both dates, at the same 3D point |
| **T** | carries the selection to the other date, the whole object if SNAP is ready, and its labels with it |
| **K** | proposes the change of the selected object, from both dates (preview, then Apply) |
| **M** | turns the mirror on or off: every class applied is also applied on the other date |
| **Ctrl+Z** | undoes each of these operations, both dates at once |

The full list of shortcuts is under **F1**.

---

## 2. Counterparts

All these operations rest on the same question: **does this point have a
counterpart on the other date?**

- **The counterpart** is the nearest point of the other cloud, searched in
  the **whole** cloud, not among the selected points only. At the foot of a
  wall, the ground of the other date is closer to the wall than a wall
  point itself: searching among the selection would pass it off as wall.
- **The radius** is the “Growth radius” of the selection tools. It is raised
  to **1.5 times the point spacing** of the sparser cloud. Without that, a
  pass half as dense would leave gaps between its points, and those gaps
  would pass for change. The **K** preview says when the radius was raised.
- **Active filters** (unlabelled only, one class, extent, height above
  ground, section…) apply on the other date too.

---

## 3. T: carrying over to the other date

T works in three steps:

1. **The selection moves to the counterparts** on the other date.
2. **The whole object**, if the SNAP service has answered and the selection
   fits in its block. SNAP starts again from three well-spread clicks on the
   counterparts and takes the object on the other date. The two passes have
   neither the same density nor the same occlusions, and proximity alone
   would leave pieces behind. A polygon selection over a whole block of
   buildings is not “an object”: it stops at step 1.
3. **The labels follow.** Each counterpart gets the class of the nearest
   point of the selection. What SNAP found in addition (the part seen only
   on the other date) gets the object's majority class.

Never overwritten:

- what is **already manual** on the other date;
- what was **already selected** there: you put it there.

On the change layer, only “unchanged”, “transient” and “unobserved” carry
over. An “added” point exists only on its own date.

---

## 4. K: the object's change

Select the object on both dates (G with “t0 + t1”, or one date then T), then
press **K**.

| Point | has a counterpart | has none |
|---|---|---|
| of **t0** | unchanged | removed |
| of **t1** | unchanged | added |

What was “unobserved” stays so: the other date did not see this place, and
the lack of a counterpart proves nothing there.

**The preview** gives the counts per date before anything is written. It
warns in three cases:

- **the radius was raised** to the point spacing;
- **no change pre-label** covers the object. Without the pre-label's rays,
  nothing tells where the other date was blind: a point without a
  counterpart may be hidden there (occlusion, footprint) rather than
  removed;
- **manual change labels** are about to be replaced (their number is
  given).

It also says when nothing is selected on one of the dates. “Apply” writes a
manual decision, undoable with Ctrl+Z.

---

## 5. M: the mirror

With **“Mirror onto the other date (M)”** checked (“Edit target” panel), every
class applied (A, or a click in the list) is also applied to the
selection's counterparts on the other date.

- **The mirror works even when only one date is targeted.** That is the
  point of it: you work on t0, and t1 follows.
- **It never overwrites a manual label** on the other date.
- **On the change layer**, it only mirrors “unchanged”, “transient” and
  “unobserved”.
- The status bar gives the number of mirrored points. The setting is
  remembered from one session to the next.

---

## 6. The buffer: transferring to neighbours

Each tile is surrounded by a **buffer**, a band that belongs to the core of
a neighbouring tile. You often annotate there to finish an object cut by the
boundary. But the export keeps only the core, and that work would be lost.

**Review ▸ Tile-wide corrections ▸ “Transfer the buffer to neighbouring tiles…”**
sends it to the tile whose core holds each point:

- **What moves**: the buffer's **manual** labels and discarded points.
  Automatic pre-labels stay, and the tile's core never moves.
- **Matching** is by point identifier (assigned at ingestion). Each point
  has exactly one owning core.
- **Neighbours are locked** while they are read and written. A neighbour
  open elsewhere is skipped, and the message names it.
- **Conflicts**: where the neighbour already carries another class, set by
  hand, a window lists each disagreement (tile, date, layer, here, there,
  number of points). Nothing is ticked by default. Tick the rows where the
  buffer should win; the others keep their label.

From the command line, for the whole project or one tile:

```
cdforge transfer-buffer <project> [--pair P] [--tile T] [--conflicts keep|overwrite]
```

By default, `keep` leaves the neighbours their manual labels.
