# Gods
**Stage:** complete
**Keywords:** god, icon, group, religion, potential, allow, on_activate, on_fully_activated, on_deactivate, country_modifier, province_modifier, location_modifier, add_god, remove_god

> **System type: Gameplay**

## Overview
Gods are religion-specific deities that countries can adopt, granting modifiers and triggering effects during adoption and deactivation. Each god definition associates with one or more religion groups (via `group` blocks) or individual religions, defines eligibility triggers, and provides country/location modifiers with optional time-gated activation. Vanilla ships `gods/common.txt` for the Abrahamic god shared by Christians, Jews, and Muslims, plus separate files for folk religions (`folk_african.txt`, `folk_american.txt`, `folk_asian.txt`, etc.). Gods are added/removed via `add_god`/`remove_god` effects.

## Vanilla File Locations
- `in_game/common/gods/` — 11 `.txt` files (includes readme + folk religion variants)
- Full file list in `_file_index.csv`.

## Block Structure
```pdx
<god_id> = {
    icon = <icon_key>    # UI icon for this deity

    # --- Religion association (repeatable; group and religion blocks share the same structure) ---
    group = {
        group    = <religion_group_id>
        name_key = <localization_key>   # localized name for this religion's version of the god
    }
    religion = {
        religion = <religion_id>
        name_key = <localization_key>   # localized name for this religion's version of the god
    }

    # --- Availability ---
    potential = { <trigger> }   # root = country; controls UI visibility
    allow     = { <trigger> }   # root = country; controls whether adoption is possible

    # --- Lifecycle effects ---
    on_activate        = { <effect> }          # fired when adoption begins
    on_fully_activated = { <effect> }          # fired when time-gated activation completes
    on_deactivate      = { <effect> }          # fired when the god is removed

    # --- Modifiers ---
    country_modifier = {
        scale = { <script value> }            # optional scaling factor
        potential_trigger = { <trigger> }     # condition for modifier to apply
        <modifier_key> = <value>
    }
    province_modifier = { <modifier_key> = <value> }    # applied to specific provinces
    location_modifier = { <modifier_key> = <value> }    # applied to locations

    # --- Time-gated activation ---
    years  = <int>
    months = <int>
    weeks  = <int>
    days   = <int>
}
```

## Key Fields Reference
| Field | Purpose | Key constraint |
|---|---|---|
| `group` (repeatable) | Associates god with a religion group + localized name | Multiple `group` blocks allow one god to be shared across faiths (e.g. the Abrahamic god) |
| `religion` | Associates god with a single religion | Block structure identical to `group` (uses `religion` + `name_key` sub-fields); can be combined with `group` blocks |
| `potential` | UI visibility trigger | Root = country |
| `allow` | Adoption eligibility trigger | Root = country; shown but disabled if fails |
| `on_activate` | Effect when adoption begins | Fires immediately when `add_god` is called |
| `on_fully_activated` | Effect when adoption reaches 100% | Fires immediately if no time delay is set; fires after the delay period if `years`/`months`/`weeks`/`days` is configured |
| `on_deactivate` | Effect when god is removed | Fires when `remove_god` is called |
| `country_modifier` | Country-level modifiers while god is active | Supports `scale` (script value multiplier) and `potential_trigger` (conditional application) |
| `years`/`months`/`weeks`/`days` | Activation delay | Delays `on_fully_activated`; without a time delay, `on_fully_activated` fires immediately on adoption |

## Modding Notes
- **Gods are added/removed via effects:** `add_god = <god_id>` and `remove_god = <god_id>`. These are called from events, laws, or generic actions.
- **A god can be shared across multiple religion groups** using multiple `group` blocks with different `name_key` values. The Abrahamic god in vanilla (`god = {}` in `common.txt`) is shared between Christian, Jewish, and Muslim groups with different localized names.
- **`country_modifier` supports `scale` and `potential_trigger`** for conditional or scaled modifier application. The `scale` field takes a script value; the modifier's numeric values are multiplied by the scale.
- **Time-gated activation** (`years`/`months`/`weeks`/`days`) delays `on_fully_activated`. Use this for gradual conversion or adoption mechanics where immediate full effect is not desired.
- **Cross-system:** gods interact with the holy sites system (`god` field on holy site definitions), the religion system (religion/group association), and the modifier system (country/location modifiers).

## Example
`god` (abbreviated) — the Abrahamic deity shared by three groups:

```pdx
god = {
    icon = _default
    group = {
        group    = christian
        name_key = CHRISTIAN_GOD
    }
    group = {
        group    = israelite_group
        name_key = JUDAISM_GOD
    }
    group = {
        group    = muslim
        name_key = MUSLIM_GOD
    }

    country_modifier = { }    # empty in vanilla — no mechanical bonus
}
```

This single definition covers three religion groups with different localized names. All Christian, Jewish, and Muslim countries share the same god entry with culturally appropriate names. The empty `country_modifier` is valid — the god exists for flavor and holy site association without granting modifiers.
