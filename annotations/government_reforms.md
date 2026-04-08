# Government Reforms System
**Stage:** complete
**Game version:** 1.1.10
**Keywords:** government_reforms, major, age, country_modifier, on_activate, potential, allow

> **System type: Gameplay**

## Overview

Government reforms are optional modifiers and mechanics that a country can adopt on top of its base government type. Unlike laws (which have mutually exclusive policies), a country can hold **many reforms simultaneously**. Reforms are separated by government type and age, with some being cross-type (`common.txt`).

Each reform applies `country_modifier`, `province_modifier`, and/or `location_modifier` while active, plus optional `on_activate` / `on_deactivate` effects.

---

## File Locations

| Path | Purpose |
|---|---|
| `in_game/common/government_reforms/` | All reform definitions |
| `in_game/common/government_reforms/readme.txt` | Attribute reference |

| File | Contents |
|---|---|
| `monarchy.txt` | Monarchy-specific reforms |
| `republic.txt` | Republic-specific reforms |
| `theocracy.txt` | Theocracy-specific reforms |
| `steppe_horde.txt` | Steppe horde reforms |
| `common.txt` | Cross-government-type reforms |
| `country_specific.txt` | Tag-gated unique reforms |
| `00_government_reforms.info` | Info file (metadata, not parsed as reforms) |

---

## Block Structure

```
reform_key = {
    # --- Scope ---
    government = <government_type_key>  # Optional. Restricts to a specific gov type.
    age        = <age_key>              # Optional. Reform becomes available from this age onward.

    # --- Exclusivity ---
    major      = yes/no                 # Major reforms are mutually exclusive (only one per country).
    unique     = yes/no                 # Adds extra UI fluff for notable reforms.
    block_for_rebel = yes/no            # Cannot be adopted by rebel governments.

    # --- Naming ---
    male_regnal_names   = { <names> }   # Optional names for male rulers under this reform.
    female_regnal_names = { <names> }   # Optional names for female rulers.

    # --- Availability ---
    potential = { <triggers> }          # Root = country. Whether the reform is visible.
    allow     = { <triggers> }          # Root = country. Whether the reform can be adopted.
    locked    = { <triggers> }          # Root = country. Whether the reform is currently locked.

    # --- Timing ---
    years   = <int>
    months  = <int>
    weeks   = <int>
    days    = <int>

    # --- Effects ---
    on_activate        = { <effects> }  # Root = country. Fired when reform is adopted.
    on_fully_activated = { <effects> }  # Root = country. Fired at 100% implementation.
    on_deactivate      = { <effects> }  # Root = country. Fired when reform is removed.

    # --- Modifiers ---
    country_modifier  = { <modifiers> }
    province_modifier = { <modifiers> }
    location_modifier = { <modifiers> }

    # --- Societal values ---
    societal_values = { <value_keys> }  # Reform counts toward unlocking these societal values.
}
```

---

## Key Fields Reference

| Field | Purpose | Key constraint |
|---|---|---|
| `government` | Restricts the reform to a specific government type. | Optional; omit for cross-type reforms. |
| `age` | Reform becomes available from this age onward. | Optional; omit = available from game start. |
| `major` | Marks the reform as mutually exclusive — only one `major = yes` reform can be active per country. | All `major` reforms compete globally across all files. |
| `unique` | Adds extra UI emphasis for notable reforms. | No gameplay effect beyond presentation. |
| `potential` | Trigger block (root = country) controlling whether the reform is visible in the UI. | — |
| `allow` | Trigger block (root = country) controlling whether the reform can be adopted. | — |
| `locked` | Prevents the player changing this reform while true; does not remove it. | Trigger block (root = country). |
| `years` / `months` / `weeks` / `days` | Implementation time — how long until `on_fully_activated` fires and modifiers reach 100%. | Use one field; they are additive if multiple are set. |
| `on_activate` | Effects block (root = country) fired when the reform is first adopted. | — |
| `on_fully_activated` | Effects block (root = country) fired when implementation reaches 100%. | — |
| `on_deactivate` | Effects block (root = country) fired when the reform is removed. | — |
| `country_modifier` / `province_modifier` / `location_modifier` | Modifiers applied to the country, its provinces, or specific locations while the reform is active. | Support scaled+triggered form; scale from 0 to full during implementation. |
| `societal_values` | Lists societal value keys this reform counts toward unlocking. | Links to the societal values system. |

---

## Key Mechanics

### Major reforms
Only one `major = yes` reform can be active at a time. They represent fundamental government structures (e.g. `admiralty_regime_reform`, `judicate_reform`). Adopting a new major reform removes the previous one.

### Age gating
`age = age_2_renaissance` means the reform can only be adopted once that age has begun. No age field = available from the start.

### Government scoping
`government = monarchy` means the reform only appears for monarchies. Common reforms in `common.txt` omit this field and apply to all government types.

### Modifier scaling
`country_modifier` supports the scaled+triggered form:
```
country_modifier = {
    scale = <scripted_maths>
    potential_trigger = { <triggers> }
    <modifier_key> = <value>
}
```

---

## Examples

### 1. Standard monarchy reform
```
autocracy = {
    government = monarchy
    country_modifier = {
        global_crown_estate_power = 0.1
        monthly_towards_aristocracy = societal_value_monthly_move
        monthly_towards_absolutism = societal_value_monthly_move
    }
    years = 2
}
```

### 2. Age-gated with potential gate
```
feudal_nobility = {
    age = age_1_traditions
    government = monarchy
    potential = {
        NOT = { has_reform = government_reform:french_feudal_nobility }
        NOT = { has_reform = government_reform:french_appanage_reform }
    }
    country_modifier = {
        subject_loyalty = 5
        nobles_estate_target_satisfaction = medium_permanent_target_satisfaction
        monthly_towards_traditional_economy = societal_value_monthly_move
    }
    years = 2
}
```
The `potential` block prevents adoption if mutually exclusive reforms are already active.

### 3. Major cross-type reform with on_activate
```
admiralty_regime_reform = {
    age = age_3_discovery
    major = yes
    potential = {
        num_of_ports > 0
        NOT = { government_type = government_type:steppe_horde }
        NOT = { government_type = government_type:tribe }
    }
    on_activate = {
        custom_tooltip = admiralty_regime_reform_tt
    }
    country_modifier = {
        monthly_towards_outward   = societal_value_monthly_move
        naval_force_limit_modifier = 0.25
    }
}
```

---

## Modding Notes

- Add new reforms to the appropriate file, or create a new file — all `.txt` files in the folder are loaded.
- Use `potential` to gate on tags, existing reforms, or game state. Use `allow` for "enabled but not visible" checks.
- To make a reform country-specific, gate with `potential = { has_or_had_tag = XXX }` or use `country_specific.txt`.
- `major = yes` reforms compete with each other across all files — be careful not to introduce unintended exclusion.
- `societal_values` links reforms to the societal values unlock system.
- **Laws** hold one active **policy** at a time (mutually exclusive choices within a slot). **Reforms** can all be active simultaneously (additive stacking), but `major = yes` reforms are mutually exclusive among themselves. Both use the same `country_modifier` / `on_activate` structure.
