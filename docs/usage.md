# Usage (as documented upstream)

← Back to [README](../README.md) · [Documentation index](./README.md)

> **Status: not verified in this fork.** This page restates the upstream authors' procedure from the
> [original README](./original/README.upstream.md), [OTHERS.md](./original/OTHERS.md) and the Sphinx guide
> (`docs/guide/*.rst`), and adds constraints found in the code. Nothing here was executed during the reorganization.

## 1. Environment

The upstream "development environment" (from the original README):

| Component | Version |
|---|---|
| Python | 3.5 (Anaconda recommended) |
| CUDA | 10.0 (setup docs target CUDA < 11) |
| tensorflow-gpu | 1.13 (`setup.py` pins `1.13.1`) |
| opencv-python | 4.1.1.26 ("important") |
| Shapely | 1.7.1 |
| tqdm | 4.54.0 |
| tfplot | 0.2.0 (optional) |

Upstream alternatives:

- Docker image `yangxue2docker/yx-tf-det:tensorflow1.13.1-cuda10-gpu-py3`
- For RTX 30xx (CUDA ≥ 11), an NVIDIA TF1 container such as `nvcr.io/nvidia/tensorflow:20.11-tf1-py3`
- `pip install alpharotate` from PyPI ("not suitable for dev")

## 2. Install and compile extensions

```shell
pip install -r requirements.txt
pip install -v -e .            # or: python setup.py develop
```

Manual in-place build (CUDA ≥ 11 path in the upstream docs):

```shell
cd alpharotate/libs/utils/cython_utils
rm *.so *.c *.cpp
python setup.py build_ext --inplace   # or: make

cd alpharotate/libs/utils/
rm *.so *.c *.cpp
python setup.py build_ext --inplace
```

Caveats observed in the code:

- `setup.py` locates `nvcc` when it is imported, and raises `EnvironmentError` if CUDA is absent.
- The CUDA kernels are compiled with `-arch=sm_35`.

## 3. Pre-trained backbone weights

Place the weights in `dataloader/pretrained_weights/` (layout in
[`dataloader/pretrained_weights/README.md`](../dataloader/pretrained_weights/README.md)):

1. **MXNet/GluonCV ResNet-v1d/v1b** (upstream default, `NET_NAME = 'resnet50_v1d'`), converted with `thirdparty/gluon2TF`
2. TF-slim ResNet, EfficientNet, MobileNetV2, DarkNet53
3. PyTorch torchvision models, converted with MMdnn (see [OTHERS.md](./original/OTHERS.md))

The download links (Baidu Drive / Google Drive) are listed in the original README. Their availability has not been checked.

## 4. Choose an experiment

1. Pick `#DETECTOR` and `#DATASET` (for example `retinanet` and `DOTA`).
2. Edit `configs/#DATASET/#DETECTOR/cfgs_xxx.py` (`CLASS_NUM`, `DATASET_NAME`, `VERSION`, …).
3. Copy it over `configs/cfgs.py`.
4. For a new dataset, add its classes to `alpharotate/libs/label_name_dict/label_dict.py` and its name to
   `dataloader/dataset/read_tfrecord.py`.

## 5. Build TFRecords

```shell
# Large images (DOTA): crop first
cd dataloader/dataset/DOTA
python data_crop.py

# Convert XML annotations (format: docs/examples/example.xml) to TFRecord
cd dataloader/dataset
python convert_data_to_tfrecord.py --root_dir='/PATH/TO/DOTA/' \
                                   --xml_dir='labeltxt' --image_dir='images' \
                                   --save_name='train' --img_format='.png' --dataset='DOTA'
```

The upstream README links `example.xml` at the repository root. In this reorganized repository it is at
[`docs/examples/example.xml`](./examples/example.xml).

## 6. Train

```shell
cd tools/#DETECTOR
python train.py
```

Scripts **must be run from inside `tools/<detector>/`**. `ROOT_PATH` in the base config is `os.path.abspath('../../')`.

`SAVE_WEIGHTS_INTE` defines one "epoch" in iterations: DOTA1.0 = 27000, DOTA1.5 = 32000 and DOTA2.0 = 40000 for single-GPU
training. Upstream recommends multiplying it by 2 for multi-GPU training.

## 7. Test

```shell
# Large images (DOTA): outputs in tools/#DETECTOR/test_dota/<VERSION>/
python test_dota.py --test_dir='/PATH/TO/IMAGES/' --gpus=0,1,2,3 [-ms] [-s] [-cn]
python test_dota_sota.py --test_dir='/PATH/TO/IMAGES/' --gpus=0,1,2,3 [-s] [-cn]   # upstream-recommended

# Small images (HRSC2016)
python test_hrsc2016.py --test_dir='/PATH/TO/IMAGES/' --gpu=0 --image_ext='bmp' \
                        --test_annotation_path='/PATH/TO/ANNOTATIONS' [-s]
```

`-ms` = multi-scale test, `-s` = save visualizations, `-cn` = CPU NMS (slightly better, slower).
Result files are opened in `a+` mode, so you must delete old results before you re-test the same `VERSION`.

## 8. Extras (from OTHERS.md)

- **Export frozen graph:** `cd tools/retinanet && python exportPb.py`
- **SWA:** train *n* extra epochs with `train_swa.py`, average the weights with `tools/swa.py`, then test.
- **TensorBoard:** `cd output/summary && tensorboard --logdir=.`
