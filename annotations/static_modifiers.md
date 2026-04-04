# Static Modifiers
**Stage:** complete
**Keywords:** `static_modifier_key`, `game_data`, `category`, `decaying`, `remove_if`, `<modifier_fields>`

> **System type: Gameplay**

## Overview
Static modifiers are named modifier blocks applied to scoped entities — countries, locations, characters, units, and more — by the game engine at hardcoded moments (bankruptcy, war, subject status, etc.) or by scripted effects. Unlike auto modifiers (which activate automatically when conditions are met), static modifiers are pushed onto an entity explicitly and persist until removed by effect or until their `remove_if` condition triggers. The `game_data` block declares the entity category and optional decay/removal behaviour. In v1.1.10, all vanilla static modifier definitions live in `main_menu/common/static_modifiers/`; the `in_game/common/static_modifiers/` folder is empty.

## Vanilla File Locations

| File | Content |
|---|---|
| `main_menu/common/static_modifiers/country.txt` | Country-scope static modifiers (is_bankrupt, is_subject, threatening_enemies, etc.) |
| `main_menu/common/static_modifiers/location.txt` | Location-scope modifiers |
| `main_menu/common/static_modifiers/character.txt` | Character-scope modifiers |
| `main_menu/common/static_modifiers/unit.txt` | Unit-scope modifiers |
| `main_menu/common/static_modifiers/religion.txt` | Religion-scope modifiers |
| `main_menu/common/static_modifiers/estates.txt` | Estate-scope modifiers |
| `main_menu/common/static_modifiers/institutions.txt` | Institution modifiers |
| `main_menu/common/static_modifiers/international_organization.txt` | IO-scope modifiers |
| `main_menu/common/static_modifiers/mercenary.txt` | Mercenary-scope modifiers |
| `main_menu/common/static_modifiers/province.txt` | Province-scope modifiers |
| `main_menu/common/static_modifiers/rebel.txt` | Rebel-scope modifiers |
| `main_menu/common/static_modifiers/societal_values.txt` | Societal value modifiers |
| `main_menu/common/static_modifiers/subunit.txt` | Subunit modifiers |
| … *(17 files total)* | One file per entity category |
| `in_game/common/static_modifiers/` | **Empty** in v1.1.10 |

## Block Structure

```pdxscript
modifier_key = {
    game_data = {
        category  = country          # entity type: country, location, character, unit, religion,
                                     # mercenary, province, internationalorganization, rebel, dynasty
        decaying  = no               # optional; yes = modifier weakens over time
        remove_if = none             # optional: none (default), religion, culture
                                     # religion → auto-removed if entity's religion changes
                                     # culture  → auto-removed if entity's culture changes
    }

    # Standard modifier fields for the declared category:
    total_loan_capacity_modifier = -0.5
    land_morale_modifier         = -0.9
    research_speed_modifier      = -0.9
    # ... any valid modifier key for the category
}
```

## Key Fields Reference

| Field | Purpose | Notes |
|---|---|---|
| `category` | Entity type this modifier targets | Must match the scope type the modifier is applied to |
| `decaying` | Modifier weakens over time | Default `no`; exact decay rate is engine-side |
| `remove_if` | Auto-removal trigger | `religion` or `culture` change removes the modifier |

## Modding Notes
- **Separation from auto_modifiers**: auto modifiers activate whenever their `limit` is true and deactivate when false; static modifiers must be explicitly added via effect (`add_modifier = { modifier = key }`) and removed via effect or `remove_if`.
- **Hardcoded application**: many static modifiers (e.g. `is_bankrupt`) are applied directly by engine code — the script definition only sets what the modifier *does*, not when it applies.
- **File location quirk**: unlike every other common/ system, vanilla static modifier definitions are in `main_menu/common/static_modifiers/`, not `in_game/`. Mod files should go in `in_game/common/static_modifiers/` (the engine loads both paths).
- **No merge on duplicate keys**: last-loaded file wins per modifier key.
- **`decaying` modifiers**: the strength of the modifier decreases each month; useful for temporary recovery penalties that fade naturally.

## Example

```pdxscript
# main_menu/common/static_modifiers/country.txt
is_bankrupt = {
    game_data = {
        category = country
    }
    total_loan_capacity_modifier = -0.5
    land_morale_modifier         = -0.9
    research_speed_modifier      = -0.9
    great_power_score            = -100
}

# Applied by engine code; can also be applied in script:
# add_modifier = { modifier = is_bankrupt  years = 5 }
```
