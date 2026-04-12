# Institutions
**Stage:** complete
**Keywords:** institution, age, can_spawn, promote_chance, spread, embrace, spread_from_friendly_coast_border_location, spread_from_any_coast_border_location, spread_from_any_import

> **System type: Gameplay**

## Overview

Institutions are age-linked systems (Feudalism, Meritocracy, Printing Press, etc.) that spread organically between locations through trade, borders, and control. A location that meets the spawn conditions gains a promote chance each tick; once enough locations in a country have embraced the institution, the country itself embraces it (the embrace threshold is engine-determined and not exposed as a scriptable field in v1.1.10) and gains modifiers. Each institution is tied to an age and has a suite of spread rate fields that the engine evaluates each calculation tick.

## Vanilla File Locations

| Path | Purpose |
|---|---|
| `in_game/common/institution/readme.txt` | Field reference |
| `in_game/common/institution/age_1_traditions_institutions.txt` | Age 1 (Feudalism, Legalism, Meritocracy, …) |
| `in_game/common/institution/age_2_renaissance_institutions.txt` | Age 2 (Renaissance, …) |
| `in_game/common/institution/age_3_discovery_institutions.txt` | Age 3 (Exploration, Global Trade, …) |
| `in_game/common/institution/age_4_reformation_institutions.txt` | Age 4 (Printing Press, Reformation, …) |
| `in_game/common/institution/age_5_absolutism_institutions.txt` | Age 5 (Absolutism, …) |
| `in_game/common/institution/age_6_revolutions_institutions.txt` | Age 6 (Enlightenment, Industrial Revolution, …) |

Full file list in `_file_index.csv`.

## Block Structure

```
institution_key = {
    age = <age_key>                # The age in which this institution becomes relevant.

    # --- Spawn condition ---
    can_spawn = { <triggers> }     # Root = location. Which locations can initially generate it.

    # --- Spawn probability ---
    promote_chance = <scripted_maths>   # Root = location. Score added each tick toward spawning.

    # --- Spread rates (script values; root = location) ---
    spread_from_friendly_coast_border_location = <scripted_value>
    spread_from_any_coast_border_location      = <scripted_value>
    spread_from_any_import                     = <scripted_value>
    spread_from_any_export                     = <scripted_value>
    spread_scale_on_control_if_owner_embraced  = <scripted_value>  # multiplier when owner has embraced
    spread_embraced_to_capital                 = <scripted_value>
    spread_to_market_member                    = <scripted_value>
    spread                                     = <scripted_value>  # Generic fallback spread speed.
}
```

## Key Fields Reference

| Field | Purpose | Key constraint |
|---|---|---|
| `age` | Which age this institution belongs to | Must match an age key |
| `can_spawn` | Location trigger; only qualifying locations can originate the institution | Root = location |
| `promote_chance` | Script maths block summing scores from pop types, literacy, etc. | Higher = spawns sooner |
| `spread_from_friendly_coast_border_location` | Spread to border/coast locations owned by friendly countries | Script value |
| `spread_from_any_coast_border_location` | Spread to any adjacent coastal/border location | Script value |
| `spread_from_any_import` / `spread_from_any_export` | Spread via trade connections | Script value |
| `spread_scale_on_control_if_owner_embraced` | Multiplier on spread when the controlling country has embraced | Multiplier (e.g. 2) |
| `spread_embraced_to_capital` | Direct spread push to owner's capital if they've embraced | Script value |
| `spread_to_market_member` | Spread through market membership connections | Script value |
| `spread` | Fallback generic spread speed | Script value |

## Modding Notes

- One institution definition per file is the convention, but multiple per file work fine.
- All spread values reference named script values (e.g. `institution_base_spread_from_friendly_neighbor_with_early`) from `script_values/`. Reuse these to keep consistent spread rates across your institutions.
- `can_spawn` is the initial seeding condition — it does not control ongoing spread. Once spawned in any qualifying location, spread fields drive propagation.
- `promote_chance` accumulates additively from pop types, literacy, and owner state; the engine picks locations each tick and accumulates promote score — when the score crosses an internal threshold, the institution spawns at that location.
- Institutions that have already spawned globally do not respawn elsewhere — the seeding is one-time.
- No `on_embrace` effect field exists at the institution level in v1.1.10; embrace effects are handled in `on_action/` hooks.
- Cross-system: embrace effects (on_embrace callbacks) are handled in `on_action/` hooks, not in the institution definition itself — see `on_action.md`.

## Example

The `feudalism` institution (Age 1), abbreviated:
```
feudalism = {
    age = age_1_traditions

    can_spawn = {
        continent = continent:europe
        has_owner = yes
        num_pop_type:nobles > 0.1
    }

    promote_chance = {
        add = {
            value    = num_pop_type:nobles
            multiply = 10
        }
    }

    spread_from_friendly_coast_border_location = {
        value = institution_base_spread_from_friendly_neighbor_with_early
        add   = { value = num_pop_type:nobles  multiply = 0.5 }
    }
    spread_from_any_coast_border_location = institution_base_spread_from_neighbor_with_early
    spread_from_any_import                = institution_trade_spread_value_early
    spread_from_any_export                = institution_trade_spread_value_early
    spread_scale_on_control_if_owner_embraced = 2
    spread_embraced_to_capital            = institution_total_embraced_to_capital_early
    spread_to_market_member               = institution_spread_to_market_member_early
}
```
Noble population density drives both seeding (`can_spawn` threshold) and propagation speed (`promote_chance` weight). The `spread_scale_on_control_if_owner_embraced = 2` doubles spread speed once the owner has embraced, accelerating diffusion across their territory.
