# Intent

← [Documentation index](../README.md) · Next: [spec.md](./spec.md) → [plan.md](./plan.md)

> **Artifact type:** Intent (the *why*). This is the first artifact in an intent → spec → plan workflow.
> **Mode:** reverse-engineered. It is written *after the fact* from what already exists in the repository. It does not
> describe new work on the detection system.

---

## 1. Intent of the original system (AlphaRotate)

**Problem.** Objects in aerial imagery, scene text, SAR ship imagery, faces and retail shelves appear at arbitrary
orientations. Horizontal bounding boxes cover them poorly, and the rotated-detection methods published around
2017–2022 were implemented in different, mutually incompatible codebases. That made fair comparison hard.

**Intended outcome** (Confirmed, `docs/index.rst`):

1. Provide reusable **modules** for building rotation detectors.
2. Provide **reference implementations** of state-of-the-art rotation detectors, including the authors' own CSL,
   DCL, GWD, KLD, KFIoU, BCD and R³Det.
3. **Benchmark** them under identical datasets, backbones, schedules and evaluation protocols.

**Primary users** (Inferred): computer-vision researchers and students who work on oriented object detection,
especially remote sensing (DOTA, DIOR-R, HRSC2016).

**Success signals recorded by the authors** (Confirmed, original README and config docstrings): DOTA v1.0 / v1.5 /
v2.0 mAP per method, with every number traceable to a specific config file and downloadable weights.

**Out of scope for the original system** (Inferred from the absence of related code): serving or deployment, a stable
public API, a training UI, and support for TensorFlow 2.

## 2. Intent of this repository (the fork)

| | |
|---|---|
| **Origin** | Fork of `yangxue0827/RotationDetection` at commit `6adfd2d` |
| **Owner contribution to the code** | None (Confirmed: all commits belong to upstream authors) |
| **Reason for the fork** | Unknown. *The original repository does not provide enough information to determine this.* |

## 3. Intent of the reorganization

**Goal:** turn the fork into a clear, navigable, honestly documented historical reference that someone can understand
without reading ~790 Python files, while keeping the original implementation exactly as it was.

**Principle:** *modernize the repository, not the project.*

### Must

- Keep every source file (`*.py`, `*.pyx`, `*.cu`, `*.cpp`, `*.h*`, `*.c`, `*.sh`, Makefiles, configs) byte-for-byte unchanged.
- Keep all original documents, outputs and data archives.
- Attribute the work correctly to its upstream authors, and respect the Apache-2.0 license.
- Mark every claim as Confirmed, Inferred or Unknown.
- Use English for all new documentation.

### Must not

- Refactor, reformat, fix or modernize code. Do not change dependencies.
- Present the fork as original work by the fork owner, or present the code as recent.
- Add infrastructure that the project never had (Docker, CI, linters, test frameworks, package managers, Makefiles).
- Invent context, commands, versions or results.

### Definition of done

- A new root `README.md` explains the project in under five minutes of reading.
- `docs/` contains context, architecture, code overview, usage, possible improvements, and these SDLC artifacts.
- The original documents are preserved under `docs/original/`.
- Checksums of every source file are identical before and after the change (see [plan.md](./plan.md#5-verification)).
- The work is delivered on a dedicated branch and is ready for a pull request.
