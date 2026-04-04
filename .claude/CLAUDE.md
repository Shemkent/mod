# EU5 Modding Project
**Game version:** 1.1.10

## Structure
- `annotations/` — vanilla system references
- `mod/`

## Git
- Before committing, pause and confirm with the user.
- `main` — stable
- `vanilla-annotation` — annotation work; PR to main when complete. Read `.claude/annotator.md`.
- `mod-dev` — mod development. Read `.claude/modder.md`.

## Planning
- start with the fewest deliverables that solve the problem. Present that. User may ask more. Each iteration should converge toward fewer moving parts, not more.

## Agents
- **Writing review:** after writing an annotation, spawn `reviewer` agent on the output

## Memory
Do not write new files to memory directory (`~/.claude/projects/.../memory/`). `feedback_commits.md` only permitted. Project knowledge lives in `CLAUDE.md`, `.claude/annotator.md`, `.claude/modder.md`, and `annotations/`.
