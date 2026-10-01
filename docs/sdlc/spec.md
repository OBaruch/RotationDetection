# Specification

← [intent.md](./intent.md) · Next: [plan.md](./plan.md) · [Documentation index](../README.md)

> **Artifact type:** Specification (the *what*). It is reverse-engineered from the existing code.
> It describes the behavior of the **existing** system, as observed in the code at commit `6adfd2d`. It is not a
> design for new features. Requirement IDs exist so that the docs and future work can refer to them.
> Status labels: **Confirmed** = visible in code/docs, **Inferred** = reasonable reading, **Unverified** = documented upstream but not executed here.

---

## 1. System Summary

A TensorFlow 1.x library (`alpharotate`) plus script collection (`tools/`, `dataloader/`, `configs/`). It trains and
evaluates rotated object detectors on public benchmarks. It runs from the command line on one or more NVIDIA GPUs.

## 2. Functional Requirements (as implemented)

### FR-1 Data preparation

| ID | Requirement | Status | Where |
|---|---|---|---|
| FR-1.1 | Convert dataset-native annotations (txt/json) to VOC-style XML with 4-vertex polygons `x0..y3` | Confirmed | `dataloader/dataset/<DATASET>/txt2xml.py`, `json2xml.py` |
| FR-1.2 | Tile very large images (DOTA) into overlapping crops with matching annotations | Confirmed | `dataloader/dataset/DOTA/data_crop.py`, `data_slicer.py` |
| FR-1.3 | Serialize images and polygons to TFRecord, named `<DATASET>_<save_name>` | Confirmed | `dataloader/dataset/convert_data_to_tfrecord.py` |
| FR-1.4 | Read TFRecords with augmentation: short-side resize, max length, horizontal/vertical flip, rotation, RGB→gray, aspect jitter, image pyramid | Confirmed | `read_tfrecord.py`, `image_augmentation.py`, `_base_/datasets` flags |

### FR-2 Model construction

| ID | Requirement | Status | Where |
|---|---|---|---|
| FR-2.1 | Select a backbone by name: ResNet (slim / GluonCV v1b/v1d / PyTorch-converted), MobileNetV2, EfficientNet, DarkNet53 | Confirmed | `backbones/build_backbone_*.py`, `utils/pretrain_zoo.py` |
| FR-2.2 | Build FPN (P3–P7 or P2–P6) or BiFPN necks | Confirmed | `necks/` |
| FR-2.3 | Generate horizontal (`METHOD='H'`) or rotated (`'R'`) anchors | Confirmed | `anchor_heads/` |
| FR-2.4 | Provide 24 detector graphs, single-stage, refined single-stage, two-stage and anchor-free | Confirmed | `detectors/` |
| FR-2.5 | Represent boxes as 5-parameter `(x, y, w, h, θ)` with angle range 90° or 180°, or as 8-point quadrilaterals | Confirmed | `coordinate_convert.py`, `ANGLE_RANGE` |
| FR-2.6 | Encode angles as regression, sin/cos (`atan`), CSL (smooth label) or DCL (binary/Gray code) | Confirmed | `utils/smooth_label.py`, `utils/densely_coded_label.py` |

### FR-3 Losses

| ID | Requirement | Status |
|---|---|---|
| FR-3.1 | Classification: focal loss, cross-entropy | Confirmed |
| FR-3.2 | Regression: smooth-L1, IoU-smooth-L1 (log/exp), modulated loss, Hungarian loss | Confirmed |
| FR-3.3 | Gaussian-based regression: GWD, ProbIoU, KLD (+ Jeffreys / JS), KFIoU, BCD | Confirmed |

### FR-4 Training

| ID | Requirement | Status | Where |
|---|---|---|---|
| FR-4.1 | Multi-GPU data-parallel training with tower gradient averaging | Confirmed | `tools/train_base.py` |
| FR-4.2 | Momentum SGD, linear warm-up, piecewise-constant decay by epoch, bias-gradient multiplier, gradient-norm clipping | Confirmed | `train_base.py`, `tools/*/train.py` |
| FR-4.3 | Restore backbone from TF / PyTorch-converted / MXNet-converted weights | Confirmed | `get_restorer()` in base networks |
| FR-4.4 | Save checkpoints every `SAVE_WEIGHTS_INTE` iterations (`max_to_keep=2`) to `output/trained_weights/<VERSION>` | Confirmed | `train_base.py` |
| FR-4.5 | Write TensorBoard scalars and images with drawn GT/anchors/detections to `output/summary/<VERSION>` | Confirmed | `train_base.py`, `show_box_in_tensor.py` |
| FR-4.6 | SWA: extra-epoch training + checkpoint averaging | Confirmed | `tools/*/train_swa.py`, `tools/swa.py` |

