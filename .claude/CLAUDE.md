# EU5 Modding Project
**Game version:** 1.1.10 — update when game updates, re-check affected annotations.

## Structure
- `annotations/` — vanilla system references (read-only)
- `mod/` — one folder per mod
- Git tracks only `annotations/` and `mod/`

## Branches
- `main` — stable
- `vanilla-annotation` — annotation work; PR to main when complete
- `mod-dev` — mod development

## Mod conventions
- Mirror EU5 paths: `in_game/common/` and `main_menu/common/`
- Prefix mod files with a short mod ID (e.g. `BPW_country.txt`)
- Read `.metadata/README.md` before touching any mod files

## Agents
- **Annotation work:** spawn `annotator`, then `reviewer` on the output
- **Mod work:** spawn `modder`

## Memory
Do not write new files to the user memory directory (`~/.claude/projects/.../memory/`). The only permitted file there is `feedback_commits.md`. Project knowledge lives in `CLAUDE.md`, `agents/`, and `annotations/`.

## Completeness criteria
An annotation is `complete` when: all major fields documented, at least one example, all source files marked `annotated` in `_file_index.csv`, `GAME_STRUCTURE_GUIDE.md` entry links to the annotation.
