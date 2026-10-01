# Plan

← [intent.md](./intent.md) · [spec.md](./spec.md) · [Documentation index](../README.md)

> **Artifact type:** Plan (the *how*). This is the third artifact in the intent → spec → plan workflow.
> It records how the repository reorganization was carried out, the decisions taken, and how the result was
> verified. It covers the **repository**, not the detection code.

---

## 1. Phases

| # | Phase | Output | Status |
|---|---|---|---|
| 1 | **Discover**: inventory all files; read README, OTHERS.md, Sphinx docs, `setup.py`, configs, base classes, detectors, tools, devkits; inspect figures; read git history and remote | Evidence base for [project-context.md](../project-context.md) | Done |
| 2 | **Classify**: decide project origin from evidence | "Other: fork of a third-party open-source research codebase" | Done |
| 3 | **Specify**: reverse-engineer intent and behavior | [intent.md](./intent.md), [spec.md](./spec.md) | Done |
| 4 | **Restructure**: minimal, non-breaking file moves | See §3 | Done |
| 5 | **Document**: README + `docs/*.md` + `AGENTS.md` | See §4 | Done |
| 6 | **Verify**: checksums, link check, diff review | See §5 | Done |
| 7 | **Deliver**: commit as repository owner on branch `docs/repository-refactor`, push, open a PR | Branch ready for review | Done (PR to be opened by the owner) |

## 2. Key Decisions

| ID | Decision | Rationale |
|---|---|---|
| D-1 | **Do not move code into `src/`** or rename any code directory | Scripts depend on CWD-relative paths (`ROOT_PATH = abspath('../../')`, `sys.path.append("../../")`, `'../../dataloader/tfrecord'`). `setup.py` hard-codes extension paths. Docstrings reference `../../images/*.png`. Moving any of these would break the original implementation |
| D-2 | **Keep `images/` at the root** | Referenced by code docstrings (`alpharotate/utils/smooth_label.py`, `densely_coded_label.py`) and the Sphinx guide |
| D-3 | **Keep `docs/` Sphinx sources in place** and add Markdown next to them | Sphinx only builds `.rst` (`source_suffix = '.rst'`), so the new `.md` files do not interfere |
| D-4 | **Move the upstream README and OTHERS.md to `docs/original/`** unchanged | The new root README replaces them as the entry point. The originals stay available verbatim, and a mapping note explains their root-relative links |
| D-5 | **Move `example.xml` to `docs/examples/`** | It is a documentation sample of the annotation format. No code references it |
| D-6 | **Remove the 7 tracked `.DS_Store` files** | macOS Finder metadata with no project information. Already listed in the original `.gitignore`. No directory became empty |
| D-7 | **Keep the committed test output** `tools/kl/test_msra_td500/.../res_img_0059.txt`, data archives (`*.zip`), `iou.xlsx`, `results.txt` files | Historical outputs and data. Preservation takes priority over cleanup |
| D-8 | **Do not change `.gitignore`, `setup.py`, `requirements.txt`, `MANIFEST.in`, `LICENSE`** | They are part of the original project's build and packaging |
| D-9 | **Add no new infrastructure** (Docker, CI, linters, tests, Makefile) | Out of scope by intent. The project never had them |
| D-10 | **Add `AGENTS.md`** | Records the preservation rules in the conventional place where automated contributors look first |
| D-11 | **Attribute authorship to the upstream authors** in every new document | Evidence shows that all code is upstream work. Apache-2.0 requires that attribution be preserved |

## 3. File Operations

| Operation | From | To |
|---|---|---|
| move | `README.md` | `docs/original/README.upstream.md` |
| move | `OTHERS.md` | `docs/original/OTHERS.md` |
| move | `example.xml` | `docs/examples/example.xml` |
| delete | `dataloader/.DS_Store`, `dataloader/pretrained_weights/.DS_Store`, `dataloader/pretrained_weights/darknet/.DS_Store`, `output/trained_weights/.DS_Store`, `tools/.DS_Store`, `tools/retinanet/.DS_Store`, `tools/rsdet/.DS_Store` | — |
| add | — | `README.md` (new), `AGENTS.md` |
| add | — | `docs/README.md`, `docs/project-context.md`, `docs/architecture.md`, `docs/code-overview.md`, `docs/usage.md`, `docs/possible-improvements.md` |
| add | — | `docs/original/README.md`, `docs/sdlc/{intent,spec,plan}.md` |

No other path was touched.

## 4. Resulting Layout

```
.
├── README.md                 new
├── AGENTS.md                 new
├── LICENSE  setup.py  MANIFEST.in  requirements.txt  .gitignore      original
├── alpharotate/  configs/  dataloader/  tools/  demo/  thirdparty/   original (unchanged)
├── images/  output/                                                 original (unchanged)
└── docs/
    ├── README.md  project-context.md  architecture.md  code-overview.md
    ├── usage.md  possible-improvements.md                           new
    ├── sdlc/ intent.md spec.md plan.md                              new
    ├── examples/example.xml                                         moved
    ├── original/ README.md (new note) README.upstream.md OTHERS.md  moved
    └── index.rst conf.py Makefile make.bat requirements.txt api/ guide/ _templates/ images/   original
```

## 5. Verification

| Check | Method | Result |
|---|---|---|
| Source code unchanged | SHA-256 of every tracked non-Markdown, non-moved file at `6adfd2d` vs. the branch head (`git diff --stat 6adfd2d -- . ':!*.md'` lists only the `.DS_Store` deletions and the `example.xml` rename) | No content changes |
| Moved files unchanged | `git diff -M100% --stat 6adfd2d` shows 100% similarity renames | Pass |
| Relative links in new docs resolve | Script that extracts `](./…)` / `](../…)` targets from new Markdown and checks that they exist | Pass |
| No fabricated commands | Every command in README/usage.md is quoted from upstream docs or argparse/flags in code | Pass |
| Execution | Not attempted: TF 1.13 / CUDA 10 / GPU unavailable | Documented as Unverified |

## 6. Follow-ups (not part of this change)

- Optionally mirror selected items from [possible-improvements.md](../possible-improvements.md) as GitHub issues.
- If the code is ever revived, do it on a separate branch or project, so that this branch stays a faithful historical snapshot.
