# Disasters System
**Stage:** complete
**Game version:** 1.1.10
**Keywords:** disasters, monthly_spawn_chance, can_start, can_end, on_start, on_end, on_monthly, modifier, fire_only_once

> **System type: Gameplay**

## Overview

Disasters are country-level crisis events that apply ongoing penalties and fire monthly events while active. Each disaster defines when it can start, when it ends, what effects fire at the start/end, and what modifiers it applies during the crisis. Only one disaster can be active at a time per country (`has_any_active_disaster`).

---

## Vanilla File Locations

| Path | Purpose |
|---|---|
| `in_game/common/disasters/` | All disaster definitions (one per file) |
| `in_game/common/disasters/readme.txt` | Attribute reference |

---

## Block Structure

```
disaster_key = {
    image = "gfx/.../disaster_image.dds"   # Optional. UI illustration path.
    map_mode = <map_mode_tag>               # Optional. Map mode shown during the disaster.
    fire_only_once = yes/no                 # Default: no. If yes, can only occur once per country.

    monthly_spawn_chance = <scripted_value|float>  # Monthly probability to start (0..1).
                                                    # Root = country, scope:disaster = disaster type.

    can_start  = { <triggers> }  # Root = country, scope:disaster = disaster type.
    can_end    = { <triggers> }  # Root = country, scope:disaster = disaster type.

    modifier   = { <modifiers> }  # Applied to the country while disaster is ongoing.

    on_start   = { <effects> }   # Root = country, scope:disaster = disaster type.
    on_monthly = { <effects> }   # Root = country, scope:disaster = disaster type.
    on_end     = { <effects> }   # Root = country, scope:disaster = disaster type.
}
```

---

## Key Fields Reference

| Field | Purpose | Key constraint |
|---|---|---|
| `monthly_spawn_chance` | Monthly probability (0..1) that the disaster starts; evaluated each month against the country | References a scripted value from `script_values/`; root = country, `scope:disaster` = disaster type |
| `can_start` | Trigger block that must pass for the disaster to begin | Should include `has_any_active_disaster = no` to prevent stacking |
| `can_end` | Trigger block evaluated each month; when it passes the disaster ends | Typically a named scripted trigger from `scripted_triggers/` for reuse in events/decisions |
| `modifier` | Modifier block applied to the country for the entire duration of the disaster | Removed automatically when the disaster ends; no manual cleanup needed |
| `on_start` | Effect block fired once when the disaster begins | Root = country, `scope:disaster` = disaster type |
| `on_monthly` | Effect block fired every month while the disaster is active | Keep lightweight — see Modding Notes for performance implications |
| `on_end` | Effect block fired once when the disaster ends | Root = country, `scope:disaster` = disaster type |
| `fire_only_once` | Prevents the disaster from occurring more than once per country | Default `no`; set `yes` for historical one-time crises |
| `image` | Path to the UI illustration displayed during the disaster | Optional; `.dds` format |
| `map_mode` | Map mode tag activated while the disaster is active | Optional |

## Mechanics

### Spawn chance
`monthly_spawn_chance` is evaluated each month. It references a scripted value (from `script_values/`) returning a float 0..1. Vanilla uses named values like `monthly_spawn_chance_very_low`, `monthly_spawn_chance_medium`.

### One at a time
The `can_start` block should include `has_any_active_disaster = no` to prevent stacking. Disasters are mutually exclusive by convention (not enforced by the engine).

### fire_only_once
Historical disasters that can only happen once (e.g. specific succession crises) use `fire_only_once = yes`.

### Lifecycle
```
monthly_spawn_chance passes → on_start fires → modifier applied → on_monthly each month → can_end passes → on_end fires → modifier removed
```

---

## Example

### 1. Generic succession crisis
```
succession_crisis = {
    image = "gfx/interface/illustrations/disaster/succession_crisis.dds"
    monthly_spawn_chance = monthly_spawn_chance_very_low
    can_start = {
        has_any_active_disaster = no
        government_type = government_type:monarchy
        stability < 10
        legitimacy < 50
        in_civil_war = no
    }
    can_end = {
        succession_crisis_end_trigger = yes
    }
    modifier = {
        monthly_pretender_rebel_growth = 0.01
        global_crown_estate_power = -0.2
        blocked_from_creating_subjects = yes
    }
    on_start = {
        trigger_event_non_silently = succession_crisis.1
    }
    on_monthly = {
        random_list = {
            14 = { custom_tooltip = an_event_occurs_tt }
            86 = { custom_tooltip = no_event_occurs_tt }
        }
    }
    on_end = {
        add_legitimacy = legitimacy_extreme_bonus
        trigger_event_non_silently = succession_crisis.100
    }
}
```

### 2. Estate-driven disaster
```
peasants_war = {
    monthly_spawn_chance = monthly_spawn_chance_medium
    can_start = {
        has_any_active_disaster = no
        stability < 0
        at_war = no
        societal_value:serfdom_vs_free_subjects < 50
        estate_satisfaction:peasants_estate < 0.5
        OR = {
            government_type = government_type:theocracy
            government_type = government_type:monarchy
        }
    }
    can_end = { peasants_war_end_trigger = yes }
    modifier = {
        global_peasants_assimilation_blocked = yes
        peasants_estate_target_satisfaction = big_permanent_target_satisfaction_penalty
    }
    on_start  = { ... }
    on_end    = { ... }
}
```

---

## Modding Notes

- One file per disaster is the vanilla convention, but multiple disasters per file work fine.
- Always include `has_any_active_disaster = no` in `can_start` unless intentionally allowing overlap.
- `on_monthly` fires every month for all active disasters simultaneously — keep it lightweight; use `random_list` to probabilistically dispatch events rather than running effects unconditionally each tick.
- End triggers (`can_end`) are typically defined as named scripted triggers in `scripted_triggers/` for reuse in events/decisions.
- `modifier` is removed automatically when the disaster ends.
