# Architecture

← Back to [README](../README.md) · [Documentation index](./README.md)

This document describes the architecture **as it exists in the code**. It adds no layers or services that the code
does not have. The system is a set of TensorFlow 1.x graph-building libraries plus command-line scripts. It is not a
service or an application.

## 1. Component Map

```
                    ┌──────────────────────────────────────────────┐
                    │ configs/cfgs.py  (active experiment)         │
                    │   ← copied from configs/<DATASET>/<det>/…     │
                    │   ← star-imports configs/_base_/{models,     │
                    │      datasets,schedules}                     │
                    └───────────────┬──────────────────────────────┘
                                    │ module-level constants (NET_NAME, CLASS_NUM, LR, VERSION …)
        ┌───────────────────────────┼─────────────────────────────────────────┐
        ▼                           ▼                                         ▼
┌───────────────────┐   ┌───────────────────────────────┐        ┌──────────────────────────┐
│ dataloader/       │   │ tools/<det>/train.py          │        │ tools/<det>/test_*.py     │
│  dataset/*/       │   │   class X(Train)              │        │   class X(TestDOTA / …)   │
│  → XML → TFRecord │──▶│ tools/train_base.py           │        │ tools/test_*_base.py      │
│  read_tfrecord.py │   │   multi-GPU towers, grad avg, │        │   multi-GPU workers,      │
│  image_augment.py │   │   warm-up LR, saver, summaries│        │   crop/merge, NMS, eval   │
└───────────────────┘   └───────────────┬───────────────┘        └────────────┬─────────────┘
                                        │ build_whole_detection_network()     │
                                        ▼                                     ▼
                  ┌──────────────────────────────────────────────────────────────────────┐
                  │ alpharotate/libs/models/detectors/<det>/build_whole_network.py        │
                  │   extends single_stage_base_network.DetectionNetworkBase  (RetinaNet- │
                  │   family, R³Det-family, CSL, DCL, GWD, KLD, KFIoU, BCD, ATSS …)        │
                  │   or two_stage_base_network.DetectionNetworkBase (R²CNN, RRPN, SCRDet)│
                  └───┬──────────────┬──────────────┬──────────────┬─────────────┬───────┘
                      ▼              ▼              ▼              ▼             ▼
               backbones/+necks/  anchor_heads/   samplers/      losses/      roi_extractors/
               (BuildBackbone)   (GenerateAnchors) (anchor/proposal (focal, smooth-L1,  box_heads/
                                                   target layers)  IoU-smooth-L1, CSL,  (two-stage)
                                                                   DCL, GWD, KLD, KF, BCD …)
                      │                                                  │
                      └──────────────┬───────────────────────────────────┘
                                     ▼
                     alpharotate/libs/utils/  (bbox_transform, coordinate_convert,
                     rotated IoU / NMS: Python + Cython + CUDA, drawing)
                     alpharotate/utils/        (CSL/DCL label coding, Gaussian metrics)
                     alpharotate/libs/val_libs (VOC-style rotated evaluation)
```

## 2. Configuration Model

- The **only** configuration mechanism is Python modules. Every script does `from configs import cfgs`.
- `configs/cfgs.py` is the "active" experiment. The upstream workflow is to **copy** a file from
  `configs/<DATASET>/<detector>/cfgs_res50_*.py` over it.
- Each experiment file star-imports three base layers:
  - `configs/_base_/models/retinanet_r50_fpn.py` or `faster_rcnn_r50_fpn.py`: backbone, FPN, anchors, method (`'H'` horizontal or `'R'` rotated anchors), NMS, and **`ROOT_PATH = os.path.abspath('../../')`**
  - `configs/_base_/datasets/dota_detection.py`: dataset name, class count, pixel mean/std, image size, augmentation flags
  - `configs/_base_/schedules/schedule_1x.py`: GPUs, LR, momentum, decay epochs, gradient clipping
- It then overrides values and sets a unique `VERSION` string. That string names the checkpoint, summary and
  test-output folders.

**Implication:** `ROOT_PATH` is derived from the *current working directory*, so every script must be run from two
levels below the repository root (for example `tools/retinanet/`). This is the main reason the directory layout was
preserved during the reorganization.

## 3. Detector Families

