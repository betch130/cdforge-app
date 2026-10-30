# How the pre-label decides on a change

This document describes what the code actually does (`cdforge/labels/prelabel.py`,
`cdforge/labels/visibility.py`, `cdforge/core/trajectory.py`). Each setting
mentioned carries its name in `project.yaml`, section `prelabel:`.

The pre-label does not replace annotation: it proposes, with a confidence, and
the annotator validates. Ground-truth labels and labels set by hand are
**never** overwritten.

---

## 1. Overview

A tile holds two epochs, **t0** and **t1**, already registered onto each
other (t1 received the registration matrix at tiling). The pre-label processes
each epoch in turn, against the other, **with exactly the same code**:

- t0 points are compared with t1 → “unchanged”, “removed”, “not observed”;
- t1 points are compared with t0 → “unchanged”, “added”, “not observed”.

Below, **A** is the epoch whose points are being labelled and **B** the other.

For each point **p** of A:

```
                       ┌───────────────────────────────┐
                       │ 1. Measure: does B have enough│
                       │    points facing p?           │
                       └──────────────┬────────────────┘
                     yes (n_B ≥ min_pts)    no (n_B < min_pts)
                 ┌─────────────┘                 └──────────────┐
   ┌─────────────▼─────────────┐          ┌───────────────────▼──────────────────┐
   │ 2. Distance |d| > thresh.?│          │ 3. Was B looking at this place?      │
   │  yes → added / removed    │          │  (coverage, then rays if a           │
   │  no  → unchanged          │          │   trajectory exists)                 │
   └───────────────────────────┘          │  yes → added / removed (disappeared) │
                                          │  no  → not observed (blind spot)     │
                                          └──────────────────────────────────────┘
                       ┌───────────────────────────────┐
                       │ 4. Small clusters → unchanged │
                       └───────────────────────────────┘
   “transient” is never decided here: the annotator proposes it.
```

Ray casting only comes in **at step 3**, and only for points that have no
support in the other epoch. Everywhere else, it changes nothing.

---

## 2. Step 1 — the measurement (simplified M3C2, or C2C)

### M3C2 (`change_method: m3c2`, default)

1. **Surface orientation**: the normal of p is estimated by PCA over its
   `k_normals` nearest neighbours in A.
2. **Cylinder**: a cylinder of radius `cyl_radius`, half-length `cyl_depth`,
   centred on p and oriented along the normal. It counts the points of A (n_A)
   and of B (n_B), at most `k_cyl` of each.
3. **Same surface** (`same_surface_angle`, `same_surface_planarity`): around
   a point whose neighbourhood is clearly planar, only neighbours whose normal
   is within `same_surface_angle` degrees of p's count.
   Without this rule, the cylinder of a ground point caught the wall of a new
   building or the neighbouring car, and declared changed a ground that had
   not moved.
4. **Distance**: d = mean of B − mean of A, along the normal.
5. **Level of detection (LoD95)**, after Lague et al. (2013):

   ```
   LoD95 = 1.96 × ( √(σ_A²/n_A + σ_B²/n_B) + registration error )
   ```

   σ_A and σ_B are the roughnesses in the cylinder. The registration error
   (`reg_error`) defaults to the NMAD residual of the pair's registration.

### C2C (`change_method: c2c`)

Distance to the nearest neighbour in B. No normal, faster, but no per-point
roughness: the threshold is the registration error **plus the median spacing
of B's points** (two samplings of the same surface never coincide, the
distance is already of the order of the spacing).

### What comes out of step 1

- **n_B ≥ `min_pts`**: B has enough points facing p to measure. Go to
  step 2.
- **n_B < `min_pts`**: B has (almost) nothing facing p. Two incompatible
  explanations are possible, and step 3 decides.

---

## 3. Step 2 — did the point move?

| Condition | Label | Confidence |
|---|---|---|
| \|d\| ≤ LoD95 | unchanged | 0.5 to 1 depending on the margin |
| \|d\| > LoD95 | added (t1 point) / removed (t0 point) | 0.5 to 1 depending on the margin |

---

## 4. Step 3 — was B looking at this place?

This is **the** hard question. A point of A with nothing facing it in B can
mean:

- **the object disappeared**: B was looking there and saw nothing → real change;
- **the object was hidden**: a parked truck, a tree, a wall masked the place
  when B passed → it is not a change, it is a blind spot.

Two methods answer, and can be combined.