### FR-5 Inference and evaluation

| ID | Requirement | Status | Where |
|---|---|---|---|
| FR-5.1 | Multi-process, multi-GPU inference over an image folder | Confirmed | `tools/test_*_base.py` |
| FR-5.2 | Sliding-window inference for large images, then merge | Confirmed | `test_dota_base.py` (`--h_len`, `--w_len`, overlaps) |
| FR-5.3 | Optional multi-scale (`-ms`) and flip (`-f`) TTA, visualization (`-s`), CPU NMS (`-cn`) | Confirmed | argparse in `test_dota_base.py` |
| FR-5.4 | Rotated NMS on GPU (CUDA) or CPU | Confirmed | `nms_rotate.py`, `rotate_polygon_nms_kernel.cu` |
| FR-5.5 | Output DOTA Task-1 per-class text files (`a+` append mode) | Confirmed | `test_dota_base.py` |
| FR-5.6 | Local VOC-07/12 rotated mAP evaluation | Confirmed | `val_libs/voc_eval_r.py`, `USE_07_METRIC` |
| FR-5.7 | ICDAR / MSRA-TD500 / Total-Text evaluation via the RRC scripts | Confirmed | `thirdparty/icdar_msra_td500_devkit/` |
| FR-5.8 | Export a frozen `.pb` graph | Confirmed | `tools/*/exportPb.py`, `*_pb.py` graphs |
| FR-5.9 | Measure inference speed | Confirmed | `tools/test_speed_base.py` |

## 3. Interfaces

| Interface | Form |
|---|---|
| Configuration | Python module `configs/cfgs.py` (module-level constants). Selected by copying an experiment file |
| Training CLI | `python train.py` from `tools/<method>/` (no arguments; everything comes from `cfgs`) |
| Test CLI | `python test_<dataset>.py --test_dir … --gpus … [-ms] [-s] [-f] [-cn]` and, for some datasets, `--test_annotation_path`, `--image_ext` |
| Data CLI | `convert_data_to_tfrecord.py --root_dir --xml_dir --image_dir --save_name --save_dir --img_format --dataset` |
| Library API (documented) | `alpharotate.utils.{smooth_label, densely_coded_label, gaussian_metric}` (Sphinx `api/src.rst`) |
| Annotation format | VOC-style XML, `<object><name/><difficult/><bndbox><x0/>…<y3/></bndbox></object>`, see [`../examples/example.xml`](../examples/example.xml) |

## 4. Non-Functional Characteristics

| ID | Characteristic | Status |
|---|---|---|
| NFR-1 | Runtime: Python 3.5, TF-GPU 1.13.1, CUDA 10.0, OpenCV 4.1.1.26, Shapely 1.7.1 | Confirmed (pinned) |
| NFR-2 | Requires an NVIDIA GPU and `nvcc` to build/install | Confirmed (`setup.py`) |
| NFR-3 | Scripts must run with CWD = `tools/<method>/` (relative `ROOT_PATH`, `sys.path`) | Confirmed |
| NFR-4 | Reproducibility is anchored by unique `VERSION` strings and recorded results in config docstrings | Confirmed |
| NFR-5 | License: Apache-2.0 | Confirmed |
| NFR-6 | Executability today, given EOL dependencies | Unverified |

## 5. Data Entities

| Entity | Shape / format |
|---|---|
| Ground truth (8-point) | `[N, 9]`: `x1,y1,…,x4,y4,label` |
| Rotated box | `[N, 5]` `(x, y, w, h, θ)` (+ label → `[N, 6]`) |
| Horizontal box | `[N, 4]` `(x1, y1, x2, y2)` (+ label → `[N, 5]`) |
| Detection | boxes, scores, category per class after NMS |
| Experiment identity | `VERSION`, e.g. `RetinaNet_DOTA_KF_1x_20210902` |

## 6. Specification of the Reorganization (this change)

| ID | Requirement |
|---|---|
| RR-1 | No source file content changes. Verified by SHA-256 over all tracked code files before and after |
| RR-2 | New root `README.md` with overview, context, problem, objective, structure, original-implementation note, technologies, how it works, I/O, running, docs, historical note |
| RR-3 | `docs/` Markdown set: project-context, architecture, code-overview, usage, possible-improvements, index |
| RR-4 | Original README and OTHERS.md preserved verbatim under `docs/original/`, with a link-mapping note |
| RR-5 | Directory layout of code kept, because of NFR-3 |
| RR-6 | Remove only OS metadata (`.DS_Store`), which carries no project information |
| RR-7 | `AGENTS.md` with guardrails for future automated or AI-assisted contributions |
| RR-8 | Commits authored by the repository owner, on a dedicated branch, ready for a PR |
