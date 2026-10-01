# AGENTS.md

Guidance for automated contributors (coding agents, bots, scripts) and for human reviewers of their work in this repository.

## What this repository is

A preserved fork of **AlphaRotate** (TensorFlow 1.x rotated object detection benchmark) by Xue Yang et al., Shanghai
Jiao Tong University, Apache-2.0. Start with [README.md](./README.md), then [docs/README.md](./docs/README.md).

## Hard rules

1. **Do not modify original source code.** This covers every `*.py`, `*.pyx`, `*.cu`, `*.cpp`, `*.c`, `*.h`, `*.hpp`, `*.i`,
   `*.sh`, `Makefile`, `setup.py`, `requirements.txt`, `MANIFEST.in`, `.gitignore`, and everything under `alpharotate/`,
   `configs/`, `dataloader/`, `tools/`, `demo/` and `thirdparty/`. No formatting, renaming, fixes or dependency bumps.
2. **Do not move code directories.** Scripts depend on CWD-relative paths (see [docs/sdlc/plan.md](./docs/sdlc/plan.md), decision D-1).
3. **Do not edit files in `docs/original/`.** They are verbatim upstream documents.
4. **Do not delete** outputs, archives or data files (`*.zip`, `*.xlsx`, `results.txt`, committed test results).
5. **Do not add infrastructure** (Docker, CI, linters, formatters, test frameworks, package managers, Makefiles).
6. **Keep attribution.** Code authorship belongs to the upstream authors. Never present it as new or as written by the fork owner.
7. **No invented facts.** Label claims as Confirmed, Inferred or Unknown. Never invent commands, versions or results.

## Allowed changes

- Markdown documentation outside `docs/original/` (`README.md`, `docs/*.md`, `docs/sdlc/*.md`, this file).
- Proposed improvements go into [docs/possible-improvements.md](./docs/possible-improvements.md), never into the code.

## Workflow

Follow intent → spec → plan:

1. Update or add the *why* in [docs/sdlc/intent.md](./docs/sdlc/intent.md).
2. State *what* changes in [docs/sdlc/spec.md](./docs/sdlc/spec.md).
3. Record *how* and the verification in [docs/sdlc/plan.md](./docs/sdlc/plan.md).
4. Work on a dedicated branch and open a pull request. Never push directly to `main`.

## Verification before you commit

```shell
# Must print nothing: no code file content may differ from the upstream snapshot.
git diff --name-only --diff-filter=M 6adfd2d -- . ':!*.md'
```

Also check that every relative link in the Markdown files you changed resolves to an existing path.