### 4.1 Coverage (always computed)

It only looks at **where** B has points, not where it was looking from.

| Part | Switch | Setting | What it tests |
|---|---|---|---|
| Same ground cell | `obs_column` | `obs_cell` | B has a point in the same cell of a ground grid of `obs_cell` metres |
| Nearby point | `obs_nearby` | `obs_radius` | B has a point within `obs_radius` metres of p (in 3D) |
| Covered sides | `obs_sides` | `obs_quadrants`, `obs_radius` | B has points in at least `obs_quadrants` of the 4 quadrants around p |

```
coverage = (same cell  OR  nearby point)   — among the active parts
           AND covered sides               — if the part is active
```

If the first two parts are switched off, coverage always answers
“not observed”.

**Why covered sides.** At the edge of what B surveyed, B only has points on
one side. Without this rule, a band of false “removed” `obs_radius` wide
followed every edge of the survey (61,000 points on the measurement scene, no
real change lost by adding it). The test is done with four small queries: a
disc of radius `obs_radius/2` at the centre of each quadrant.

**The limit of coverage.** A parked truck hides a façade, but the truck itself
is indeed there: B has points very close, coverage concludes “observed”, hence
“disappeared”. That is why rays exist.

### 4.2 Rays (when B has a trajectory)

The question becomes geometric: **was the line between B's sensor and the
location of p clear?**

#### a) The trajectory

- The trajectory file of B's source (`trajectory` in the source) is a CSV.
  Recognised columns: `x / easting / x_utm…`, `y / northing / y_utm…`, and an
  altitude.
- **Several altitudes** (Boreas: `h_ellipsoidale` and `H_cgvd2013`, 37 m
  apart). The one **in the same system as the cloud** is needed. If
  `trajectory_z` is given in the source, it is that one. Otherwise cdforge
  recognises it from the cloud: the right one puts the sensor between 0.3 and
  6 m above the ground measured under it (5th percentile of the points within
  2 m). If none, or more than one, is plausible, **no ray is cast** on this
  tile rather than a wrong ray. On Boreas: `h_ellipsoidale`, sensor 2.1 m
  above the ground.
- **Viewpoints**: poses are thinned, one is only kept if it is more than
  `ray_pose_spacing` metres from the previous kept one. At 200 Hz,
  consecutive poses are a few centimetres apart: “the three closest” would
  otherwise designate the same place three times.
- **Registration**: t1 was moved by the registration matrix at tiling. Its
  trajectory undergoes **the same** transformation, otherwise every ray would
  aim beside the point.

#### b) The occupancy grid

B's points are stored in a grid of voxels (cubes) of `ray_cell` metres. A
voxel is “occupied” if it contains at least one point of B. No mesh: a cloud
has no surfaces.

#### c) The ray

For point p:

1. Take the `ray_viewpoints` viewpoints closest to p, within
   `ray_max_range` metres.
2. For each, step along the sensor → p segment, by steps of `ray_cell`.
3. **Skip** the first `ray_min_range` metres (the “ignore the vehicle” part,
   switch `ray_skip_vehicle`): the platform always sees itself.
4. **Skip** the last `ray_clearance` metres (the “ignore the point's surface”
   part, switch `ray_skip_target`): a surface *at* the point must not count
   as an obstacle *in front of* it.
5. If a sample falls into an occupied voxel, the line is **cut**.

```
 sensor ●──────────────────────────────●──────────────● p
        │← ray_min_range →│   step of ray_cell     │← ray_clearance →│
        (ignored)          ■ occupied voxel?        (ignored)
```

- **At least one clear line** → B was looking there → **observed**.
- **All lines cut** → p was hidden → **not observed**.
- **No viewpoint in range** → rays know nothing (“undecided”).

#### d) What is approximated

- An occupied voxel is not necessarily opaque: sparse foliage counts as a
  wall.
- With a finite step, a very thin obstacle can slip between two samples.

Both errors lean towards “observed”, hence “disappeared” — the same direction
as coverage alone. Rays correct part of the cases, they make none worse.

The Python and Rust implementations give **the same answers to the bit**
(same sampling); Rust merely stops each ray at its first obstacle.

### 4.3 Combining the two

`visibility` chooses the principle:

- `coverage`: coverage alone, no rays;
- `auto` (default): rays where B has a trajectory, coverage elsewhere.
  (`rays` is accepted and behaves like `auto`.)

When rays are active, four parts decide their weight:

