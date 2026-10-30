# The command line

Everything the studio does goes through the `cdforge` command — the studio itself runs it for each stage. It also works on its own: on a server, in a script, to resume a run.

The project folder (the one holding `project.yaml`) is the last argument, and defaults to `.`. `cdforge <command> --help` gives every option.

## The pipeline

| Command | What it does | Useful options |
|---|---|---|
| `cdforge init FOLDER` | creates a project: commented `project.yaml`, folders, 10 base classes | |
| `cdforge validate` | checks `project.yaml`, the source files and the label schemes | |
| `cdforge ingest` | reads, crops, harmonises labels, writes `work/ingested/*.laz` | `--source`, `--overwrite` |
| `cdforge align` | registers t1 onto t0 | `--pair`, `--overwrite`, `--implementation` |
| `cdforge tile` | cuts each pair into tiles | `--pair`, `--size`, `--overlap`, `--min-points`, `--workers`, `--overwrite` |
| `cdforge prelabel` | change and semantic pre-labels | `--pair`, `--tile`, `--workers`, `--implementation` |
| `cdforge split` | train/val/test split without leakage | |
| `cdforge export` | writes the dataset | `--format laz` or `pointcept` |
| `cdforge report` | writes `reports/report.html` | `--fast` (without reading the tiles again) |
| `cdforge run` | chains everything, from ingestion to the report, skipping what is up to date | `--workers`, `--resume`, `--overwrite` |

`--pair` and `--tile` can be repeated to process several. `--implementation` picks the Rust kernels (`rust`), Python (`python`) or Rust when installed (`auto`, the default); the result does not depend on it.

`cdforge run --resume` resumes an interrupted run (crash, Ctrl+C) without redoing the tiles already finished.

## Semantic models

| Command | What it does |
|---|---|
| `cdforge models` | the catalogue; `--model NAME` for one model in detail |
| `cdforge fetch-weights MODEL` | downloads the weights and checks their fingerprint |
| `cdforge stage-inference MODEL` | prepares the tiles for the model and prints the inference command (`--python`: the interpreter where torch is installed) |
| `cdforge import-predictions FOLDER` | imports `<tile>/<epoch>_semantic.npy` predictions as pre-labels (`--scheme`, `--min-conf`, `--keep-low-conf`, `--reset-auto`) |

## Annotation

| Command | What it does |
|---|---|
| `cdforge studio` | opens cdforge Studio |
| `cdforge gui` | opens the annotator directly (`--pair`, `--tile`, `--no-3d` on a machine without OpenGL) |
| `cdforge transfer-buffer` | gives back to the core of neighbouring tiles what was labelled by hand in the buffers (`--conflicts keep` or `overwrite`) |
| `cdforge locks` | lists tile locks (a tile open in an annotator is locked); `cdforge locks clean` removes those left by a crash |

## Following and checking

| Command | What it does |
|---|---|
| `cdforge status` | tile progress, per pair |
| `cdforge dag` | the chaining of the stages (`--show` for the graph) |
| `cdforge history FILE -p PROJECT` | the processing chain that produced a file |
| `cdforge evaluate` | compares a field to a ground truth (`--gt-field`, `--pred-field`) |
| `cdforge merge` | reassembles the tiles of a pair into complete t0 and t1 clouds (`--no-core-only` to keep the buffers) |
| `cdforge migrate` | brings a project to the current format (`--dry-run`, backup by default) |

## Others

| Command | What it does |
|---|---|
| `cdforge demo [FOLDER]` | creates a synthetic project with known changes and runs the whole pipeline |
| `cdforge install-launcher` | adds cdforge Studio to the applications menu (`--uninstall` to remove it) |

## Environment variables

| Variable | Role |
|---|---|
| `CDFORGE_LANG` | the language (`fr`, `en`), ahead of the studio setting |
| `CDFORGE_TORCH_PYTHON` | the interpreter with torch to use for models, ahead of the AI module |
| `CDFORGE_ALIGN_TMPDIR` | a temporary folder for registration, on a disk with room |
| `CDFORGE_ALIGN_RAM_MB` | the memory kept per registration voxel store (0: everything on disk) |
| `CDFORGE_ALIGN_IMPLEMENTATION`, `CDFORGE_TILING_IMPLEMENTATION`, `CDFORGE_PRELABEL_IMPLEMENTATION` | each stage's default engine |
| `CDFORGE_PRELABEL_WORKERS` | the tiles pre-labelled in parallel |

To finish: [Frequently asked questions](13_faq.md).
