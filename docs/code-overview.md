# Code Overview

← Back to [README](../README.md) · [Documentation index](./README.md)

This is a map of the original source code, organized by directory. Nothing described here was modified. For how the
pieces interact at runtime, see [architecture.md](./architecture.md).

---

## `alpharotate/`: core library (importable package)

### `alpharotate/libs/models/`

| Sub-package | Responsibility | Key files |
|---|---|---|
| `backbones/` | Feature extractors and the dispatcher that picks one from `cfgs.NET_NAME` | `build_backbone_p3top7.py` (single-stage, P3–P7), `build_backbone_p2top6.py` (two-stage, P2–P6), `resnet.py` (TF-slim), `resnet_gluoncv.py` (ResNet-v1b/v1d from GluonCV weights), `resnet_pytorch.py` (MMdnn-converted), `mobilenet_v2.py`, `darknet.py`, `efficientnet/` (vendored TF EfficientNet + CondConv + lite), `mobilenet/`, `slim_nets/` (vendored TF-slim model zoo incl. its unit tests) |
| `necks/` | Multi-scale feature fusion | `fpn_p3top7.py`, `fpn_p2top6.py`, `bifpn_p3top7.py`, `scrdet_neck.py` (SF-Net / MDA-Net attention for SCRDet) |
| `anchor_heads/` | Anchor generation | `generate_anchors.py` (dispatcher), `generate_h_anchors.py` / `_tf.py` (horizontal), `generate_r_anchors.py` (rotated), `anchor_utils.py` |
| `samplers/` | Target assignment: match anchors/proposals to ground truth, produce labels and regression targets. One folder per method | `samper.py` (base), `retinanet/`, `r3det/refine_anchor_sampler_r3det.py`, `r3det_dcl/`, `r3det_kf/`, `csl/`, `dcl/`, `gwd/`, `atss/`, `fcos/`, `rsdet/`, `r2cnn/`, `rrpn/` |
| `detectors/` | Complete network graphs, one package per method (24 in total) | `single_stage_base_network.py`, `single_stage_base_network_batch.py`, `two_stage_base_network.py`, and `<method>/build_whole_network*.py` |
| `losses/` | Loss functions; subclasses of `losses.Loss` | `losses.py` (focal, smooth-L1, IoU-smooth-L1 log/exp, modulated), `losses_csl.py`, `losses_dcl.py`, `losses_gwd.py` (GWD and ProbIoU variants), `losses_kl.py` (KLD, Jeffreys, JS), `losses_kfiou.py` (Kalman-filter IoU, one- and two-stage), `losses_bcd.py` (Bhattacharyya), `losses_rsdet.py`, `losses_ridet.py` (Hungarian), `losses_fcos.py`, `losses_gwd_pytorch.py` (reference port) |
| `roi_extractors/` | RoI pooling / RoI align for two-stage detectors | `roi_extractors.py` |
| `box_heads/` | Second-stage classification/regression head | `box_head_base.py` |

**Detector packages and their graph builders**

| Package | Family | Files |
|---|---|---|
| `retinanet` | single-stage | `build_whole_network.py`, `_atan.py` (sin/cos angle), `_batch.py`, `_pb.py` |
| `refine_retinanet` | single-stage, cascaded | `build_whole_network.py` |
| `r3det`, `r3det_bcd`, `r3det_dcl`, `r3det_gwd`, `r3det_kfiou`, `r3det_kl` | single-stage + feature refinement | `build_whole_network.py` (+ `_pb.py` for `r3det`, `r3det_gwd`) |
| `csl`, `dcl` | single-stage, angle as classification | `build_whole_network.py` |
| `gwd`, `kl`, `kfiou`, `bcd` | single-stage, Gaussian-based regression loss | `build_whole_network.py` (+ `_batch.py`, `_pb.py` for some) |
| `atss` | single-stage, adaptive sample selection | `build_whole_network.py` |
| `rsdet` | single-stage, 5-parameter or 8-point + modulated loss | `build_whole_network_5p.py`, `build_whole_network_8p.py` |
| `ridet` | single-stage, 8-point + Hungarian loss | `build_whole_network_8p.py` |
| `fcos` | anchor-free | `build_whole_network_r.py`, `_batch_rbox.py`, `_batch_quad.py` |
| `r2cnn`, `r2cnn_bcd`, `r2cnn_kfiou`, `r2cnn_kl` | two-stage (Faster R-CNN with rotated head) | `build_whole_network.py` |
| `rrpn` | two-stage with rotated anchors / RRoI pooling | `build_whole_network.py` |
| `scrdet` | two-stage with attention neck | `build_whole_network.py` |

