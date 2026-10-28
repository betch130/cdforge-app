# Composants tiers / Third-party components

L'application et le module IA incluent les composants ci-dessous, chacun sous sa propre licence. La licence de cdforge (voir [LICENSE](LICENSE)) ne s'applique pas à eux.
*The application and the AI module bundle the components below, each under its own licence. The cdforge licence does not apply to them.*

## Points particuliers / Notes

- **Qt / PySide6** (LGPL-3.0) : liés dynamiquement. Pour les remplacer par une autre version, extraire l'image (`./cdforge-*.AppImage --appimage-extract`), substituer les bibliothèques dans `squashfs-root/usr/lib/python3.13/site-packages/PySide6`, puis lancer `squashfs-root/AppRun`. Sources : <https://download.qt.io/official_releases/QtForPython/>.
  *Qt is dynamically linked; extract the AppImage to replace it with another build.*
- **Bibliothèques CUDA de NVIDIA** (module IA) : composants redistribuables, sous le [contrat de licence CUDA](https://docs.nvidia.com/cuda/eula/), tels que fournis avec PyTorch.
  *NVIDIA CUDA redistributable components, under the CUDA EULA, as shipped with PyTorch.*
- **SNAP** (neu-vi, MIT) : <https://github.com/neu-vi/SNAP>, révision `59fa79b`.
- **Pointcept** (MIT) : <https://github.com/Pointcept/Pointcept>, révision `1342eda`, non modifié.
- **Python 3.13** (PSF) et bibliothèques système de conda-forge (libxcb, xcb-util, OpenSSL, zlib, zstd, libffi, SQLite, Tk…), chacune sous sa licence d'origine.

## Application (AppImage)

| Paquet | Version | Licence |
|---|---|---|
| annotated-doc | 0.0.5 | MIT |
| annotated-types | 0.8.0 | MIT |
| attrs | 26.1.0 | MIT |
| certifi | 2026.10.23 | Mozilla Public License 2.0 (MPL 2.0) |
| charset-normalizer | 3.5.2 | MIT |
| cloth_simulation_filter | 1.1.7 | Apache-2.0 |
| colorama | 0.4.6 | BSD License |
| contourpy | 1.4.0 | BSD-3-Clause |
| cycler | 0.12.1 | BSD License |
| cyclopts | 5.3.2 | Apache-2.0 |
| docstring_parser | 0.18.0 | MIT License |
| fonttools | 4.68.0 | MIT |
| idna | 3.20 | BSD-3-Clause |
| kiwisolver | 1.5.1 | BSD License |
| laspy | 2.7.0 | BSD-2-Clause |
| lazrs | 0.8.2 | MIT |
| markdown-it-py | 4.2.0 | MIT License |
| matplotlib | 3.11.2 | Python Software Foundation License |
| mdurl | 0.1.2 | MIT License |
| numpy | 2.5.4 | BSD-3-Clause AND 0BSD AND MIT AND Zlib AND CC0-1.0 |
| packaging | 26.3 | Apache-2.0 OR BSD-2-Clause |
| pillow | 13.0.0 | MIT-CMU |
| pip | 26.2.2 | MIT |
| platformdirs | 4.13.7 | MIT |
| pooch | 1.9.0 | BSD-3-Clause |
| psutil | 7.2.2 | BSD-3-Clause |
| pydantic | 2.14.1 | MIT |
| pydantic_core | 2.50.1 | MIT |
| Pygments | 2.21.0 | BSD-2-Clause |
| pyparsing | 3.3.3 | MIT |
| pyproj | 3.8.0 | MIT |
| pyqtgraph | 0.14.0 | MIT |
| PySide6_Essentials | 6.12.0 | LGPL-3.0-only OR GPL-2.0-only OR GPL-3.0-only |
| python-dateutil | 3.0.0 | BSD License, Apache Software License |
| pyvista | 0.49.1 | MIT |
| pyvista-validation | 0.2.2 | MIT |
| pyvistaqt | 0.13.1 | MIT License |
| PyYAML | 6.0.3 | MIT License |
| QtPy | 2.4.3 | MIT License |
| requests | 2.34.2 | Apache Software License |
| rich | 15.0.0 | MIT License |
| rich-rst | 2.2.0 | MIT |
| scipy | 1.18.1 | BSD License |
| scooby | 0.12.1 | MIT |
| shellingham | 1.5.4 | ISC License (ISCL) |
| shiboken6 | 6.12.0 | LGPL-3.0-only OR GPL-2.0-only OR GPL-3.0-only |
| six | 1.17.0 | MIT License |
| threadpoolctl | 3.7.0 | BSD-3-Clause |
| typer | 0.27.3 | MIT |
| typing-inspection | 0.4.4 | MIT |
| typing_extensions | 4.16.0 | PSF-2.0 |
| tzdata | 2026.6 | Apache-2.0 |
| urllib3 | 2.8.0 | MIT |
| vtk | 9.7.1 | BSD License |

## Module IA / AI module

| Paquet | Version | Licence |
|---|---|---|
| accelerate | 1.15.0 | Apache Software License |
| addict | 2.4.0 | MIT License |
| aiohappyeyeballs | 2.7.1 | Python Software Foundation License |
| aiohttp | 3.14.5 | Apache-2.0 AND MIT |
| aiosignal | 1.4.0 | Apache Software License |
| alembic | 1.20.1 | MIT |
| annotated-doc | 0.0.5 | MIT |
| annotated-types | 0.8.0 | MIT |
| anyio | 4.15.1 | MIT |
| attrs | 26.1.0 | MIT |
| ccimport | 0.4.4 | MIT License |
| certifi | 2026.10.23 | Mozilla Public License 2.0 (MPL 2.0) |
| charset-normalizer | 3.5.2 | MIT |
| click | 8.5.0 | BSD-3-Clause |
| clip (OpenAI) | 1.0 | MIT |
| colorlog | 6.12.0 | MIT License |
| conda-pack | 0.9.2 | BSD-3-Clause |
| contourpy | 1.4.0 | BSD-3-Clause |
| cumm-cu126 | 0.7.11 | MIT License |
| cwevals | 0.0.1b1 | Apache Software License |
| cycler | 0.12.1 | BSD License |
| distro | 1.9.0 | Apache Software License |
| einops | 0.8.2 | MIT License |
| filelock | 3.32.3 | MIT |
| fire | 0.7.1 | Apache-2.0 |
| flash_attn | 2.8.3 | BSD License |
| fonttools | 4.68.0 | MIT |
| frozenlist | 1.8.0 | Apache-2.0 |
| fsspec | 2026.7.0 | BSD-3-Clause |
| ftfy | 6.3.1 | Apache-2.0 |
| gmpy2 | 2.3.2 | LGPL-3.0-or-later |
| googleapis-common-protos | 1.76.0 | Apache-2.0 |
| h11 | 0.16.0 | MIT License |
| hf-xet | 1.7.0 | Apache-2.0 |
| httpcore | 1.0.9 | BSD-3-Clause |
| httpcore2 | 2.16.0 | BSD-3-Clause |
| httpx | 0.28.1 | BSD License |
| httpx2 | 2.16.0 | BSD-3-Clause |
| huggingface_hub | 2.3.0 | Apache Software License |
| idna | 3.20 | BSD-3-Clause |
| Jinja2 | 3.1.6 | BSD License |
| kiwisolver | 1.5.1 | BSD License |
| lark | 1.3.1 | MIT License |
| llvmlite | 0.50.0 | BSD-2-Clause AND Apache-2.0 WITH LLVM-exception |
| Mako | 1.4.3 | MIT |
| markdown-it-py | 4.2.0 | MIT License |
| MarkupSafe | 3.0.3 | BSD-3-Clause |
| matplotlib | 3.11.2 | Python Software Foundation License |
| mdurl | 0.1.2 | MIT License |
| mpmath | 1.3.0 | BSD License |
| multidict | 7.2.1 | Apache-2.0 |
| networkx | 3.6.1 | BSD-3-Clause |
| ninja | 1.13.2 | Apache-2.0 |
| numba | 0.68.1 | BSD License |
| numpy | 2.5.2 | BSD-3-Clause AND 0BSD AND MIT AND Zlib AND CC0-1.0 |
| nvidia-cublas-cu12 | 12.8.3.14 | Other/Proprietary License |
| nvidia-cuda-cupti-cu12 | 12.8.57 | Other/Proprietary License |
| nvidia-cuda-nvrtc-cu12 | 12.8.61 | Other/Proprietary License |
| nvidia-cuda-runtime-cu12 | 12.8.57 | Other/Proprietary License |
| nvidia-cudnn-cu12 | 9.7.1.26 | Other/Proprietary License |
| nvidia-cufft-cu12 | 11.3.3.41 | Other/Proprietary License |
| nvidia-cufile-cu12 | 1.13.0.11 | Other/Proprietary License |
| nvidia-curand-cu12 | 10.3.9.55 | Other/Proprietary License |
| nvidia-cusolver-cu12 | 11.7.2.55 | Other/Proprietary License |
| nvidia-cusparse-cu12 | 12.5.7.53 | Other/Proprietary License |
| nvidia-cusparselt-cu12 | 0.6.3 | NVIDIA Proprietary Software |
| nvidia-nccl-cu12 | 2.26.2 | Other/Proprietary License |
| nvidia-nvjitlink-cu12 | 12.8.61 | Other/Proprietary License |
| nvidia-nvtx-cu12 | 12.8.55 | Other/Proprietary License |
| ocnn | 2.3.3 | MIT |
| opentelemetry-api | 1.45.1 | Apache-2.0 |
| opentelemetry-exporter-http-transport | 0.66b1 | Apache-2.0 |
| opentelemetry-exporter-otlp-common | 0.66b1 | Apache-2.0 |
| opentelemetry-exporter-otlp-proto-common | 1.45.1 | Apache-2.0 |
| opentelemetry-exporter-otlp-proto-http | 1.45.1 | Apache-2.0 |
| opentelemetry-proto | 1.45.1 | Apache-2.0 |
| opentelemetry-sdk | 1.45.1 | Apache-2.0 |
| opentelemetry-semantic-conventions | 0.66b1 | Apache-2.0 |
| optuna | 5.0.0 | MIT License |
| packaging | 26.3 | Apache-2.0 OR BSD-2-Clause |
| pandas | 3.1.0 | BSD-3-Clause |
| pccm | 0.4.16 | MIT License |
| peft | 0.21.2 | Apache Software License |
| pillow | 12.3.0 | MIT-CMU |
| pip | 26.2.2 | MIT |
| platformdirs | 4.13.6 | MIT |
| pointgroup_ops (Pointcept) | 0.0.0 | MIT |
| pointops (Pointcept) | 1.0 | MIT |
| portalocker | 4.4.1 | BSD-3-Clause |
| propcache | 0.5.4 | Apache-2.0 |
| protobuf | 7.36.2 | 3-Clause BSD License |
| psutil | 7.2.2 | BSD-3-Clause |
| pybind11 | 3.1.0 | BSD-3-Clause |
| pydantic | 2.14.1 | MIT |
| pydantic_core | 2.50.1 | MIT |
| Pygments | 2.21.0 | BSD-2-Clause |
| pyparsing | 3.3.3 | MIT |
| python-dateutil | 3.0.0 | BSD License, Apache Software License |
| PyYAML | 6.0.3 | MIT License |
| regex | 2026.9.29 | Apache-2.0 AND CNRI-Python |
| requests | 2.34.2 | Apache Software License |
| rich | 15.0.0 | MIT License |
| safetensors | 0.8.0 | Apache Software License |
| scipy | 1.18.1 | BSD License |
| setuptools | 78.1.0 | MIT License |
| shellingham | 1.5.4 | ISC License (ISCL) |
| six | 1.17.0 | MIT License |
| sniffio | 1.3.1 | MIT License, Apache Software License |
| spconv-cu126 | 2.3.8 | Apache Software License |
| SQLAlchemy | 2.1.6 | MIT |
| sympy | 1.14.0 | BSD License |
| termcolor | 3.3.0 | MIT |
| timm | 1.0.30 | Apache Software License |
| tokenizers | 0.23.3 | Apache Software License |
| torch | 2.7.1+cu128 | BSD License |
| torch-geometric | 2.8.1 | MIT |
| torch_cluster | 1.6.3+pt27cu128 | MIT License |
| torch_scatter | 2.1.2+pt27cu128 | MIT License |
| torchaudio | 2.7.1+cu128 | BSD License |
| torchvision | 0.22.1+cu128 | BSD |
| tqdm | 4.70.1 | MPL-2.0 AND MIT |
| transformers | 5.21.0 | Apache 2.0 License |
| triton | 3.3.1 | MIT License |
| truststore | 0.10.5 | MIT |
| typer | 0.27.3 | MIT |
| typing-inspection | 0.4.4 | MIT |
| typing_extensions | 4.16.0 | PSF-2.0 |
| tzdata | 2026.5 | Apache-2.0 |
| urllib3 | 2.8.0 | MIT |
| wandb | 0.31.0 | MIT License |
| wcwidth | 0.9.4 | MIT License |
| xxhash | 4.0.2 | BSD-2-Clause |
| yapf | 0.43.0 | Apache Software License |
| yarl | 1.25.1 | Apache-2.0 |
