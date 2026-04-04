# Scripted Lists
**Stage:** complete
**Keywords:** `scripted_list_key`, `base`, `conditions`, `every_`, `any_`, `random_`, `ordered_`

> **System type: Scripted Logic**

## Overview
Scripted lists are named, filtered views of the game's built-in scope iteration lists. Define one entry block and the engine automatically registers four iteration variants: `every_<key>`, `any_<key>`, `random_<key>`, and `ordered_<key>`. The filtering `conditions` block works identically to a trigger block, and `PREV` resolves to the calling scope, enabling context-sensitive lists. Modders use this system to avoid repeating the same `limit = { ... }` block across many effects and triggers.

## Vanilla File Locations

| File | Content |
|---|---|
| `in_game/common/scripted_lists/character_lists.txt` | Character-scope lists: courtier, general, admiral, explorer |
| `in_game/common/scripted_lists/country_lists.txt` | Country-scope lists: junior_partner, american_colonial_country, cardinal_country; commented example shows `known_country` base (ally) |
| `in_game/common/scripted_lists/religion_lists.txt` | Religion-scope lists: protestant_religion |
| `main_menu/common/scripted_lists/scripted_lists.info` | Info file only — no main-menu definitions in v1.1.10 |

## Block Structure

```pdxscript
list_key = {
    base = <scriptlist_type>     # required — e.g. character, country, religion, union_partner
    conditions = {               # optional trigger block; filters elements included in the list
        <triggers>
        # PREV = calling scope
    }
}
```

Once defined, all four variants become available in script:

```pdxscript
every_list_key = { <effects> }
any_list_key = { <triggers> }
random_list_key = { <effects> }
ordered_list_key = { order_by = <script_value>  <effects> }  # order_by required; sorts ascending by value before iterating
```

## Key Fields Reference

| Field | Purpose | Notes |
|---|---|---|
| `base` | Underlying EU5 script list type | Must be a valid list root (e.g. `character`, `country`, `religion`, `union_partner`, `known_country`, `country_in_religion`); the four prefix variants derive from it |
| `conditions` | Trigger block filtering which elements are included | Same syntax as any trigger block; `PREV` resolves to the calling scope |

## Modding Notes
- All `.txt` files in `in_game/common/scripted_lists/` are loaded; file names are arbitrary.
- `base` must exactly match a valid EU5 script list root — using an unknown type causes a load error.
- `PREV` in `conditions` refers to the scope that called `every_list_key = {}`, enabling lists like "all allies of the current country" without hardcoding the scope.
- The conditions act as an implicit `limit` — they do not fire at definition time, only at iteration time.
- Scripted lists cannot be parameterized (no `$variable$` substitution); use parameterized `scripted_triggers` inside the `conditions` block for dynamic filtering.
- Duplicate `list_key` names across files: later-loaded files overwrite earlier definitions; there is no merge. Keep list keys unique across your mod.
- No scripted list definitions exist in `main_menu/common/scripted_lists/` in v1.1.10.

## Example

```pdxscript
# in_game/common/scripted_lists/character_lists.txt
courtier = {
    base = character
    conditions = {
        is_courtier = yes
        is_ruler = no
        is_heir = no
        is_regent = no
    }
}
# Usage in effects:
every_courtier = { add_prestige = 5 }
random_courtier = { add_trait = lucky }
```

```pdxscript
# Context-sensitive list using PREV
# PREV used as a block key here saves the calling scope as a named alias before
# the conditions run — this is the only way to reference the caller inside conditions.
junior_partner = {
    base = union_partner
    conditions = {
        PREV = { save_temporary_scope_as = senior_partner }  # alias the caller
        union ?= {
            country_has_special_status = {
                type = special_status:senior_partner
                country = scope:senior_partner
            }
            NOT = {
                country_has_special_status = {
                    type = special_status:senior_partner
                    country = PREV
                }
            }
        }
    }
}
```
