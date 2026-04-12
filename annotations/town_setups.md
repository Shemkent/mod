# Town Setups
**Stage:** complete
**Keywords:** `town_setups`

> **System type: Data/Reference**
> **Cluster:** Geography & Map

## Overview
Town setups are named presets that define the initial building configuration for newly generated locations. Each setup is a flat list of `building_type = level` pairs. They are referenced from country or region setup files to give historically appropriate starting infrastructure to locations when a scenario begins. A Scandinavian coastal town might start with a brewery and marketplace; a Russian city might start with granaries and a castle.

## Vanilla File Locations
`in_game/common/town_setups/` — 1 file:
- `00_default.txt` — all vanilla town setup presets, organised by region

## Block Structure

```pdx
my_town_setup = {
    brewery = 1
    temple = 1
    tools_guild = 1
    weapon_guild = 1
    marketplace = 1
    mason = 1
}
```

Each key is a building type ID; the value is the initial building level. Only buildings that exist in `common/building_types/` are valid.

## Key Fields Reference

| Field | Purpose | Notes |
|---|---|---|
| `<building_id> = <level>` | Starting building at this level | Must be a valid building type |

## Modding Notes
- Town setup IDs are referenced from location/province definition files (typically under `in_game/setup/`), not from within `common/`.
- Building levels are capped by the location rank and any building cap modifiers — setting a level higher than the cap will be silently clamped.
- Setups are applied once at game start; they do not enforce a minimum or prevent the player from demolishing them.
- Adding new buildings to vanilla setups is safe provided the building type exists.
- Cross-references: `buildings` (building type IDs), `location_ranks` (determines which buildings are available).

## Example

```pdx
important_scandinavian_town = {
    brewery = 1
    temple = 1
    tools_guild = 1
    tannery = 1
    naval_supplies_guild = 1
    weapon_guild = 1
    marketplace = 2
    mason = 1
    pottery_guild = 1
}
```
*A major Nordic coastal town starts with level-2 marketplace and naval infrastructure, reflecting its role as a trade hub.*
