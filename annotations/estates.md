# Estates System
**Stage:** complete
**Keywords:** `estates`, `estate_privileges`, `power_per_pop`, `tax_per_pop`, `satisfaction`, `high_power`, `low_power`, `privilege`, `can_revoke`

> **System type: Gameplay**

## Overview

Estates are political factions within a country (nobles, clergy, burghers, peasants, etc.). Each estate has:
- **Power** — determined by pop counts and privileges; drives scaled modifiers
- **Satisfaction** — drives approval-based modifiers
- **Opinion** — multi-factor diplomatic opinion calculation (used for foreign estate relations)

**Estate privileges** are specific grants to an estate that apply country/province/location modifiers while active. A country can hold many privileges at once; each privilege belongs to one estate.

## Vanilla File Locations

| Path | Purpose |
|---|---|
| `in_game/common/estates/00_default.txt` | All estate type definitions |
| `in_game/common/estate_privileges/` | All privilege definitions, split by estate |
| `in_game/common/estate_privileges/readme.txt` | Privilege attribute reference |

Full list in `_file_index.csv`.

## Block Structure

### Estate definition
```
estate_key = {
    color = <color_key>                     # Map/UI color.

    power_per_pop = <int>                   # Power gained per 1000 pops of this estate type.
    tax_per_pop   = <int>                   # Tax income per 1000 pops.

    rival    = <float>                      # Opinion modifier per estate rival.
    alliance = <float>                      # Opinion modifier per estate ally.

    revolt_court_language = <language_key>  # Language used in revolts from this estate.

    characters_have_dynasty = always|sometimes|never
    can_spawn_random_characters = yes/no
    can_generate_mercenary_leaders = yes/no
    bank = yes/no                           # Can this estate loan money?

    ruler = yes                             # Crown estate only — marks this as the ruler estate.
    priority_for_dynasty_head = yes         # Prioritize when calculating dynasty head.

    satisfaction = { <modifiers> }          # Scaled by (satisfaction - LOW_SATISFACTION_THRESHOLD).
    high_power = { <modifiers> }            # Scaled by (power - LOW_POWER_THRESHOLD) when above.
    low_power = { <modifiers> }             # Scaled by -(power - LOW_POWER_THRESHOLD) when below.
    power = { <modifiers> }                 # Always active, scaled by power level.

    opinion = {
        add = { desc = <loc_key>  value = <scripted_maths> }
        # ... multiple add blocks
    }
}
```

### Estate privilege
```
privilege_key = {
    estate = <estate_key>           # Required. Which estate this privilege belongs to.

    potential  = { <triggers> }     # Root = country. Whether the privilege is visible.
    allow      = { <triggers> }     # Root = country. Whether the privilege can be enacted.
    can_revoke = { <triggers> }     # Root = country. Whether the privilege can be revoked.

    years   = <int>
    months  = <int>
    weeks   = <int>
    days    = <int>

    on_activate        = { <effects> }
    on_fully_activated = { <effects> }
    on_deactivate      = { <effects> }

    country_modifier  = { <modifiers> }
    province_modifier = { <modifiers> }
    location_modifier = { <modifiers> }
}
```

## Key Fields Reference

| Field | Purpose | Key constraint |
|---|---|---|
| `power_per_pop` / `tax_per_pop` | Power and tax per 1000 pops | Integer |
| `satisfaction` / `high_power` / `low_power` | Scaled modifier blocks | Multiplied by distance from threshold |
| `ruler` | Marks the crown estate | Only one estate should have this |
| `estate` (privilege) | Binds privilege to an estate | Must match an estate key |
| `potential` / `allow` / `can_revoke` | Gating triggers for privileges | Root scope = country |
| `country_modifier` / `province_modifier` / `location_modifier` | Modifier blocks on privileges | Supports scaled+triggered form (see laws.md) |
| `on_activate` / `on_fully_activated` / `on_deactivate` | Lifecycle effects | Fired when privilege state changes |

## Modding Notes

- New estates: add a block to `estates/00_default.txt`. Requires corresponding pop type, localization, and icons.
- New privileges: add to the appropriate estate's file in `estate_privileges/`, or create a new file.
- Privileges are referenced as `estate_privilege:<privilege_key>` in triggers (e.g. `has_estate_privilege = estate_privilege:nobles_land_rights`).
- `country_modifier` on privileges supports the scaled+triggered form (see laws.md / government_reforms.md for syntax).
- Enacting a privilege usually grants satisfaction to the estate — set this via `nobles_estate_target_satisfaction` or equivalent modifier keys.
- `can_revoke = {}` — empty trigger means always revocable.

## Example

Gated privilege with revoke condition:
```
auxilium_et_consilium = {
    estate = nobles_estate
    potential = { government_type = government_type:monarchy }
    allow = {
        has_advance = noble_knights
        NOT = { has_estate_privilege = estate_privilege:monetary_fiefs }
    }
    can_revoke = {}
    country_modifier = {
        global_nobles_estate_power = 0.5
        levy_combat_efficiency_modifier = 0.10
        nobles_estate_levy_size = 0.20
    }
}
```
`potential` restricts to monarchies; `allow` gates behind an advance and mutual exclusion with another privilege; empty `can_revoke` means always revocable.
