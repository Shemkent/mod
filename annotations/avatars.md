# Avatars
**Stage:** complete
**Keywords:** `avatar_id`, `god`, `potential`, `allow`, `years`/`months`/`weeks`/`days`, `on_activate`, `on_fully_activated`, `on_deactivate`, `country_modifier`, `province_modifier`, `location_modifier`

## Overview
Avatars are divine manifestations in the Hindu religion system, representing different forms of a god (e.g. the ten avatars of Vishnu, the forms of Shakti). A country with Hinduism can adopt an avatar via scripted effect (`add_avatar`) to receive ongoing modifiers and trigger effects. Each avatar belongs to a god and optionally has adoption conditions, an activation delay, and modifiers at country/province/location scope. Currently only `hindu.txt` is defined in vanilla.

## Vanilla File Locations

| File | Content |
|---|---|
| `in_game/common/avatars/hindu.txt` | 20 avatar definitions for Vishnu and Shakti |
| `in_game/common/avatars/readme.txt` | Field documentation |

## Block Structure
```
avatar_id = {
    god = vishnu_god               # The god this avatar belongs to (required)

    potential = { ... }            # Country-scope; avatar appears as an option only if true
    allow    = { ... }             # Country-scope; can be adopted only if true

    years  = 5                     # Activation duration (all optional; instant if omitted)
    months = 0
    weeks  = 0
    days   = 0

    on_activate        = { ... }   # Effect fired when avatar is chosen (root = country)
    on_fully_activated = { ... }   # Effect fired when activation period completes
    on_deactivate      = { ... }   # Effect fired when avatar is removed

    country_modifier  = { ... }    # Modifier applied to the adopting country
    province_modifier = { ... }    # Modifier applied to provinces
    location_modifier = {          # Modifier applied to locations; supports potential_trigger
        potential_trigger = { is_port = yes }
        local_monthly_development_modifier = 0.1
    }
}
```

## Key Fields Reference

| Field | Purpose | Notes |
|---|---|---|
| `god` | Parent god | Required; groups avatars for UI and game logic |
| `potential` / `allow` | Visibility / availability | Both country-scope; `potential` hides, `allow` blocks |
| Duration fields | Activation ramp time | Modifiers scale with completion progress; instant if omitted |
| `on_*` lifecycle effects | Scripted events at state transitions | All optional; root = country |
| `country_modifier`, `province_modifier`, `location_modifier` | Ongoing effects | `location_modifier` supports `potential_trigger` for conditional application |

## Modding Notes
- Applied via scripted effects: `add_avatar = avatar_id` / `remove_avatar = avatar_id`.
- `location_modifier` supports a nested `potential_trigger` to conditionally apply to only certain locations (e.g. only ports).
- Vanilla bodies can be empty (`country_modifier = {}`); the engine requires the block but it can be inert.
- New gods must be defined elsewhere (religion system) before avatars can reference them.
- Only Hindu countries are expected to use this system in vanilla; modders adding religions with similar avatar mechanics should mirror this pattern.

## Example
```
matsya_avatar = {
    god = vishnu_god
    allow = {
        has_ports = yes
    }
    country_modifier = {}
    location_modifier = {
        potential_trigger = { is_port = yes }
        local_monthly_development_modifier = 0.1
    }
}

kali_avatar = {
    god = shakti_god
    country_modifier = {
        army_tradition_from_battle = 2
    }
}
```
*`matsya` (fish avatar) requires ports and buffs port locations; `kali` is unconditional with a combat tradition bonus.*
