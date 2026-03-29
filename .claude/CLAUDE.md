# EU5 Modding Project Guidelines

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

## How to Use .metadata/ Reference Files

When working on a mod, **always read `.metadata/README.md` first**. It contains:
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
- When a system is not yet annotated, read the vanilla files and summarize into `annotations/[system].md`

## Git & Branches
- `main` — stable scaffolding only
- `vanilla-annotation` — annotation work; PR to main when a system is fully documented
- `mod-dev` — active mod development

## Mod File Conventions
- Mod files mirror EU5 standard: `in_game/common/` and `main_menu/common/`
- Prefix mod-specific files with a short mod identifier (e.g. `BPW_country.txt`)
- In-file comments are encouraged for complex or non-obvious changes
- Commit frequently with descriptive messages
