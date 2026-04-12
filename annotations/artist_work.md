# Artist Work
**Stage:** complete
**Keywords:** `work_type_id`, `captured`, `allow`, `location_modifier`, `country_modifier`, `religion_scale_modifier`

> **System type: UI/Presentation**

## Overview
Artist work types define what categories of cultural artifact (painting, symphony, palace, treatise, etc.) can be created by artists and what effects they produce. Each type specifies whether it can be looted by conquering armies (`captured`), which artist type(s) can create it (`allow`), and what modifiers it applies — to the owner country, to the location where it resides, or scaled to the whole religion. Cross-references `artist_types` via the `artist_type` trigger in `allow`.

## Vanilla File Locations

| File | Content |
|---|---|
| `in_game/common/artist_work/00_default.txt` | All ~20 work type definitions |
| `in_game/common/artist_work/readme.txt` | Field documentation |

## Block Structure
```
work_type_id = {
    captured = yes                   # Whether the work can be looted/captured (yes/no)

    allow = {                        # Character-scope trigger for creation eligibility
        artist_type = painter
        NOR = {                      # Example: age restriction
            current_age = age_5_absolutism
        }
    }

    country_modifier   = { ... }     # Modifier applied to the owning country
    location_modifier  = {           # Modifier applied to the work's location
        potential_trigger = { is_port = yes }   # Optional scoped condition
        local_monthly_development_modifier = 0.1
    }
    religion_scale_modifier = some_scripted_value   # Modifier scaled to whole religion
}
```

## Key Fields Reference

| Field | Purpose | Notes |
|---|---|---|
| `captured` | Lootable on conquest | `yes` = can be stolen; `no` = immovable (buildings, monuments) |
| `allow` | Creation filter | Character scope; typically `artist_type = X` ± age/condition checks |
| `country_modifier` | Owner-wide bonus | Standard modifier block |
| `location_modifier` | Location bonus | Can include `potential_trigger` to scope further |
| `religion_scale_modifier` | Religion-wide bonus | References a scripted value; rare — only used for `icon` type in vanilla |

## Modding Notes
- `captured = no` is appropriate for immovable works (buildings, monuments, oral traditions).
- Multiple `allow` conditions with `OR` let several artist types create the same work type.
- `allow = { always = no }` is used for `scripture` to prevent new instances from being created in-game while keeping the type defined for existing items.
- Age-locking in `allow` (e.g. motets only in early ages, symphonies only Age 5+) is the vanilla pattern for historically appropriate cultural output.
- `location_modifier` with a `potential_trigger` allows scoped conditional bonuses (e.g. only applying to ports).

## Example
```
icon = {
    captured = yes
    allow = {
        artist_type = iconographer
    }
    religion_scale_modifier = religious_icon_power_modifier
    location_modifier = {
        local_unrest              = -0.2
        local_max_rgo_size_modifier = 0.10
    }
}
```
*Icons are lootable, restricted to `iconographer` type, apply a location bonus at the site, and scale a modifier across the whole religion.*
