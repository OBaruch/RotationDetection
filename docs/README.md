# Documentation Index

← Back to [README](../README.md)

This folder contains two separate kinds of documentation:

1. **Modern documentation (Markdown)**, added during the repository reorganization. It describes the original project
   and does not change it.
2. **Original documentation**: the upstream Sphinx / Read the Docs sources and the upstream README. These are kept unchanged.

## Modern documentation

| Document | Content |
|---|---|
| [project-context.md](./project-context.md) | Origin, evidence (Confirmed / Inferred / Unknown), timeline, upstream results |
| [architecture.md](./architecture.md) | Component map, configuration model, detector families, training and inference data flow, native extensions |
| [code-overview.md](./code-overview.md) | What every package, folder and important script does |
| [usage.md](./usage.md) | Install, data preparation, training and testing, as documented upstream (not verified) |
| [possible-improvements.md](./possible-improvements.md) | Observed technical debt and modernization ideas (**not applied**) |
| [sdlc/intent.md](./sdlc/intent.md) | Why the project exists and why it was reorganized |
| [sdlc/spec.md](./sdlc/spec.md) | Reverse-engineered functional and technical specification of the existing system |
| [sdlc/plan.md](./sdlc/plan.md) | Reorganization plan, decisions, file moves and verification |
| [examples/example.xml](./examples/example.xml) | Annotation-format sample (8-point polygon boxes in VOC-style XML) |

## Original documentation (unchanged)

| Source | Content |
|---|---|
| [original/README.upstream.md](./original/README.upstream.md) | Upstream README: feature list, full DOTA benchmark tables, model download links, install/train/test instructions, citation |
| [original/OTHERS.md](./original/OTHERS.md) | Export to frozen graph, SWA, PyTorch→TF weight conversion with MMdnn |
| `index.rst`, `guide/*.rst`, `api/src.rst`, `conf.py` | Sphinx sources published upstream on Read the Docs |
| `images/` | Application-domain figures used by `guide/introduction.rst` |

See [original/README.md](./original/README.md) for notes on relative links inside the preserved files.
