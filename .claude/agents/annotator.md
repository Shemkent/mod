# Annotator
**Model:** sonnet

Annotate EU5 vanilla game systems for modder reference.

## Key files
| File | Purpose |
|---|---|
| `annotations/[system].md` | Output — human-readable reference |
| `annotations/_file_index.csv` | `file, systems, stage, game_version` |
| `annotations/GAME_STRUCTURE_GUIDE.md` | System index, domain clusters, priority tiers |

## Workflow
Run both phases together when scope fits in one pass.

**Phase A — Map**
1. Read plain-text game script files in scope (skip binaries, images, JSON assets)
2. Identify system(s) per file; note block types and keywords
3. Add rows to `_file_index.csv` (`stage=mapped`); `|`-separate multiple systems
4. Add/expand system entry in `GAME_STRUCTURE_GUIDE.md`

**Phase B — Annotate**
1. Read files not yet at `stage=annotated`
2. Write `[system].md` using the format below
3. Mark rows `stage=annotated` in `_file_index.csv`
4. Set stage header in `[system].md` to `complete`
5. Commit on `vanilla-annotation`: `annotate: [system name]`
6. Recommend 1–2 next systems from the same domain cluster that fit a single session

## Annotation format
```markdown
# System Name
**Stage:** complete
**Keywords:** top-level identifiers

> **System type: [Gameplay | AI | UI/Presentation | Scripted Logic | Data/Reference]**

## Overview
What it does and why modders care (one paragraph).

## Vanilla File Locations
Folder and key files. Full list in `_file_index.csv`.

## Block Structure
Syntax skeleton with inline comments. For large blocks: list categories, show 1–2 entries only.

## Key Fields Reference
Table: field | purpose | key constraint. Combine related fields; skip self-evident ones.

## Modding Notes
- Load order, override approach
- Risky vs. safe fields
- Cross-system dependencies
- Non-obvious patterns

## Example
One representative vanilla block with brief commentary.
```

## System types
- **Gameplay** — rules, content, mechanics
- **AI** — AI decisions
- **UI/Presentation** — display only, no gameplay logic
- **Scripted Logic** — shared triggers/effects
- **Data/Reference** — static lookup tables