| Situation | Switch | Enabled (default) | Off |
|---|---|---|---|
| Rays: clear, coverage: no | `ray_reveal` | observed → **disappeared** | coverage keeps “not observed” |
| Rays: cut, coverage: yes | `ray_hide` | **not observed** (hidden) | coverage keeps “observed” |
| Rays and coverage agree | — | their answer | their answer |
| No viewpoint in range | `ray_fallback` | coverage answers | **not observed** |

Switching off `ray_reveal` and `ray_hide` together amounts to computing rays
to show them in the annotator without them changing any label.

### 4.4 The resulting label

| Was B looking there? | Label | Confidence |
|---|---|---|
| yes | added / removed (disappearance) | 0.8 |
| no | not observed | 0.5 |

---

## 5. Step 4 — cleaning, and what is left to the annotator

- **Small clusters**: changed points are grouped (two points within
  `cluster_radius` are in the same cluster). A cluster of fewer than
  `min_cluster` points is reset to “unchanged” (confidence 0.3): a real change
  has an extent, an isolated point is noise. `min_cluster: 1` disables this
  filter.
- **Transients**: the pre-label does not decide them. A vehicle parked at t0
  and gone at t1 therefore comes out as “removed”. In the annotator,
  **“Transient changes…”** lists the project's transient groups
  (`transient_classes`, set in the class editor: vehicle, person by default),
  counts the added/removed points carrying their class on the tile, and
  requalifies them as “transient” if you confirm (undoable).

---

## 6. Seeing a decision in the annotator

Aim at a point and press **R** (“Explain the decision”). The annotator
**replays** the decision on the open tile with **the same function** as the
pipeline (`change_prelabel`, inputs prepared by `change_inputs`) and the
current settings of `project.yaml`. The first R on a tile takes the time of a
pre-label of that tile (the normals of both epochs); the following ones are
immediate.

The **“Why this label?”** panel (in a tab next to “Annotation”) details each
step for this point, the deciding step in bold:

- 1 — how many points of the other epoch face it (`n_B`, threshold `min_pts`);
- 2 — the distance `|d|` against the LoD95 threshold, if it could be measured;
- 3a/3b/3c — each part of the coverage: yes, no, or “part switched off” if
  you disabled it;
- 3d — the rays: how many tested, clear or cut; then the rule that combined
  rays and coverage (`ray_reveal`, `ray_hide`, `ray_fallback`);
- 4 — small cluster reset to “unchanged”.

The 3D view shows, **on top of the cloud** (a cut ray goes right through the
points that cut it):

| Drawing | Meaning |
|---|---|
| yellow sphere | the explained point |
| blue sphere | a position of the other epoch's sensor (a viewpoint) |
| thick green line | clear ray: the other epoch saw the point from there |
| thick then thin red line, red sphere | cut ray: thick up to the first occupied voxel (the obstacle), thin beyond |
| grey line | ignored segment (`ray_min_range` on the sensor side, `ray_clearance` on the point side); viewpoint not tried because a previous one already saw the point; or ray shorter than both segments combined (nothing to test: counted clear, the panel says so) |
| yellow square | the ground cell (`obs_cell`) of the “same cell” part |
| yellow circle | the `obs_radius` disc of the “nearby point” part |
| four small circles | the sides tested: green, the other epoch has points there; grey, empty |

Coverage and rays are only drawn for points **without a measurement**:
elsewhere they had no weight. The drawing follows display changes (t0, t1,
t0 + t1); “Clear the drawing” removes it. In “side by side” mode the epochs
are shifted on screen: switch back to “t0 + t1”.

The rays are also projected in the **plan** (same colours), where you can see
which side they come from and what they cross.

**Sensor trajectories** (View tab, “Sensor and rays” group, on by default):
where each epoch's sensor passed around the tile — t0 in orange, t1 in blue —
in the 3D view and the plan, with the viewpoints the rays start from (one
every `ray_pose_spacing` metres). Only the part within `ray_max_range` of the
tile is drawn: the part the rays use. The t1 trajectory is registered like
its cloud. The group says, per epoch, how many viewpoints are in reach, or why
there are none (source without a trajectory, unreadable file, trajectory too
far). The framing does not move: the view stays on the tile. In the
**section**, a triangle in the epoch's colour marks each pass of the sensor
across the section line, at its height: you can see whether it rides above
the road, and what surrounds it.

