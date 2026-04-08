# Production Methods System
**Stage:** complete
**Game version:** 1.1.10
**Keywords:** production_methods, building_maintenance, goods, input, output, category, no_upkeep, potential, allow

> **System type: Gameplay**

## Overview

Production methods (PMs) are the **goods consumption and production recipes** for buildings. They define what goods a building needs to operate (inputs) and optionally what it produces (outputs). A building can reference PMs in two ways:

- **Named PMs** — defined in `in_game/common/production_methods/` and referenced by key from building definitions using `possible_production_methods`.
- **Inline/unique PMs** — defined directly inside a building's `unique_production_methods` block (see [buildings.md](buildings.md)).

Most maintenance-only PMs live in the named PM files. Production-heavy and building-specific PMs are usually inlined.

---

## File Locations

| Path | Purpose |
|---|---|
| `in_game/common/production_methods/` | Named PM definitions |
| `in_game/common/production_methods/__readme.txt` | Format reference |
| `in_game/common/production_methods/unsorted_building_inputs.txt` | Miscellaneous named PMs |

---

## Block Structure

```
pm_key = {
    # --- Inputs (goods consumed per tick per building level) ---
    <good_key> = <float>    # e.g. tools = 0.2, lumber = 0.5

    # --- Output ---
    produced = <good_key>   # Optional. The good this PM produces.
    output   = <float>      # Amount of good produced per tick per level.

    # --- Category ---
    category = <pm_category>    # Required. Groups PMs in UI and logic.

    # --- Flags ---
    no_upkeep = yes/no          # If yes, blocks maintenance cost calculation for this PM.

    # --- Availability ---
    potential = { <triggers> }  # Root = country. Whether this PM is a valid option.
    allow     = { <triggers> }  # Root = country. Whether this PM can be selected.
}
```

---

## Key Fields Reference

| Field | Purpose | Key constraint |
|---|---|---|
| `<good_key>` (input goods) | Goods consumed per tick per building level to keep the building operating | Any valid good key; multiple entries allowed; amounts scale with building level and goods access when referenced via `possible_production_methods` |
| `produced` | The good this PM outputs | Optional; omit for maintenance-only PMs |
| `output` | Quantity of the produced good per tick per building level | Required when `produced` is set |
| `category` | Groups the PM in UI and logic | Required; must be a valid PM category key (e.g. `building_maintenance`, `guild_input`, `building_production`) |
| `no_upkeep` | Suppresses maintenance cost UI display; goods inputs in the PM are still consumed as normal | Default `no`; set `yes` for PMs that should appear "free" in the UI |
| `potential` | Trigger block controlling whether this PM appears as a valid option | Root = country owning the building |
| `allow` | Trigger block controlling whether this PM can be selected | Root = country owning the building; failing `allow` greys out the PM |

---

## PM Categories

`category` is a required tag. Common values:

| Category | Purpose |
|---|---|
| `building_maintenance` | Standard operating costs; no production output |
| `guild_input` | Guild buildings; declares a production good and input goods |
| `building_production` | Dedicated production PMs |

---

## Named vs Inline PMs

### Named PMs (`possible_production_methods`)
Defined in `production_methods/*.txt` and referenced by key:
```
# In building definition:
possible_production_methods = {
    bridge_maintenance
    pound_lock_canal_maintenance
}
```
The building presents these as selectable options.

### Inline PMs (`unique_production_methods`)
Defined directly in the building block. Inline PMs have the same syntax as named PMs but are only available to that building:
```
unique_production_methods = {
    lumber_mill_maintenance = {
        produced = lumber
        output = 1
        tools = 0.4
        category = building_maintenance
    }
}
```

---

## Example

### 1. Pure maintenance PM (named)
```
soldier_building_maintenance = {
    firearms = 0.5
    leather  = 0.5
    cloth    = 0.10
    paper    = 0.20
    weaponry = 0.5
    category = building_maintenance
}
```
No `produced` or `output` — this PM only consumes goods to keep the building running.

### 2. Production PM with conditional availability
```
hab_reformed_monastery_maintenance = {
    potential = {
        has_variable = hab_monastery_reform_accepted
    }
    glass    = 0.025
    paper    = 0.025
    cloth    = 0.025
    beer     = 0.15
    beeswax  = 0.05
    category = building_maintenance
}
```
`potential` gates the PM to countries with a specific variable set — used for reform-altered building variants.

### 3. PM with no_upkeep
```
lutheran_preacher_maintenance = {
    books    = 0.02
    no_upkeep = yes
    category = building_maintenance
}
```
`no_upkeep = yes` suppresses maintenance cost calculation. Used for PMs that are "free" to operate.

---

## Modding Notes

- Named PMs are referenced by key from buildings. The key must be unique across all PM files.
- For simple one-off building recipes, prefer inline `unique_production_methods` — keeps the PM co-located with the building.
- For PMs shared across multiple buildings, define them as named PMs.
- `potential` and `allow` are scoped to the **country** owning the building, not the location.
- Scaling: PM goods amounts are multiplied by building level and (for buildings) goods access — same as `modifier` scaling on buildings.
- PM goods amounts are scaled by building level and goods access — the same scaling as `modifier` on buildings — but only when referenced via `possible_production_methods`; check whether inline PMs in `unique_production_methods` scale identically.