### `alpharotate/libs/utils/`

| File(s) | Responsibility |
|---|---|
| `bbox_transform.py` | Encode/decode horizontal and rotated box deltas (`rbbox_transform`, `rbbox_transform_inv`) |
| `coordinate_convert.py` | 8-point ↔ `(x, y, w, h, θ)` conversions (`forward_convert`, `backward_convert`), angle-range conversions, horizontal min-area rectangle |
| `iou.py`, `iou_rotate.py`, `iou_cpu.pyx`, `rbbox_overlaps.*` | Horizontal and rotated IoU in Python, Cython and CUDA |
| `nms.py`, `nms_rotate.py`, `rotate_polygon_nms.*`, `rotate_gpu_nms.hpp`, `nms_cython/`, `cython_utils/` | Horizontal and rotated NMS in TF, Cython and CUDA |
| `box_sample.py`, `mask_sample.py` | Sampling helpers |
| `draw_box_in_img.py`, `show_box_in_tensor.py` | Drawing boxes on images (NumPy/OpenCV) and in TensorBoard (`DrawBoxTensor`) |
| `setup.py` | Stand-alone in-place build of the CUDA/Cython extensions |

### `alpharotate/libs/val_libs/`

- `voc_eval.py`: PASCAL-VOC AP for horizontal boxes
- `voc_eval_r.py`: the same protocol with rotated IoU (used by HRSC2016, UCAS-AOD, DIOR-R, etc.)

### `alpharotate/libs/label_name_dict/label_dict.py`

This file maps class names to ids for every supported dataset (DOTA 1.0/1.5/2.0, HRSC2016, DIOR-R, UCAS-AOD, ICDAR,
COCO, …). It is keyed by `cfgs.DATASET_NAME`.

### `alpharotate/utils/`: stand-alone method implementations (documented in the Sphinx API)

| File | Content |
|---|---|
| `smooth_label.py` | **CSL**: circular smooth label with gaussian / rectangular / pulse / triangle windows (`angle_smooth_label`) |
| `densely_coded_label.py` | **DCL**: binary- and Gray-coded angle labels (`angle_label_encode` / `angle_label_decode`) |
| `gaussian_metric.py`, `gaussian_wasserstein_distance.py`, `gwd.py`, `kld.py`, `kf.py`, `bcd.py` | Box → 2-D Gaussian conversion and distances: GWD, KLD, KFIoU, Bhattacharyya, plus rotated IoU/DIoU variants |
| `hiou_riou.py`, `iou_approx_evar.py`, `iou_approx_evar_.py`, `iou.xlsx` | Experiments comparing IoU approximations. The spreadsheet appears to hold the related numeric data (*inferred*; it was not opened because it is binary) |
| `order_points.py`, `sort_polygon.py`, `quad2rbox.py`, `box_ops.py` | Polygon/quad utilities |
| `pretrain_zoo.py` | `PretrainModelZoo`: lists of TF / PyTorch / MXNet backbone names and resolution of pre-trained weight paths |
| `img2npy.py`, `tools.py` | Small helpers (image dump, `makedirs`, etc.) |

---

## `configs/`: experiments

- `cfgs.py`: the **active** config imported by every script (currently a RetinaNet-H + KFIoU DOTA experiment).
- `_base_/models/{retinanet_r50_fpn,faster_rcnn_r50_fpn}.py`, `_base_/datasets/dota_detection.py`,
  `_base_/schedules/schedule_1x.py`: shared defaults.
- `<DATASET>/<method>/cfgs_<backbone>_<dataset>_<method>_v<N>.py`: 207 experiment files. Many end with a docstring
  that contains the evaluation output (mAP and per-class AP) that the upstream authors obtained with that config.
  This is useful historical evidence of the benchmark results.
- `configs/DOTA/probiou/` has a config but no matching detector or tool folder. ProbIoU is implemented as a loss
  mode inside `losses_gwd.py` and the `gwd` detector.

## `dataloader/`: data preparation and input pipeline