| Base class | Detectors | Pipeline |
|---|---|---|
| `single_stage_base_network.DetectionNetworkBase` | `retinanet`, `refine_retinanet`, `r3det`, `r3det_{bcd,dcl,gwd,kfiou,kl}`, `csl`, `dcl`, `gwd`, `kl`, `kfiou`, `bcd`, `atss` | backbone+FPN (P3–P7) → shared cls/reg subnets per level → anchors → anchor target layer (`tf.py_func`) → loss → per-class decode + rotated NMS |
| (same base, variants) | `rsdet` (5-param / 8-point), `ridet` (8-point), `fcos` (anchor-free: rbox / quad) | as above, with quadrilateral or anchor-free heads |
| `two_stage_base_network.DetectionNetworkBase` | `r2cnn`, `r2cnn_{bcd,kfiou,kl}`, `rrpn`, `scrdet` | backbone+FPN (P2–P6) → RPN → proposal post-processing → level assignment → RoI extraction → box head → rotated regression |

**Refinement detectors** (R³Det and its variants) add feature-refinement stages on top of the RetinaNet pipeline:
horizontal anchors in stage 1, refined rotated boxes in later stages (the "H→R" column in the benchmark table).

Several detectors have alternative graph builders next to `build_whole_network.py`:

- `*_batch.py`: batched training
- `*_pb.py`: graph for `exportPb.py` (frozen-graph export)
- `*_atan.py`: angle encoded as (sin θ, cos θ)

## 4. Training Data Flow

1. `ReadTFRecord.next_batch()` reads `dataloader/tfrecord/<DATASET>_train*` through TF queue runners. It decodes the
   image and 8-point boxes, applies augmentation, and resizes to `IMG_SHORT_SIDE_LEN`.
2. For each GPU tower, the training script converts 8-point ground truth into
   - a rotated box `(x, y, w, h, θ, label)` via `backward_convert` (OpenCV `minAreaRect`, in `tf.py_func`), and
   - a horizontal box `(x1, y1, x2, y2, label)` via `get_horizen_minAreaRectangle`.
3. `build_whole_detection_network()` returns the loss dict. `Train.average_gradients()` averages tower gradients.
   Optional bias-gradient multiplier and gradient clipping are applied, then a Momentum optimizer with warm-up and
   piecewise LR decay runs.
4. Every `SAVE_WEIGHTS_INTE` iterations (one "epoch" of iterations) a checkpoint is written to
   `output/trained_weights/<VERSION>/`, and summaries (including images with drawn boxes) go to `output/summary/<VERSION>/`.

## 5. Inference / Evaluation Data Flow (DOTA example)

1. `tools/test_dota_base.py` lists test images and splits them across GPU worker processes (`multiprocessing`).
2. Large images are tiled into overlapping crops (`--h_len`, `--w_len`, `--h_overlap`, `--w_overlap`). Optional
   multi-scale (`-ms`) and flip (`-f`) test-time augmentation can be enabled.
3. Detections from all crops are shifted back to image coordinates and merged with rotated NMS (GPU kernel by default;
   CPU with `-cn`).
4. Results are appended (`a+` mode) to per-class DOTA Task-1 text files for the official evaluation server. Smaller
   datasets (HRSC2016, UCAS-AOD, ICDAR, …) are evaluated locally with `alpharotate/libs/val_libs/voc_eval_r.py`
   or the devkits in `thirdparty/`.

## 6. Native Extensions

Built by `setup.py` (and by `alpharotate/libs/utils/setup.py` / `cython_utils/Makefile` in the manual path):

| Extension | Sources | Purpose |
|---|---|---|
| `rbbox_overlaps` | `rbbox_overlaps_kernel.cu` + `.pyx` | GPU rotated-box IoU matrix (used for anchor matching) |
| `rotate_polygon_nms` | `rotate_polygon_nms_kernel.cu` + `.pyx` | GPU rotated NMS |
| `iou_cpu` | `iou_cpu.pyx` | CPU rotated IoU |
| `cython_utils/cython_bbox`, `cython_nms` | `bbox.pyx`, `nms.pyx` | Horizontal box overlaps and NMS |
| `thirdparty/DOTA_devkit/polyiou` | SWIG `polyiou.i/.cpp` | Polygon IoU for the DOTA evaluation |

`setup.py` calls `locate_cuda()` at import time, so the package cannot be built without an `nvcc` installation.
The kernels are compiled with `-arch=sm_35`.

## 7. What Is *Not* Here

- No service, API, database, CLI entry point, test suite, or CI. The `*_test.py` files under `backbones/slim_nets/` and
  `mobilenet/` are vendored TensorFlow unit tests and are not wired into any runner.
- No datasets, pre-trained weights or trained checkpoints. These were distributed separately (Baidu/Google Drive links in
  the [original README](./original/README.upstream.md)).
