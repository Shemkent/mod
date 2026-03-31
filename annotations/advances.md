# Advances
**Stage:** complete
**Keywords:** `advance_id`, `age`, `potential`, `allow`, `requires`, `government`, `country_type`, `for`, `unlock_*`, `allow_children`, `modifier_while_progressing`

## Overview
Advances are the research tree of EU5 — discrete nodes a country researches to unlock content and gain modifiers. Each advance belongs to an age and optionally restricts to a government type, country type, or specialization (`for`). The `unlock_*` family of keys is the primary mechanism for gating buildings, levies, laws, production methods, CBs, reforms, and more behind research. Files are organized by the scope that gates them: country tag, culture, culture group, government type, religion, estate, geographic area, or country type.

## Vanilla File Locations
`in_game/common/advances/` — 100+ files. Naming conventions:

| Pattern | Scope | Example |
|---|---|---|
| `country_XXX.txt` | Country tag | `country_FRA.txt` |
| `culture_[name].txt` | Culture | `culture_arabian.txt` |
| `culture_group_[name].txt` | Culture group | `culture_group_chinese.txt` |
| `government_[type].txt` | Government type | `government_monarchy.txt` |
| `ctype_[type].txt` | Country type | `ctype_army.txt` |
| `religion_[name].txt` | Religion | `religion_protestant.txt` |
| `estate_[name].txt` | Estate | `estate_cossacks.txt` |
| `area_[name].txt`, `region_[name].txt` | Geography | `area_tuscany.txt` |
| `3_[thing].txt` | Thematic unlocks | `3_reform_unlocks.txt` |
| `_advances_template.txt` | Dev template (all commented) | — |

## Block Structure
```
advance_id = {
    age         = age_3_discovery      # Age this advance appears in (required)
    potential   = { has_or_had_tag = FRA }  # Visibility condition (omit = always visible)
    allow       = { ... }              # Research condition (omit = always researchable)
    requires    = some_other_advance   # Prerequisite; determines tree position
    government  = monarchy             # Restrict to a government type (omit = any)
    country_type = army                # Restrict to a country type (omit = any)
    for         = mil                  # Restrict to adm/dip/mil specialization (omit = any)

    # Unlock keys (any number, all optional):
    # unlock_unit, unlock_ability, unlock_interaction, unlock_country_interaction,
    # unlock_relation_type, unlock_building, unlock_law, unlock_levy,
    # unlock_government_reform, unlock_casus_belli, unlock_subject_type,
    # unlock_production_method, unlock_heir_selection

    allow_children = no                # Force leaf node; logs error if violated

    [modifier keys]                    # Permanent modifiers granted on research

    modifier_while_progressing = {     # Temporary modifier while being researched
        potential_trigger = { ... }
        scale = { ... }
        [modifier keys]
    }
}
```

## Key Fields Reference

| Field | Purpose | Notes |
|---|---|---|
| `age` | Which age tree this advance belongs to | Required; must match a defined age ID |
| `potential` / `allow` | Visibility vs. availability | `potential` hides; `allow` shows but blocks — both use country scope |
| `requires` | Prerequisite advance ID | Can appear multiple times for multiple prereqs |
| `government`, `country_type`, `for` | Gating filters | All optional; combine freely |
| `unlock_*` | Content unlock | Primary modding target; many variants |
| `allow_children = no` | Forces leaf node | Useful for mutually exclusive branches |
| `modifier_while_progressing` | Scaled in-progress buff | `scale` uses scripted value math |

## Modding Notes
- **File placement:** add a new file with a clear name prefix. Load order is alphabetical within the folder.
- **`requires` chains:** the engine places advances in the tree based on `requires`. An advance with no `requires` floats to the root; multiple `requires` creates a merge node.
- **`potential` is not retroactive:** advances that become visible mid-run are not unlocked retroactively (noted in readme).
- **`country_type` values:** `army`, `location`, `building`, `pop` — these are the four non-standard country types. Vanilla uses these to gate advances exclusive to specialized play modes.
- **`unlock_*` vs. modifier keys:** `unlock_*` is one-time gating of content; modifier keys give permanent stat bonuses once researched.

## Example

```
noble_knights = {
    age      = age_1_traditions
    government = monarchy
    requires = feudalism_advance
    unlock_levy = levy_mailed_knights
}

aristocracy = {
    age      = age_2_renaissance
    government = monarchy
    requires = recruitment_improvements_renaissance
    unlock_levy = levy_plated_knights
    allow = {
        any_market_present_in_country = {
            OR = {
                is_produced_in_market = goods:tools
                is_traded_in_market   = goods:tools
            }
        }
    }
}
```
*Pattern:* monarchy-gated chain that unlocks levy types across ages; `allow` adds a trade-goods precondition for the tier-2 advance.