**Lidar beams** (same group): where each point was **actually** measured from
— a line from the lidar position at the time of the shot to the point, for
every visible point, in the epoch's colour — narrow a slice (clipping) to
see clearly. Voxelised Boreas
exports no longer have GPS time (`gps_time` at 0), but each point keeps in
`point_source_id` the number of the scan that measured it, and the trajectory
has one row per kept scan (one in two): the scan's pose is row
`point_source_id / 2`. Checked: every point is then within 160 m of its pose,
the range set at export; half come from more than 27 m. A voxelised cloud
keeps one scan per voxel: many points were seen from afar, along the road.

**Pre-label lines of sight** (same group): these are **not** measured beams.
It is the line the pre-label tests between a point and the nearest viewpoints
of the **other** epoch, to know whether the other epoch could see it. For
every visible point without a measurement — the only ones rays decide. Green: clear; red: blocked, drawn up
to the obstacle. “Show” keeps all lines, the blocked ones or the clear ones.
They follow clipping and hidden classes. The verdicts are those of the
replayed decision (as with R).

If the label stored in the tile differs from the replayed label, the panel
says so. Three possible reasons: the tile was pre-labelled with other
settings; a label was set by hand; the semantic class changed since (semantic
transfer, correction), which changes step 5.

---

### Spotting the points rays requalified

The pre-label writes, for each point, whether rays **changed** the coverage's
answer (the tile's `chg_ray` field):

- **seen** (orange): coverage said “not observed”, a clear line says
  otherwise → the point becomes added/removed;
- **hidden** (blue): coverage said “observed”, all lines are cut → the point
  becomes not observed.

In the annotator:

- **“rays”** colour: grey nothing, orange seen, blue hidden;
- Review ▸ **Progress**: “rays: N requalified”, per epoch;
- Review ▸ **Select the requalified (rays)**;
- in the tile queue, the **“rays”** badge marks the tiles where it happens;
- R on one of these points shows why.

A protected label (set by hand, ground truth) is never marked: rays did not
decide it.

---

## 7. What the report says

For each tile and each epoch, `visibility` in the pre-label report gives:

- `method`: `rays` or `coverage`;
- `n_rays`: points decided by rays;
- `n_coverage`: points decided by coverage;
- `n_disagree`: points where rays **changed** the coverage's answer — the
  number that says whether rays were useful;
- `n_revealed`, `n_hidden`: that same number, split into “seen” and “hidden”.

An unreadable trajectory is reported once per source, in the run log
(“trajectory of … unreadable, rays disabled for this source”). The
Pre-labels ▸ Configure screen shows “Readable trajectories: n of N sources”.

---

## 8. Settings summary

| Name | Default | Role |
|---|---|---|
| `change_method` | m3c2 | simplified M3C2 or C2C |
| `k_normals` | 20 | neighbours for the normal |
| `cyl_radius`, `cyl_depth` | 0.5 m, 3 m | measurement cylinder |
| `k_cyl` | 48 | points at most per cylinder |
| `same_surface_angle`, `same_surface_planarity` | 45°, 0.5 | “same surface” rule |
| `min_pts` | 3 | below it, B “has nothing facing” |
| `reg_error` | auto | registration uncertainty in the threshold |
| `obs_column`, `obs_cell` | yes, 1 m | “same ground cell” part |
| `obs_nearby`, `obs_radius` | yes, 2 m | “nearby point” part |
| `obs_sides`, `obs_quadrants` | yes, 3/4 | “covered sides” part |
| `visibility` | auto | rays when a trajectory exists |
| `ray_cell` | 0.5 m | voxels and ray step |
| `ray_viewpoints` | 3 | viewpoints tried |
| `ray_pose_spacing` | 2 m | spacing of the viewpoints |
| `ray_max_range` | 40 m | range of a viewpoint |
| `ray_skip_vehicle`, `ray_min_range` | yes, 2 m | ignore the vehicle |
| `ray_skip_target`, `ray_clearance` | yes, 0.5 m | ignore the point's surface |
| `ray_reveal` | yes | rays can reveal a disappearance |
| `ray_hide` | yes | rays can requalify as hidden |
| `ray_fallback` | yes | without a viewpoint, coverage answers |
| `min_cluster`, `cluster_radius` | 30, 0.6 m | small-cluster filter |
| `transient_classes` | vehicle, person | transient groups, used by the annotator |

Source: `trajectory` (path of the CSV), `trajectory_z` (altitude column, if
the file has several).
