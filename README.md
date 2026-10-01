# RotationDetection (AlphaRotate fork)

> A preserved fork of **AlphaRotate**, a TensorFlow 1.x benchmark for rotated (arbitrary-oriented) object detection,
> reorganized and documented so it can be understood without reading every file.

| | |
|---|---|
| **Project origin** | Other: fork of a third-party open-source research codebase (see [Project Context](#project-context)) |
| **Upstream** | [`yangxue0827/RotationDetection`](https://github.com/yangxue0827/RotationDetection), AlphaRotate v1.0.1 |
| **Original authors** | Xue Yang, Yue Zhou, Junchi Yan and contributors (Shanghai Jiao Tong University) |
| **Language / framework** | Python 3, TensorFlow 1.13 (`tf.contrib.slim`), Cython, CUDA C++ |
| **License** | [Apache License 2.0](./LICENSE) (unchanged) |
| **Last upstream commit in this fork** | `6adfd2d`, 2022-08-03 ("add bcd") |

---

## Project Overview

Most object detectors predict **horizontal** boxes `(x1, y1, x2, y2)`. Many real scenes (aerial and satellite imagery,
scene text, ships in SAR images, faces, retail shelves) contain objects at **arbitrary orientations**. For those objects
a rotated box `(x, y, w, h, θ)` or a quadrilateral fits much better.

This repository contains a full benchmark for that task. It includes:

- **24 detector implementations** (for example RetinaNet-H/R, R²CNN, RRPN, SCRDet, R³Det, CSL, DCL, GWD, KLD, KFIoU,
  BCD, RSDet, RIDet, FCOS, ATSS, and hybrids such as R³Det-KLD).
- **Interchangeable backbones**: ResNet (TF-slim, GluonCV-converted, PyTorch-converted), MobileNetV2, EfficientNet and DarkNet53.
- **Necks**: FPN and BiFPN.
- **207 experiment configuration files** across 11 datasets (DOTA 1.0/1.5/2.0, DIOR-R, HRSC2016, ICDAR2015, MLT,
  MSRA-TD500, UCAS-AOD, FDDB, Total-Text).
- **Dataset preparation, training, evaluation and visualization scripts**, plus custom CUDA/Cython kernels for rotated
  IoU and rotated NMS.

![AlphaRotate module overview](./images/projects.png)

*Module overview from the upstream project: data → backbone → detector → neck → anchor head → RoI extractor → bbox head → loss.*

## Project Context

**Confirmed (from the repository itself):**

- All 51 commits in this repository were written by the upstream authors (`yangxue`: 48 commits, `erenzhou`: 3 commits),
  between 2021-09-15 and 2022-08-03. The fork owner has no source commits.
- Source file headers, `setup.py`, the Sphinx docs (`docs/conf.py`) and the original README name
  **Xue Yang / Shanghai Jiao Tong University** as author and maintainer.
- The project is the code release that accompanies *"AlphaRotate: A Rotation Detection Benchmark using TensorFlow"*
  (arXiv:2111.06677) and several of its authors' papers (CSL/ECCV'20, DCL/CVPR'21, GWD/ICML'21, KLD/NeurIPS'21, KFIoU, BCD/TPAMI'22, R³Det/AAAI'21).

**Inferred:**

- This repository is a GitHub **fork** kept by the owner as a reference copy of the benchmark, likely for study or
  experimentation with rotated object detection.

**Unknown:**

- The original repository does not provide enough information to determine why the fork was made, or whether it was
  used for a specific course, thesis or personal experiment. No fork-specific documents, results or code changes exist.

See [`docs/project-context.md`](./docs/project-context.md) for the full evidence.

## Problem Statement

Rotated object detection has specific problems that horizontal detection does not have. The upstream code explicitly
targets these:

- **Angle boundary discontinuity.** Regressing θ directly produces large losses at the edges of the angle range
  (for example −90° vs. 89°). CSL and DCL address this by treating the angle as a classification problem.
- **Mismatch between loss and metric.** Smooth-L1 on `(x, y, w, h, θ)` does not track rotated IoU. GWD, KLD, KFIoU and
  BCD address this by modelling boxes as 2-D Gaussian distributions and computing distances between them.
- **Square-like objects and ordering ambiguity** for quadrilateral representations (RSDet with modulated loss, RIDet).
- **The absence of a unified, fair benchmark** for comparing these methods under identical settings.

## Objective

According to the upstream documentation (`docs/index.rst`), the project was developed to:

1. provide **modules** for building rotation-detection algorithms;
2. provide **implementations** of state-of-the-art rotation-detection methods;
3. **benchmark** existing algorithms under identical datasets and experimental settings, so they can be compared fairly.

## Repository Structure

```
RotationDetection/
├── README.md                 ← this file (new)
├── AGENTS.md                 ← guardrails for AI-assisted work on this repo (new)
├── LICENSE                   ← Apache-2.0 (original)
├── setup.py, MANIFEST.in     ← package build incl. CUDA/Cython extensions (original)
├── requirements.txt          ← (original)
│
├── alpharotate/              ← core library (original)
│   ├── libs/models/          ← backbones, necks, anchor heads, samplers, detectors, losses
│   ├── libs/utils/           ← box transforms, rotated IoU/NMS (Python, Cython, CUDA), drawing
│   ├── libs/val_libs/        ← VOC-style evaluation (horizontal and rotated)
│   ├── libs/label_name_dict/ ← class-name ↔ id maps per dataset
│   └── utils/                ← standalone implementations: CSL, DCL, GWD, KLD, KF, BCD, Gaussian metrics
│
├── configs/                  ← experiment configs: configs/<DATASET>/<detector>/cfgs_*.py + _base_/
├── dataloader/               ← dataset conversion (→ XML → TFRecord), TFRecord reader, augmentation
├── tools/                    ← per-detector train / test / export scripts + shared *_base.py runners
├── demo/                     ← video ↔ frame utilities for making demo videos
├── thirdparty/               ← DOTA devkit, ICDAR/MSRA-TD500 devkit, gluon2TF weight converter
├── output/trained_weights/   ← placeholder where trained checkpoints are expected
├── images/                   ← figures used by the original README, Sphinx docs and code docstrings
│
└── docs/
    ├── README.md             ← documentation index (new)
    ├── project-context.md    ← origin, evidence, timeline (new)
    ├── architecture.md       ← how the components fit together (new)
    ├── code-overview.md      ← what each package / script does (new)
    ├── usage.md              ← workflow as documented upstream (new)
    ├── possible-improvements.md ← observations, NOT applied (new)
    ├── sdlc/                 ← intent.md, spec.md, plan.md (new, reverse-engineered)
    ├── examples/example.xml  ← annotation-format sample (moved from repository root)
    ├── original/             ← original upstream README and OTHERS.md (moved, content unchanged)
    └── *.rst, conf.py, api/, guide/, images/ ← original Sphinx documentation (unchanged)
```

Code directories were intentionally **not** moved into a `src/` folder. The scripts use relative imports and paths
(`sys.path.append("../../")`, `ROOT_PATH = os.path.abspath('../../')`, `'../../dataloader/tfrecord'`). Moving them
would break the original implementation. See [`docs/sdlc/plan.md`](./docs/sdlc/plan.md) for every decision made.

## Original Implementation

This repository preserves the original implementation of the project. The source code has intentionally not been
refactored or modernized in order to retain the historical context and original development approach.

The source code was written by the upstream AlphaRotate authors and is kept byte-for-byte as it was at upstream commit
`6adfd2d`. The reorganization only added Markdown documentation, moved three non-code documents, and removed
macOS `.DS_Store` metadata files.

## Technologies

Identified directly from `setup.py`, `requirements.txt`, imports and the original README:

| Area | Technology |
|---|---|
| Language | Python 3 (upstream environment: Python 3.5), C++, CUDA, Cython |
| Deep learning | TensorFlow-GPU 1.13.1 (`tf.contrib.slim`, `tf.py_func`, TFRecord queues) |
| Numerics / geometry | NumPy, OpenCV 4.1.1.26, Shapely 1.7.1 |
| Build | setuptools + Cython `build_ext` with a custom `nvcc` compiler hook |
| Tooling | tqdm, TensorBoard, optional `tfplot`; SWIG for the DOTA devkit |
| Pre-trained weights | TF-slim checkpoints, GluonCV (MXNet) weights converted with `thirdparty/gluon2TF`, PyTorch weights converted with MMdnn |
| Docs | Sphinx + `sphinx_rtd_theme` + `m2r2` (Read the Docs) |
| Upstream runtime | CUDA 10.0; a Docker image `yangxue2docker/yx-tf-det:tensorflow1.13.1-cuda10-gpu-py3` is referenced |

## How It Works

1. **Prepare data.** Dataset-specific scripts in `dataloader/dataset/<DATASET>/` crop large images (DOTA) and convert
   annotations to a VOC-like XML format with 4-point boxes (see [`docs/examples/example.xml`](./docs/examples/example.xml)).
   `convert_data_to_tfrecord.py` then packs images and boxes into TFRecords.
2. **Select an experiment.** Copy a config from `configs/<DATASET>/<detector>/cfgs_*.py` over `configs/cfgs.py`.
   Every config builds on `configs/_base_/` (model, dataset, schedule) and usually ends with a docstring recording the
   upstream authors' DOTA evaluation result.
3. **Train.** `tools/<detector>/train.py` subclasses `tools/train_base.Train`, builds the detector graph
   (`alpharotate/libs/models/detectors/<detector>/build_whole_network.py`), splits batches across GPUs, averages gradients and
   writes checkpoints and TensorBoard summaries to `output/`.
4. **Test and evaluate.** `tools/<detector>/test_<dataset>.py` runs multi-GPU inference, optionally with multi-scale or
   flip test-time augmentation. Results are merged with rotated NMS, then evaluated with the VOC-style rotated metric in
   `alpharotate/libs/val_libs/` or exported in the DOTA / ICDAR submission format.

Details: [`docs/architecture.md`](./docs/architecture.md).

## Inputs and Outputs

| Kind | Location / format |
|---|---|
| Raw datasets (not included) | External paths, for example `/data/dataset/DOTA/...` hard-coded as script defaults |
| Intermediate annotations | VOC-style XML with `x0..y3` polygon vertices |
| Training input | `dataloader/tfrecord/<DATASET>_train*.tfrecord` (git-ignored) |
| Pre-trained backbones (not included) | `dataloader/pretrained_weights/` |
| Checkpoints (not included) | `output/trained_weights/<VERSION>/` |
| TensorBoard logs | `output/summary/<VERSION>/` |
| Test outputs | `tools/<detector>/test_<dataset>/<VERSION>/` (per-class DOTA txt files, visualizations) |
| Included ground-truth archives | `dataloader/dataset/{ICDAR2015,MSRA-TD500}/gt.zip`, `dataloader/dataset/Total_Text/labels.zip` |

## Running the Project

No execution was attempted during the reorganization. The dependencies (TensorFlow 1.13, CUDA 10, Python 3.5) are
end-of-life and need an NVIDIA GPU. The steps below are **the upstream authors' documented procedure**, and have not
been verified for this fork:

```shell
pip install -r requirements.txt
pip install -v -e .          # compiles the Cython/CUDA extensions; requires nvcc on PATH or $CUDAHOME

cd dataloader/dataset/DOTA && python data_crop.py
cd dataloader/dataset && python convert_data_to_tfrecord.py --root_dir=/PATH/TO/DOTA/ --dataset=DOTA ...

cd tools/retinanet && python train.py
cd tools/retinanet && python test_dota.py --test_dir=/PATH/TO/IMAGES/ --gpus=0
```

Full walkthrough with caveats: [`docs/usage.md`](./docs/usage.md).

## Documentation

| Document | Purpose |
|---|---|
| [docs/README.md](./docs/README.md) | Documentation index |
| [docs/project-context.md](./docs/project-context.md) | Origin, evidence, timeline, classification |
| [docs/architecture.md](./docs/architecture.md) | Component relationships and the training / inference data flow |
| [docs/code-overview.md](./docs/code-overview.md) | Package-by-package and script-by-script description |
| [docs/usage.md](./docs/usage.md) | Install, data preparation, training, testing, as documented upstream |
| [docs/possible-improvements.md](./docs/possible-improvements.md) | Observed issues and modernization ideas (not applied) |
| [docs/sdlc/intent.md](./docs/sdlc/intent.md) · [spec.md](./docs/sdlc/spec.md) · [plan.md](./docs/sdlc/plan.md) | Reverse-engineered intent, specification and the reorganization plan |
| [docs/original/](./docs/original/) | Original upstream README (with benchmark tables and model links) and OTHERS.md |
| `docs/index.rst` + `docs/guide/` | Original Sphinx / Read the Docs sources |

## Citation and Attribution

All credit for the implementation belongs to the upstream authors. If you use this code, cite the original work:

```bibtex
@article{yang2021alpharotate,
    author  = {Yang, Xue and Zhou, Yue and Yan, Junchi},
    title   = {AlphaRotate: A Rotation Detection Benchmark using TensorFlow},
    journal = {arXiv preprint arXiv:2111.06677},
    year    = {2021},
}
```

The upstream project has since been superseded by the PyTorch-based [MMRotate](https://github.com/open-mmlab/mmrotate),
as its own README announces.

## Historical Note

This repository was later reorganized and documented to improve readability and preserve the historical context of
the original project. The original source code remains unchanged.
