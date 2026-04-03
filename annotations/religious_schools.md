# Religious Schools, Figures & Factions
**Stage:** complete
**Keywords:** religious_school, religious_figure, religious_faction, enabled_for_country, enabled_for_character, enabled_for_religion, visible, enabled, actions, color, modifier

> **System type: Gameplay**

## Overview
Three small religion sub-systems provide religion-specific character roles, philosophical traditions, and court factions:

- **Religious schools** (`religious_schools/`) — philosophical/theological traditions that grant country and character modifiers; selectable by countries whose religion matches.
- **Religious figures** (`religious_figures/`) — named character roles (e.g., Muslim scholar, Hindu guru) enabled for specific religion groups; purely defines which character types are available to which faiths.
- **Religious factions** (`religious_factions/`) — court factions or power blocs specific to a religion (e.g., Shinto's imperial court, shogunate); define visibility, enablement, and available actions.

## Vanilla File Locations
- `in_game/common/religious_schools/` — 6 `.txt` files by religion
- `in_game/common/religious_figures/` — 2 `.txt` files
- `in_game/common/religious_factions/` — 2 `.txt` files (+ readme)
- Full file list in `_file_index.csv`.

## Block Structures

### Religious School
```pdx
<school_id> = {
    color = rgb { <R> <G> <B> }    # UI display color

    enabled_for_country = {
        # Trigger; root = country; typically: religion = religion:<id>
    }
    enabled_for_character = {
        # Trigger; root = character; typically: religion = religion:<id>
    }

    modifier = {
        # Applied to the country/character while this school is active
        tolerance_heathen  = <float>
        tolerance_heretic  = <float>
        global_max_literacy = <int>
        global_population_growth = <float>
        # etc.
    }
}
```

### Religious Figure
```pdx
<figure_id> = {
    enabled_for_religion = {
        group    = <religion_group_id>    # enabled for an entire group
        religion = <religion_id>          # OR enabled for a specific religion
    }
}
```

### Religious Faction
```pdx
<faction_id> = {
    visible = {
        # Trigger; root = country (or IO context)
        # Controls whether this faction appears in the UI
    }
    enabled = {
        # Trigger; controls whether the faction is playable/active
    }
    actions = {
        <action_id>    # generic actions available to this faction
    }
}
```

## Key Fields Reference
| Field | Purpose | Key constraint |
|---|---|---|
| `enabled_for_country` (school) | Country eligibility trigger | Root = country; usually a religion check |
| `enabled_for_character` (school) | Character eligibility trigger | Root = character; allows different filtering for character-assigned schools |
| `modifier` (school) | Modifiers while school is active | Applied to the country and/or character depending on context |
| `color` (school) | UI color for the school | RGB values; no game-mechanical effect |
| `enabled_for_religion` (figure) | Which religions/groups have this character role | Can use `group` or `religion`; both can appear in the same block |
| `visible` (faction) | Faction UI visibility | Trigger on country context; faction hidden if false |
| `enabled` (faction) | Faction gameplay availability | Can be visible but disabled (greyed out) |
| `actions` (faction) | Generic actions available through the faction | References generic action IDs |

## Modding Notes
- **Religious schools** are the most modding-relevant of the three. Adding a new school requires: a definition in any `.txt` file in `religious_schools/`, `enabled_for_country` and `enabled_for_character` triggers, a `modifier` block, and localization. The selection mechanism (how countries choose a school) lives in laws or generic actions.
- **Religious figures** are purely enabling definitions — they declare which character role types are available to which faiths. The actual character creation, hiring, and interaction logic lives elsewhere (character interactions, generic actions). Adding a new figure type requires localization and entries in `religious_figures/`.
- **Religious factions** are religion-specific extensions of the general faction system. They appear in the court UI for countries whose religion has factions listed in its `factions {}` block. The `visible`/`enabled` triggers typically check government type, IO membership, or religion-specific variables.
- **Cross-system:** religious schools interact with the modifier system (stacking with `definition_modifier`, aspects, and holy sites). Religious factions interact with the international organizations system (Shinto factions reference the shogunate IO) and generic actions. Religious figures interact with the character system.

## Example
`samkhya_school` — a Hindu philosophical school granting tolerance:

```pdx
samkhya_school = {
    color = rgb { 54 13 0 }

    enabled_for_country = {
        religion = religion:hindu
    }
    enabled_for_character = {
        religion = religion:hindu
    }

    modifier = {
        tolerance_heathen = 0.5
        tolerance_heretic = 0.5
    }
}
```

Only available to Hindu countries and characters. The school grants tolerance in both directions, reflecting the Samkhya school's non-dogmatic philosophical tradition. The `color` is purely for UI display.
