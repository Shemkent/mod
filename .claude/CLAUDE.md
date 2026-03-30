# EU5 Modding Project Guidelines

## Game Version
Current: **1.1.10**
Update this line when the game updates, then re-check affected annotations.

## Project Structure
- `annotations/` — Reference guides for vanilla game systems (read-only reference)
- `mod/` — All mods live here, one folder per mod
- Git tracks only `annotations/` and `mod/`; all game files are ignored

## Starting a New Mod
Every mod must have a `.metadata/` folder with the following:

```
mod/[mod_name]/
├── .metadata/
│   ├── metadata.json       # EU5 mod descriptor (required by game)
│   ├── thumbnail.png       # Mod image
│   ├── README.md           # Objectives, requirements, feature list ← START HERE
│   ├── design_notes.md     # Design decisions and rationale
│   └── changelog.md        # Version history
├── in_game/
│   └── common/
└── main_menu/
    └── common/
```

When working on a mod, **read `.metadata/README.md` first**. It contains:
- What the mod is trying to achieve (objectives)
- Specific requirements and constraints
- Feature list and scope

When implementing changes:
1. Check `.metadata/README.md` for objectives — understand intent before touching files
2. Check `annotations/[relevant_system].md` to understand which vanilla files to modify
3. After making changes, update `.metadata/design_notes.md` with decisions made
4. After releasing a version, update `.metadata/changelog.md`

## Working with Annotations

- `annotations/` files document vanilla game systems — read them to understand game mechanics
- Do NOT annotate vanilla game files directly (files live outside the repo)
- Always check `annotations/GAME_STRUCTURE_GUIDE.md` first — it lists known systems and their keywords
- Check `annotations/_file_index.csv` to see which files are already mapped/annotated before reading any vanilla files
- The system name in `_file_index.csv` must exactly match the `[system].md` filename slug (lowercase, underscores)
- Read only plain-text game script files. Skip binary, image, and JSON asset files
- When a system references another (e.g. buildings reference modifier keys), cross-link to the relevant annotation
- Update the `[system].md` stage header whenever the annotation advances

## Creating new annotation

### Annotation artifacts per system
| File | Purpose |
|---|---|
| `annotations/[system].md` | Main annotation — human-readable reference |
| `annotations/_file_index.csv` | Master index of all vanilla files, their systems, and stage |
| `annotations/GAME_STRUCTURE_GUIDE.md` | Index of identified systems and their keywords |

### Stage label in `[system].md`
Every annotation file must have these lines immediately after the title:
```
# System Name
**Stage:** stub
**Keywords:**
```
`Keywords` lists the top-level block identifiers found in the vanilla files — filled in during Phase A, left blank on stubs.

Valid stages:
- `stub` — file created, no content yet
- `files-identified` — relevant files listed in `_file_index.csv`, GAME_STRUCTURE_GUIDE.md updated
- `in-progress` — annotation begun but not finished
- `complete` — all fields documented, examples included

### Phase A — File Discovery
Do this when starting a new system or when `_file_index.csv` has `unmapped` rows in scope.

1. User specifies a scope (folder, feature area, or explicit file list)
2. Read only plain-text game script files within that scope — skip binaries, images, JSON assets
3. For each file, identify which system(s) it belongs to and note the key block types / top-level keywords
4. Update `annotations/_file_index.csv`:
   - Add the file row if absent; set `stage` to `mapped`, record `game_version` as current version
   - Use `|` to separate multiple systems in the `systems` field (e.g. `scripted_effects|scripted_triggers`)
5. Update `annotations/GAME_STRUCTURE_GUIDE.md` — add or expand the system entry with folder path and keywords found
6. Update `[system].md` stage to `files-identified` once all files for that system are mapped
7. Do NOT write full annotation content yet — Phase A is mapping only

### Phase B — Annotation
Do this once Phase A is complete for a system.

1. Check `annotations/GAME_STRUCTURE_GUIDE.md` to confirm the system entry exists
2. Filter `annotations/_file_index.csv` for rows where `systems` contains `[system]` — this is your reading list
3. Skip rows already at `annotated` stage
4. Read each remaining file and write/expand `annotations/[system].md` using the standard format below
5. Update each processed row in `_file_index.csv` to `stage=annotated`
6. Update `[system].md` stage header to `in-progress` or `complete` as appropriate
7. Commit on the `vanilla-annotation` branch: `annotate: [system name]`

### Annotation file format
```markdown
# [System Name]
**Stage:** [current stage]

## Overview
One paragraph: what this system does in-game and why modders care about it.

## Vanilla File Locations
Summary of where this system lives — folder path and what each key file covers.
For the full file list filtered by this system, see `_file_index.csv`.

## Block Structure
Syntax skeleton of a definition block with all major fields labeled.
Use inline comments (# ...) to explain each field.
Show one skeleton per block type if multiple types exist.

## Key Fields Reference
Bullet list or table: field name, type, what it controls, constraints.

## Modding Notes
- Safe fields to override vs. risky ones
- Load order gotchas (numbered prefixes like 00_)
- Cross-system dependencies
- Common patterns in vanilla

## Examples
1–2 representative vanilla blocks (trimmed if large) with commentary.
```

### Completeness criteria (`complete` stage)
- All major fields across all mapped files are documented
- At least one vanilla example included
- All source files marked `annotated` in `_file_index.csv`
- `GAME_STRUCTURE_GUIDE.md` entry links to `[system].md`

## Git & Branches
- `main` — stable scaffolding only
- `vanilla-annotation` — annotation work; PR to main when a system is fully documented
- `mod-dev` — active mod development

## Mod File Conventions
- Mod files mirror EU5 standard: `in_game/common/` and `main_menu/common/`
- Prefix mod-specific files with a short mod identifier (e.g. `BPW_country.txt`)
- In-file comments are encouraged for complex or non-obvious changes
- Commit frequently with descriptive messages
