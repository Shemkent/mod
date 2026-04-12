# Diseases
**Stage:** complete
**Keywords:** disease, monthly_spawn_chance, spawn, r0, environmental_infection, calc_interval_days, location_stagnation_chance, mortality_rate, character_mortality_chance, location_modifier, monthly_resistance_reduction, on_spread_to_country, map_color

> **System type: Gameplay**

## Overview

Diseases model epidemic outbreaks that spread location-by-location using an SIR-like model. Each disease definition sets its spawn chance and location, a basic reproduction number (R0), environmental infection rates, mortality, and a location modifier applied proportionally to presence. Diseases interact with the situations system — the Black Death and Great Pestilence situations each fire when their linked disease becomes active. Vanilla ships 7 diseases: bubonic plague, great pestilence (Columbian Exchange), influenza, malaria (endemic, no spread), measles, smallpox, and typhus. (malaria is environmental-only with R0 = 0 — architecturally distinct from the others; see Modding Notes)

## Vanilla File Locations

| Path | Purpose |
|---|---|
| `in_game/common/diseases/readme.txt` | Full field reference with spread rules |
| `in_game/common/diseases/bubonic_plague.txt` | Bubonic plague (Black Death trigger) |
| `in_game/common/diseases/great_pestilence.txt` | Great Pestilence (Americas) |
| `in_game/common/diseases/influenza.txt` | Influenza (moderate R0, climate-sensitive) |
| `in_game/common/diseases/malaria.txt` | Malaria (endemic; environmental only, R0 = 0) |
| `in_game/common/diseases/measles.txt` | Measles (high R0, urban-amplified) |
| `in_game/common/diseases/smallpox.txt` | Smallpox (moderate-high R0) |
| `in_game/common/diseases/typhus.txt` | Typhus (siege-linked) |

Full file list in `_file_index.csv`.

## Block Structure

```
disease_key = {
    # --- Spawn control ---
    monthly_spawn_chance = <scripted_maths>    # 0..1; scope:disease = disease type.
    spawn = { <effects> }                      # Where to call spawn_disease { disease = scope:disease ... }.

    # --- Transmission model ---
    r0 = <scripted_maths>                      # Basic reproduction number.
                                               # Root = location, scope:disease = disease.
    environmental_infection = <scripted_maths> # Ongoing env. seeding (e.g. malaria).
                                               # Root = location, scope:disease/disease_outbreak/current_presence.

    # --- Spread timing ---
    calc_interval_days = <scripted_maths>      # Days between spread calculations. Lower = faster spread.

    # --- Stagnation ---
    location_stagnation_chance = <scripted_maths>   # Chance spread stagnates at a location (0..1).
    sub_unit_stagnation_chance = <scripted_maths>   # Stagnation chance for sub-units (0..1).

    # --- Mortality ---
    mortality_rate = <scripted_maths>          # Fraction of affected pops that die (0..1).
    character_mortality_chance = <scripted_maths>  # Chance a character dies per calc interval.
    percentage_to_meet_their_fate_on_calc = <scripted_maths>  # Fraction resolved (live/die) per tick.

    # --- Persistence ---
    monthly_resistance_reduction = <scripted_maths>  # How fast resistance leaks away per month.

    # --- Outbreak thresholds ---
    location_infection_spread_threshold = <scripted_maths>  # Min presence to start spreading.

    # --- Location modifier ---
    location_modifier = { <modifiers> }        # Applied to locations × presence percentage.

    # --- Per-pop-type weight ---
    specific_pop_type_effect = {
        pop_type   = <pop_type_key>
        multiplier = <float>        # 0 = immune, >1 = extra vulnerable.
    }

    # --- Country event on spread ---
    on_spread_to_country = { <effects> }       # Root = country, scope:disease = disease.

    # --- Map display ---
    map_color           = { <script_color_block> }   # Root = location, scope:disease.
    secondary_map_color = { <script_color_block> }
}
```

## Key Fields Reference

