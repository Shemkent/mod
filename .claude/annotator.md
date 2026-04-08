# Annotation Guide

Working on `vanilla-annotation` branch? Read this.

## Key files
| File | Purpose |
|---|---|
| `annotations/[system].md` | Output — human-readable reference |
| `annotations/_file_index.csv` | `file, systems, stage, game_version` |
| `annotations/_system_index.json` | Graph nodes: system id, stage, cluster, version, summary |
| `annotations/_system_edges.json` | Graph edges: cross-system relationships |
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
5. Update `_system_index.json` (add/update node) and `_system_edges.json` (add edges) — see Graph maintenance
6. Commit on `vanilla-annotation`: `annotate: [system name]`
7. Spawn `reviewer` with prompt: `Review annotations/[system].md for prose quality`
8. Recommend 1–2 next systems from the same domain cluster that fit a single session

## Annotation format
```markdown
# System Name
**Stage:** complete
**Keywords:** top-level identifiers

> **System type: [Gameplay | AI | UI/Presentation | Scripted Logic | Data/Reference]**
> **Cluster**: see GAME_STRUCTURE_GUIDE.md.

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

## Completeness criteria
An annotation is `complete` when: all major fields documented, system type marker present, at least one example, all source files marked `annotated` in `_file_index.csv`, `GAME_STRUCTURE_GUIDE.md` entry links to the annotation, node added to `_system_index.json`, edges added to `_system_edges.json`.

## Graph maintenance

When completing an annotation, also update the relationship graph files:

1. **`annotations/_system_index.json`** — add a node entry:
   ```json
   {
     "id": "<system_id>",
     "label": "<Human Label>",
     "annotation": "<system>.md",
     "stage": "complete",
     "cluster": "<cluster_id>",
     "verified_version": "<game_version from CLAUDE.md>",
     "summary": "<one-line summary>"
   }
   ```
2. **`annotations/_system_edges.json`** — add edges for every cross-system reference in the annotation's Modding Notes:
   ```json
   { "from": "<this_system>", "to": "<target_system>", "type": "<edge_type>", "note": "<description>" }
   ```
   Edge types:
   - `registers` — this system lists IDs defined in another (e.g. religions list religious_focuses)
   - `references` — this system uses IDs from another (e.g. cultures reference a language)
   - `shared_mechanic` — same field pattern as another system (add `"bidirectional": true`); e.g. opinions blocks
   - `modifies` — this system pushes values into another (e.g. traits modify societal_values via monthly_towards_*)
   - `hierarchy` — this system contains another (e.g. language_families contain languages)

## Game version update workflow

When the game updates (new patch):
1. Update `CLAUDE.md` game version
2. Diff source files to identify changed systems
3. Re-verify affected annotations against new source files
4. Bump `verified_version` in `_system_index.json` for re-checked systems
5. Systems where `verified_version` < current game version need re-verification

## Data pipeline

```
Game Source Files (in_game/common/*)
    ↓  annotator reads, reviewer verifies
Annotation .md files (self-contained, cross-refs in Modding Notes prose)
    +
_system_index.json  ← nodes (id, stage, cluster, verified_version, summary)
_system_edges.json  ← edges (from, to, type, note)
    ↓  consumed by
Interactive Graph Viewer (future)
    ↓  static fallback
Mermaid diagram in GAME_STRUCTURE_GUIDE.md (future)
```
