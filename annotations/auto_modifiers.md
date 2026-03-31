# Auto Modifiers
**Stage:** complete
**Keywords:** `modifier_id`, `category`, `type`, `icon`, `requires_real`, `potential_trigger`, `limit`, `scales_with`, `hide_effects`, `alert`

## Overview
Auto modifiers are persistent modifiers applied automatically to a scoped entity (country, location, dynasty) whenever their conditions are met — no scripted effect required. They are the primary mechanism for translating game state (stability, war exhaustion, legitimacy, population ratios, karma, etc.) into ongoing mechanical effects. `country.txt` alone defines 70+ auto modifiers that collectively form the baseline rules of the game. Modders use this system to add continuous effects that respond dynamically to a country's state without triggering from events.

## Vanilla File Locations

| File | Content |
|---|---|
| `in_game/common/auto_modifiers/country.txt` | ~70+ country-scope auto modifiers (base values, government power, war state, population ratios, etc.) |
| `in_game/common/auto_modifiers/dynasty.txt` | Dynasty-scope auto modifiers |
| `in_game/common/auto_modifiers/international_organization.txt` | IO-scope auto modifiers |
| `in_game/common/auto_modifiers/location.txt` | Location-scope auto modifiers (empty in 1.1.10) |
| `in_game/common/auto_modifiers/readme.txt` | Field documentation |

## Block Structure
```
modifier_id = {
    category  = country          # Modifier category (default: country); must be first if set
    type      = country          # Scope type for potential_trigger (default: country); must be before potential_trigger

    icon          = "path/to/icon"   # Optional icon path
    requires_real = no               # yes (default) = rebels/pirates/mercenaries excluded
    hide_effects  = yes              # Optional: hide modifier from tooltips
    alert         = yes              # Optional: show in alerts panel when active

    potential_trigger = { ... }      # Outer condition: modifier is only evaluated if true
    limit         = { ... }          # Inner condition: modifier only applies if true (evaluated after potential_trigger)

    scales_with = {                  # Optional: multiply all effects by a computed value
        value    = stability
        multiply = 0.01
        min      = 0                 # Optional clamps
    }
    # OR: scales_with = some_scripted_value   # Shorthand for simple values

    [modifier keys]                  # The actual effects applied
}
```

## Key Fields Reference

| Field | Purpose | Notes |
|---|---|---|
| `potential_trigger` | Outer gate | If false, modifier is skipped entirely (performance opt) |
| `limit` | Inner gate | Modifier applies only when true; evaluated inside potential_trigger |
| `scales_with` | Multiplier | Supports full scripted math (`value`, `multiply`, `add`, `subtract`, `min`, `max`, `abs`) or shorthand `scales_with = scripted_value` |
| `requires_real = no` | Include non-real countries | Real = player/AI countries; non-real = rebels, pirates, mercenaries |
| `category` / `type` | Scope declaration | Must appear before `potential_trigger`/modifier keys if used |
| `hide_effects` | Tooltip suppression | Hides modifier from UI but still applies effects |
| `alert` | Alert panel visibility | Shows modifier in the alerts bar when active |

## Modding Notes
- **`country_base_values`** is the most important auto modifier: it sets the universal baseline for dozens of game values (tolerance, morale recovery, decay rates, migration rules, default ranges, etc.). Overriding it carelessly can break core mechanics.
- `potential_trigger` / `limit` split is a performance pattern: `potential_trigger` avoids evaluating `limit` when the outer condition is false.
- Signed `scales_with` math: to scale with a value where higher = worse (e.g. negative legitimacy), use `multiply = -0.01` and optionally `add = 1` to invert direction.
- The `abs = 0` pattern (with a dummy RHS) triggers absolute-value evaluation — used for balanced/neutral-state modifiers (e.g. `balanced_harmony`).
- `location.txt` is empty in 1.1.10 — the system supports location scope but vanilla hasn't used it yet.

## Example
```
war_exhaustion_impact = {
    scales_with = war_exhaustion

    land_morale_modifier  = -0.025
    naval_morale_modifier = -0.025
    global_production_efficiency = scaled_production_efficiency_penalty
    trade_efficiency             = scaled_trade_efficiency_penalty
    global_population_growth     = -0.0005
}

high_legitimacy = {
    potential_trigger = {
        uses_government_power = legitimacy
    }
    limit = {
        legitimacy > 50
    }
    scales_with = {
        value    = legitimacy
        multiply = 0.01
    }
    global_crown_estate_power  = 0.10
    pop_leave_rebels_threshold = -0.05
    monthly_righteousness      = 0.5
}
```
*`war_exhaustion_impact`: simple scale with no trigger — always active, scales linearly.*
*`high_legitimacy`: gated by government type then by value threshold; scales with legitimacy above 50.*
