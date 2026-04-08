# Situations
**Stage:** complete
**Keywords:** situations, monthly_spawn_chance, can_start, can_end, visible, on_start, on_monthly, on_ending, on_ended, hint_tag, international_organization_type, resolution, map_color

> **System type: Gameplay**

## Overview

Situations are global, multi-country historical events — crises and turning points that unfold over months or years, visible to many nations simultaneously. Unlike disasters (single-country) or events (one-off), a situation persists with monthly callbacks, optional resolution votes, and a map overlay. Vanilla situations include the Black Death, Reformation, Hundred Years' War, Sengoku, and around 20 in total. Each situation defines its own spawn chance, start/end conditions, monthly effects, and per-country visibility logic.

## Vanilla File Locations

| Path | Purpose |
|---|---|
| `in_game/common/situations/readme.txt` | Field reference |
| `in_game/common/situations/black_death.txt` | Black Death (disease-linked) |
| `in_game/common/situations/columbian_exchange.txt` | Columbian Exchange |
| `in_game/common/situations/reformation.txt` | Protestant Reformation |
| `in_game/common/situations/hundred_years_war.txt` | Hundred Years' War |
| `in_game/common/situations/sengoku.txt` | Sengoku Jidai |
| `in_game/common/situations/[17 further files]` | Other world-historical crises |

Full file list in `_file_index.csv`.

## Block Structure

```
situation_key = {
    # --- Spawn control ---
    monthly_spawn_chance = <scripted_value|float>     # 0..1; scope:situation = situation type.
    hint_tag = <loc_key>                              # Optional. Advisor hint localisation key.

    # --- Lifecycle triggers ---
    can_start = { <triggers> }   # Root = situation.
    can_end   = { <triggers> }   # Root = situation.

    # --- Per-country visibility ---
    visible = { <triggers> }     # Root = country, scope:target = situation.

    # --- Effects ---
    on_start   = { <effects> }   # Root = situation; general set-up.
    on_monthly = { <effects> }   # Root = situation; fired every month.
    on_ending  = { <effects> }   # Root = situation; fires before status changes to ended.
    on_ended   = { <effects> }   # Root = situation; fires after status changes to ended.

    # --- Optional IO/resolution integration ---
    international_organization_type = <io_type_key>  # Associate with an IO.
    resolution = <resolution_key>                    # Link a resolution vote to the situation.
    voters     = <global_list_tag>                   # Global list of characters eligible to vote.

    # --- Map presentation ---
    is_data_map = yes/no                             # Whether to use a data-driven map color.
    map_color = { <script_color_block> }             # Color per location (root = location, scope:target = situation).
    secondary_map_color = { <script_color_block> }   # Stripe color (same scope).

    # --- Legend ---
    legend_key = {
        desc  = "<loc_string>"
        color = <color_definition>
    }

    tooltip = { <effects> }    # Used to generate map tooltip text only; not executed.
                               # Root = location, scope:target = situation.
}
```

## Key Fields Reference

| Field | Purpose | Key constraint |
|---|---|---|
| `monthly_spawn_chance` | Probability the situation spawns each month | Script value; 0 = never, 1 = guaranteed |
| `can_start` | Trigger block that must pass for the situation to begin | Root = situation |
| `can_end` | Trigger block that must pass for the situation to end | Root = situation |
| `visible` | Which countries see and can interact with the situation | Root = country |
| `on_start` | One-time setup effects (fire events, set variables) | Root = situation |
| `on_monthly` | Recurring monthly effects for the duration | Root = situation |
| `on_ending` / `on_ended` | Cleanup effects; `on_ending` fires before status changes | Root = situation |
| `international_organization_type` | Ties the situation to an IO (e.g. Papal IO for Reformation) | Must match an IO type key |
| `resolution` | Links a resolution vote to the situation's lifecycle | Must match a resolution key |
| `voters` | Global list tag of characters eligible to vote in the linked resolution | Used with `resolution` field; global list must be populated before vote resolves |
| `is_data_map` | Enables engine-driven per-location map coloring via `map_color` script block | Must be `yes` to use `map_color` and `secondary_map_color` |
| `legend_key` | Defines the map legend entry shown while the situation map is active; `desc` = loc string, `color` = color definition | Only used when `is_data_map = yes` |
| `secondary_map_color` | Stripe/secondary color for the map overlay; same scope as `map_color` | Optional; used for gradient/transition effects alongside `map_color` |
| `hint_tag` | Advisor hint shown when situation is active | Localisation key |
| `tooltip` | Effect block evaluated for map tooltip text only | Root = location |

## Modding Notes

- Only one situation of each key can be active at a time. Use `situation_is_active` and `situation_has_ended` in `monthly_spawn_chance` to prevent respawning.
- `can_start` / `can_end` are scoped to `root = situation`, not country — use `any_country` iterators to test world state.
- `on_monthly` fires for all countries matching the internal scope; use `every_country` with `limit` to target specific participants.
- `visible` determines UI access per country — set it broadly if all nations can observe the situation.
- The `tooltip` block is never executed; it is only parsed for tooltip display. Safe to reference complex triggers without side effects.
- Situations interact with diseases via `disease_is_active`, `disease_outbreak_is_active`, and the `diseases/` system — vanilla's Black Death situation links to `disease:bubonic_plague`.
- `on_monthly` scope is `root = situation` — to affect countries, use `every_country = { limit = {...} ... }` iterators inside the effect block.
- Map coloring requires `is_data_map = yes` plus a `map_color` script block; `lerp` can gradient-color locations by disease presence or other numeric values.

## Example

The Columbian Exchange (abbreviated):
```
columbian_exchange = {
    monthly_spawn_chance = monthly_spawn_chance_unique
    hint_tag = hint_columbian_exchange

    can_start = {
        current_age = age_5_absolutism
        continent:america = {
            any_location_in_continent = {
                population > 0
                owner_from_old_world = yes
            }
        }
    }

    can_end = { columbian_exchange_end_trigger = yes }

    visible = {
        any_market_present_in_country = {
            save_temporary_scope_as = temp_market
            any_in_global_list = {
                variable = new_world_goods
                save_temporary_scope_as = temp_goods
                scope:temp_market = {
                    OR = {
                        is_produced_in_market = scope:temp_goods
                        is_traded_in_market = scope:temp_goods
                    }
                }
            }
        }
    }

    on_start = { ... }       # Set variables, fire events
    on_monthly = { ... }     # Drive ongoing trade/crop spread events
    on_ended = { ... }       # Clean up variables and fire conclusion events
}
```
`monthly_spawn_chance_unique` is a script value that returns 0 once the situation has already fired, ensuring single-occurrence situations don't repeat.
