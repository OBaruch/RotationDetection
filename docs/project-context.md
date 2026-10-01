# Project Context

← Back to [README](../README.md) · [Documentation index](./README.md)

This document puts together the evidence about where this repository came from, what it was built for, and what its
scope is. Each statement is labelled **Confirmed**, **Inferred** or **Unknown**.

## Classification

**Project origin: Other: fork of a third-party open-source research codebase.**

| Candidate category | Verdict | Reason |
|---|---|---|
| Academic / University Project | Not for this fork | The *upstream* project is university research (Shanghai Jiao Tong University), but no evidence links the fork owner to it |
| Coursework / Assignment | No evidence | No assignment statement, rubric, course name, report or deliverable exists |
| Personal Project | Not supported | No commits, code or results by the fork owner |
| Research / Exploration | Possible (inferred) | Keeping a fork of a benchmark is consistent with studying the field, but it cannot be confirmed |
| Proof of Concept / Technical Experiment | No evidence | No experiment-specific code or results by the fork owner |

## Evidence

### Confirmed

| Evidence | Source |
|---|---|
| Git history: 51 commits, authors `yangxue` (48) and `erenzhou` (3), 2021-09-15 → 2022-08-03 | `git log` |
| The earliest commit in the history is titled "update user guide in docs", so the imported history does not start at the project's actual beginning | `git log --reverse` |
| Package name `alpharotate`, version `1.0.1`, author `yangxue0827`, URL `github.com/yangxue0827/RotationDetection` | `setup.py` |
| Source headers: `Author: Xue Yang <yangxue-2019-sjtu@sjtu.edu.cn>`, `Copyright (c) SJTU` | e.g. `tools/train_base.py`, `alpharotate/utils/gaussian_metric.py` |
| Maintained by Xue Yang (SJTU), supervised by Prof. Junchi Yan | `docs/original/README.upstream.md`, `docs/index.rst` |
| Sphinx documentation copyright "2021, Xue Yang" | `docs/conf.py` |
| Associated paper: *AlphaRotate: A Rotation Detection Benchmark using TensorFlow*, arXiv:2111.06677 | Citation block in the upstream README |
| Apache-2.0 license | `LICENSE`, `setup.py` |
| Git remote: `github.com/OBaruch/RotationDetection` | `git remote -v` |
| Third-party components: DOTA devkit, ICDAR "RRC" evaluation scripts, `gluon2TF` by Jirui Yang, TF-slim nets, EfficientNet/MobileNet code from TensorFlow | `thirdparty/*/README*`, file headers |

### Inferred

- The repository is a **GitHub fork** of `yangxue0827/RotationDetection`, taken at or after upstream commit `6adfd2d`
  (2022-08-03).
- The fork was kept as a **reference copy** for studying or experimenting with rotated object detection.
- The upstream project moved on to PyTorch/MMRotate. The README's news banner says "MMRotate has been released",
  so this TensorFlow codebase is the final state of a line of work that was being retired.

### Unknown

- Why the fork was created, and whether it was used for a course, a thesis, a job task or personal learning.
  *The original repository does not provide enough information to determine this.*
- Whether any training or evaluation was ever run from this fork. No checkpoints, logs or result files from the fork
  owner exist.

## Upstream Project Summary

**Goal.** From `docs/index.rst`:

> AlphaRotate is an open-source Tensorflow benchmark for performing scalable rotation detection on various datasets.
> It is developed for (1) providing modules for developing rotation detection algorithms, (2) providing
> implementation of state-of-the-art rotation detection methods, and (3) benchmarking existing rotation detection
> algorithms under different dataset & experiment settings, for the purpose of fair comparison.

**Application domains** (from `docs/guide/introduction.rst`, illustrated in `docs/images/`): aerial images, scene text,
faces, 3D object detection and retail scenes. The focus is on aerial images "due to its challenging".

**Scope in numbers** (counted from the repository):

| Item | Count |
|---|---|
| Python files | ~790 |
| Detector packages (`alpharotate/libs/models/detectors/*`) | 24 |
| Per-detector tool folders (`tools/*/`) | 24 |
| Experiment configs (`configs/**/cfgs_*.py`) | 207 |
| Dataset folders under `configs/` | 11 (DOTA, DOTA1.5, DOTA2.0, DIOR-R, HRSC2016, ICDAR2015, MLT, MSRA-TD500, UCAS-AOD, FDDB, Total_Text) |
| Dataset-preparation folders under `dataloader/dataset/` | 13 (adds HRSID, OHD-SJTU, SKU110K-R, SSDD++; no configs for these) |

## Timeline (from git history)

| Period | Activity (commit subjects) |
|---|---|
| 2021-09 | KLD support, Gaussian metric utilities, user guide |
| 2021-10 – 2021-12 | Most active period (21 commits in Nov): documentation, installation, PyPI, news |
| 2022-02 | KFIoU support, config path updates, MMRotate announcement, BibTeX |
| 2022-03 | CSL supported in MMRotate, import bug fix, installation tips |
| 2022-06 | GPU NMS fix ("gpu nms worse than cpu nms"), dev-environment update |
| 2022-07 – 2022-08 | `iou_approx_evar_.py`, "add bcd" (final commit) |

## Benchmark Results Recorded Upstream

The upstream README reports DOTA v1.0 / v1.5 / v2.0 mAP for each method, all with a ResNet50-v1d FPN baseline.
Selected rows (full table: [original README](./original/README.upstream.md#latest-performance)):

| Method | Base detector | DOTA1.0 | DOTA1.5 | DOTA2.0 |
|---|---|---|---|---|
| — | RetinaNet-H | 64.17 | 56.10 | 43.06 |
| CSL | RetinaNet-H | 67.38 | 58.55 | 43.34 |
| GWD | RetinaNet-H | 68.93 | 60.03 | 46.65 |
| KLD | RetinaNet-H | 71.28 | 62.50 | 47.69 |
| KFIoU | RetinaNet-H | 70.64 | 62.71 | 48.04 |
| KLD | R³Det | 71.73 | 65.18 | 50.90 |
| — | R²CNN (Faster R-CNN) | 72.27 | 66.45 | 52.35 |
| R³Det-BCD (SOTA, R152, MS+aug) | — | 79.95 | — | — |

These numbers are the **upstream authors' reported results**. They were not reproduced in this fork. Many config files
also contain a docstring with the per-class DOTA evaluation output for that exact config.
