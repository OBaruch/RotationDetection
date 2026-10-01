# Possible Improvements

← Back to [README](../README.md) · [Documentation index](./README.md)

> **None of the items below were applied.** This repository preserves the original implementation. The source code
> has intentionally not been refactored or modernized, in order to retain the historical context and the original
> development approach. This page only records observations for anyone who wants to build on the code in a separate
> branch or project.

## 1. Runtime and Dependencies

| Observation | Evidence | Possible direction |
|---|---|---|
| TensorFlow 1.13, Python 3.5 and CUDA 10.0 are end-of-life. `tf.contrib`, `tf.py_func` and queue runners do not exist in TF 2.x | `setup.py`, original README, `import tensorflow.contrib.slim` | Run inside the upstream Docker image, or port to TF2 / PyTorch. The upstream authors chose the latter: [MMRotate](https://github.com/open-mmlab/mmrotate) |
| `requirements.txt` lists only `cython` and `numpy`, while `setup.py` pins TF, OpenCV, Shapely and tqdm | both files | Single source of truth for dependencies; lock file |
| `setup.py` calls `locate_cuda()` at import time and fails without `nvcc` | `setup.py` | Lazy CUDA detection, optional GPU extensions |
| CUDA kernels compiled for `-arch=sm_35` only | `setup.py` | Configurable / multi-arch `-gencode` flags |

## 2. Configuration and Paths

| Observation | Evidence | Possible direction |
|---|---|---|
| `ROOT_PATH = os.path.abspath('../../')` depends on the current working directory | `configs/_base_/models/*.py` | Resolve paths from `__file__` |
| Scripts use `sys.path.append("../")` / `("../../")` instead of package imports | `tools/**/*.py` | Installable package + console entry points |
| Experiments are selected by *copying* a file over `configs/cfgs.py` | upstream workflow | `--config` CLI argument |
| ~69 hard-coded absolute dataset paths (`/data/dataset/...`) as defaults in `tools/` and `dataloader/` | grep `/data/` | Required CLI args or environment variables |
| Configs use star-imports, and the effective value of a constant depends on import order | `configs/cfgs.py` | Explicit config objects (YAML/dataclass) |

## 3. Code Structure

- Large amounts of near-duplicate code: 24 `tools/<method>/` folders and many `test_*_base*.py` variants differ in small
  details. Some of this could be merged with a registry pattern, as MMDetection/MMRotate later did.
- Vendored third-party code (TF-slim nets with their unit tests, EfficientNet, gluon2TF, DOTA devkit) lives inside the
  package tree. It could be separated into dependencies or git submodules.
- `print(ROOT_PATH)` runs as a side effect when the module is imported (`retinanet_r50_fpn.py`).
- Typos in identifiers are part of the public surface (`WARM_SETP`, `MUTILPY_BIAS_GRADIENT`, `postprocess_detctions`,
  `samper.py`). Renaming them would break configs, so this would need a deprecation path.

## 4. Testing and Reproducibility

- No automated tests for the project's own code and no CI.
- Test-result files are opened in `a+` mode, so re-running the same `VERSION` silently appends to old results
  (upstream documents this as a known behavior).
- Pre-trained and trained weights depend on Baidu/Google Drive links that could disappear.

## 5. Repository Hygiene

- `.gitignore` contains `tools/*/test_masra_td500/*` (typo for `msra`), which is why
  `tools/kl/test_msra_td500/.../res_img_0059.txt` was committed upstream. The file was kept as historical evidence.
- `.gitignore` lists `*.DS_Store*` twice and `*.pb` twice.
- The original README has broken or stale references, which were preserved unchanged in
  [`docs/original/README.upstream.md`](./original/README.upstream.md):
  - `$PATH_ROO./configs/...` (typo for `$PATH_ROOT/configs/...`)
  - `output/pretained_weights` (typo; the folder is `output/trained_weights`)
  - `./dataloader/pretrained_weights/pretrain_zoo.py` (the file is `alpharotate/utils/pretrain_zoo.py`)
  - `./libs/models/backbones/resnet_pytorch.py` in OTHERS.md (the file is `alpharotate/libs/models/backbones/resnet_pytorch.py`)
- The Sphinx guide (`docs/guide/experiment.rst`) still refers to the older `libs/configs/` layout.