| Field | Purpose | Key constraint |
|---|---|---|
| `monthly_spawn_chance` | Probability the disease spawns a new outbreak each month | Script maths; 0 = dormant |
| `spawn` | Effect block that calls `spawn_disease { disease = scope:disease ... }` at a qualifying location; root = country/location depending on the iterator used inside the block | Must include `spawn_disease` |
| `r0` | Basic reproduction number (person-to-person) | Root = location; 0 for environmental-only diseases |
| `environmental_infection` | Background seeding from environment (malaria, etc.) | Root = location; accumulates regardless of R0 |
| `calc_interval_days` | Spread recalculation frequency | Lower number = more frequent ticks |
| `mortality_rate` | Fraction of affected pops that die | 0..1; scope includes presence |
| `character_mortality_chance` | Per-tick probability of character death | 0..1; root = location |
| `monthly_resistance_reduction` | How fast acquired immunity decays per month | Default 0 |
| `location_infection_spread_threshold` | Minimum presence required before disease leaves a location | 0..1 |
| `location_stagnation_chance` | Probability that disease spread stagnates at a location per calc interval, slowing spread significantly | 0..1; root = location |
| `sub_unit_stagnation_chance` | Same stagnation chance for military sub-units | 0..1; root = sub-unit |
| `percentage_to_meet_their_fate_on_calc` | Fraction of affected pops whose fate (survive and gain resistance, or die) is resolved each calc interval | 0..1 |
| `secondary_map_color` | Stripe/secondary color for the disease map overlay | Optional; same scope as `map_color` |
| `location_modifier` | Modifier block applied to affected locations | Scaled by presence percentage |
| `specific_pop_type_effect` | Per-pop-type vulnerability weight | `multiplier = 0` makes pop type immune |
| `on_spread_to_country` | Event fired when disease first enters a new country | Root = country |

## Modding Notes

- Diseases that should not spread person-to-person set `r0 = { value = 0 }` and rely on `environmental_infection` instead (malaria).
- The `spawn` block must include a `spawn_disease = { disease = scope:disease value = <float> }` call. Optionally `save_scope_as = outbreak` to reference the outbreak later.
- Use `situation_is_active` and `situation_has_ended` inside `monthly_spawn_chance` to prevent a disease from re-emerging after its linked situation has ended.
- Define a custom location modifier key `local_<disease_tag>_impact_modifier` that other files (buildings, on_actions) can apply as a negative value to reduce the disease's `location_modifier` impact. Vanilla uses this convention; other files (buildings, on_actions) apply the modifier by the expected key name, so if you deviate, those systems will not find it.
- Spread is blocked when the destination location already has ≥ 50% presence (hardcoded engine threshold, not controlled by `location_infection_spread_threshold`), when there is an embargo between owners, or when destination population is zero. `location_infection_spread_threshold` controls the *minimum* presence in the source location before it starts spreading outward — a distinct mechanic.
- Spread channels are: neighbouring locations, market centres, trade partner locations, and the location owner's capital.

## Example

Influenza (moderate respiratory disease):
```
influenza = {
    monthly_spawn_chance = { value = 0.25 }

    spawn = {
        random_ownable_location = {
            limit = {
                population > 1
                OR = { winter_level = normal  winter_level = severe }
                trigger_if = {
                    limit = { situation:great_pestilence = { situation_is_active = no  situation_has_ended = no } }
                    OR = { continent = continent:asia  continent = continent:europe  continent = continent:africa  continent = continent:oceania }
                }
            }
            spawn_disease = { disease = scope:disease  value = 0.5  save_scope_as = outbreak_scope }
        }
    }

    r0 = {
        value = { 1 2 }                 # base range
        if = {
            limit = { OR = { climate = continental  climate = cold_arid } }
            multiply = 1.2              # cold climates boost transmission
        }
        if = {
            limit = { has_building_with_at_least_one_level = marketplace }
            multiply = 1.2             # market density boosts spread
        }
    }

    mortality_rate = { value = 0.01 }
}
```
The `spawn` block filters to winter-affected populated locations; the `trigger_if` excludes the Americas until the Columbian Exchange situation has started (preventing anachronistic global pandemics). R0 is a range — the engine rolls a value each tick.