| Path | Responsibility |
|---|---|
| `dataset/convert_data_to_tfrecord.py` | Reads VOC-style XML with polygon boxes + images → TFRecord (`tf.app.flags`: `root_dir`, `xml_dir`, `image_dir`, `save_name`, `save_dir`, `img_format`, `dataset`) |
| `dataset/read_tfrecord.py` | `ReadTFRecord`: queue-based reader, decode, augmentation, batching. Reads `../../dataloader/tfrecord/<DATASET>_train*` |
| `dataset/image_augmentation.py` | Short-side resize, flips, aspect-ratio jitter, RGB→gray, rotation |
| `dataset/DOTA/` | `data_crop.py`, `data_slicer.py`: tile large images and annotations; `val_set.txt` |
| `dataset/<other datasets>/` | Per-dataset converters to XML (`txt2xml.py`, `json2xml.py`), splitters and visualizers for DIOR-R, FDDB, HRSC2016, HRSID, ICDAR2015, MLT, MSRA-TD500, OHD-SJTU, SKU110K-R, SSDD++, Total_Text, UCAS-AOD |
| `dataset/{ICDAR2015,MSRA-TD500}/gt.zip`, `Total_Text/labels.zip` | Ground-truth archives used by the text-detection evaluators (data files, kept as is) |
| `pretrained_weights/` | Placeholder folders + READMEs describing where backbone weights must be placed; `efficientnet/*/results.txt` contain reference results |

## `tools/`: entry-point scripts

**Shared runners (top level)**

| File | Role |
|---|---|
| `train_base.py` | `Train` base class: TFRecord reader, warm-up + piecewise LR, tower gradient averaging, bias-gradient scaling, session loop, checkpoint/summary writing |
| `test_dota_base.py`, `test_dota_base_sota.py`, `test_dota_base_q.py`, `test_dota_base_rh.py`, `test_dota_base_sota_quad.py` | DOTA inference: tiling, multi-GPU processes, MS/flip TTA, merge + rotated NMS, DOTA Task-1 txt output |
| `test_hrsc2016_base*.py`, `test_dior_r_base*.py`, `test_ucas_aod_base.py`, `test_icdar2015_base.py`, `test_mlt_base.py`, `test_msra_td500_base.py`, `test_fddb_base.py` | Dataset-specific inference and evaluation |
| `test_speed_base.py` | Throughput measurement |
| `swa.py` | Averages the last *n* checkpoints (SWA object detection) |
| `demo.py` | Inference/visualization demo on a folder of images |

**Per-detector folders** (`tools/<method>/`, 24 in total) contain thin subclasses of the base runners that plug in the
detector-specific graph and post-processing. Typical files: `train.py`, `train_swa.py`, `train_batch*.py`,
`test_<dataset>.py`, `test_dota_sota.py`, `test_speed.py`, `exportPb.py`.

`tools/kl/test_msra_td500/RetinaNet_MSRA_TD500_KL_2x_20210215/msra_td500_res/res_img_0059.txt` is a single committed
**test output** from the upstream authors. It is kept in place as historical evidence.

## `demo/`

- `video2image.py`: extracts frames from videos in `./videos/` into `./images/` (comment in Chinese: "read frames from video and save as images").
- `video_maker.py`: reassembles frames into a video (used for the README demo GIF, *inferred*).

## `thirdparty/`

| Folder | Origin | Purpose |
|---|---|---|
| `DOTA_devkit/` | DOTA dataset authors | Polygon IoU (SWIG C++), GPU polygon NMS, DOTA Task-1 evaluation scripts |
| `icdar_msra_td500_devkit/` | ICDAR Robust Reading Competition | Stand-alone text-detection evaluation (`script.py -g=gt.zip -s=submit.zip`) |
| `gluon2TF/` | Jirui Yang ([original repo](https://github.com/yangJirui/gluon2TF)) | Converts GluonCV ResNet-v1b/v1d weights to TF checkpoints |

## Root files

| File | Role |
|---|---|
| `setup.py` | setuptools package `alpharotate` 1.0.1 with 5 CUDA/Cython extensions, pinned install requirements |
| `requirements.txt` | Only `cython`, `numpy` (build-time prerequisites for `setup.py`) |
| `MANIFEST.in` | Includes C/C++ sources in the sdist and prunes `configs` |
| `.gitignore` | Ignores weights, TFRecords, test outputs and several unpublished experiment names (`*redet*`, `*centernet*`, `*yolo*`, …). This suggests that the upstream authors had more experiments that were never released (*inferred*) |
