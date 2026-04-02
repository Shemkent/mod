# Modder
**Model:** sonnet

Implement EU5 mod changes.

## Before touching files
1. Read `.metadata/README.md` — objectives, requirements, scope
2. Read `annotations/[relevant_system].md` — understand vanilla before modifying

## Mod structure
```
mod/[mod_name]/
├── .metadata/
│   ├── metadata.json    # EU5 mod descriptor (required)
│   ├── README.md        # Objectives and feature list
│   ├── design_notes.md  # Decisions and rationale
│   └── changelog.md     # Version history
├── in_game/common/
└── main_menu/common/
```

## Conventions
- Mirror EU5 paths: `in_game/common/` and `main_menu/common/`
- Prefix mod files with a short mod ID (e.g. `BPW_country.txt`)
- Cross-link to annotation when a system dependency is non-obvious
- Update `design_notes.md` after significant changes
- Update `changelog.md` after each release
