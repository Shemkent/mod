# Map Geography (Climates, Topography, Vegetation)
**Stage:** complete
**Keywords:** `climates`, `topography`, `vegetation`, `location_modifier`, `movement_cost`, `winter`, `blocked_in_winter`, `defender`, `proximity`, `colonial_migration_size_modifier`

> **System type: Data/Reference**
> **Cluster:** Geography & Map

## Overview
Three parallel systems define the physical properties of map locations. **Climates** set the seasonal regime (winter severity, population capacity, life expectancy). **Topography** defines terrain shape (mountains, hills, flatland, plateau, wetlands, etc.) affecting movement, combat, and infrastructure costs. **Vegetation** defines ground cover (desert, sparse, grasslands, farmland, woods, forest, jungle) affecting movement, supply, population, and combat defence. Every location on the map has all three attributes simultaneously; their `location_modifier` blocks stack.

## Vanilla File Locations
- `in_game/common/climates/00_default.txt`
- `in_game/common/topography/00_default.txt`
- `in_game/common/vegetation/00_default.txt`

## Block Structure

### Climate
```pdx
tropical = {
    winter = none                         # none / mild / normal / severe / arctic
    # has_precipitation = no              # [optional] disables precipitation (arid/cold_arid climates)
    # always_winter = yes                 # [optional] visual-only; also requires topography always_winter

    color = climate_tropical              # named color for climate map mode

    location_modifier = {
        local_population_capacity_modifier = 0.50
        local_monthly_development_modifier = -0.10
        local_life_expectancy = -5
        free_capacity_attracts_pops = yes
        local_food_decay = 0.002
    }

    unit_modifier = { }                   # modifiers applied to units stationed here

    colonial_migration_size_modifier = -0.33  # scaling on colonist migration pool size

    audio_tags = { masDataClimateZone = 1.0 } # audio zone data

    debug_color = rgb { 14 58 9 }         # map screenshot debug overlay
}
```

### Topography
```pdx
mountains = {
    color = terrain_mountain

    movement_cost = 2                     # multiplier on unit movement speed (1.0 = normal)
    proximity = -0.5                      # flat modifier to effective capital distance
    defender = 2                          # combat defence bonus for defending unit

    vegetation_density = 0.2             # density of vegetation overlay in gfx
    blocked_in_winter = yes              # location impassable when winter penalty is active

    weather_front_strength_change_percent = 0
    weather_cyclone_strength_change_percent = 0
    weather_tornado_strength_change_percent = 0

    location_modifier = {
        local_frontage_allowed = -4
        local_rgo_build_time = 1.0
        local_road_building_time = 3.0
        local_population_capacity_modifier = -0.50
        local_monthly_development_modifier = -0.50
        local_monthly_food_modifier = -0.2
        blocks_vision_from_land = yes
        blocks_vision_from_sea = yes
    }

    colonial_migration_size_modifier = -0.2

    audio_tags = { masDataTopologyElevation = 1.0 }
    debug_color = rgb { 105 24 4 }
}
```

### Vegetation
```pdx
grasslands = {
    color = terrain_grasslands

    movement_cost = 1.0
    proximity = -0.05                    # slight distance penalty

    location_modifier = {
        local_population_capacity = 50
        local_monthly_food_modifier = 0.1
        supply_limit = 2
    }

    colonial_migration_size_modifier = -0.1   # minor penalty; open land is good but not tropical

    audio_tags = { masDataVegetationDensity = 0.2 }
    debug_color = rgb { 90 235 27 }
}
```

## Key Fields Reference

| Field | System | Purpose | Notes |
|---|---|---|---|
| `winter` | Climate | Winter severity | `none`/`mild`/`normal`/`severe`/`arctic` — affects army attrition and `blocked_in_winter` |
| `movement_cost` | Topo / Veg | Unit movement multiplier | 1.0 = base; >1 = slower; stacks multiplicatively between topo and veg |
| `proximity` | Topo / Veg / Road | Capital distance modifier | Flat; mountains/forests add distance, roads subtract it |
| `defender` | Topo / Veg | Combat defence bonus | Flat bonus for the defender; woods/forest/jungle all grant +1 |
| `blocked_in_winter` | Topography | Impassable in winter | Only matters if climate `winter` ≠ `none` |
| `vegetation_density` | Topography | Vegetation overlay density | 0.0–1.0; renderer-side only |
| `has_sand` | Veg / Topo | Desert flag | Present on `desert` vegetation and `dune_wasteland` topography; used by `is_desert` trigger |
| `has_precipitation` | Climate | Disables precipitation | Present on arid/cold_arid climates; omitting it enables default precipitation behaviour |
| `always_winter` | Climate | Perpetual winter visual | Visual-only; requires the location's topography to also have `always_winter` to take effect |
| `colonial_migration_size_modifier` | All three | Colonist pool size scaling | Negative = harder to colonize |
| `location_modifier` | All three | Gameplay modifiers to location | All three stacks are summed |
| `unit_modifier` | Climate | Modifiers to units stationed here | Combat, attrition, supply |
| `audio_tags` | All three | Audio zone data | Passed to audio middleware |
| `debug_color` | All three | Screenshot map overlay color | Development-only; no gameplay effect |

## Modding Notes
- Every location has exactly one climate, one topography, and one vegetation type. All three `location_modifier` blocks stack additively.
- `movement_cost` from topography and vegetation are multiplied together (not added) — mountains × forest = very slow.
- `blocked_in_winter` only activates when the location's climate has `winter` ≠ `none` and the in-game winter season is active.
- `proximity` is the same field as in road_types — roads effectively counteract the negative proximity of difficult terrain.
- `local_frontage_allowed` in topography directly limits how many units can engage simultaneously — the primary gameplay driver of mountain/pass combat.
- `colonial_migration_size_modifier` stacks across climate, topo, and vegetation — a tropical rainforest (all three negative) is very hard to colonize.
- Assignments of climate/topo/vegetation to specific locations are in binary map data, not in these files — remapping locations requires the map editor or binary data changes, not text edits here.
- `has_precipitation = no` on arid climates disables weather precipitation; a new arid climate type without it will receive precipitation by default.
- `always_winter = yes` on a climate is visual-only and requires the location's topography to also carry `always_winter` — setting it on climate alone has no effect.
- Cross-references: `road_types` (proximity interaction), `location_ranks` (build time modifiers), `goods` (food modifiers), `diseases` (climate interacts with disease spread).

## Example

A high-altitude jungle location would receive:
- Climate `tropical`: `local_population_capacity_modifier = 0.50`, `local_food_decay = 0.002`, `colonial_migration_size_modifier = -0.33`
- Topography `mountains`: `movement_cost = 2`, `local_frontage_allowed = -4`, `colonial_migration_size_modifier = -0.2`
- Vegetation `jungle`: `movement_cost = 1.5`, `defender = 1`, `proximity = -0.5`, further `colonial_migration_size_modifier`

All three stacks sum — the combined result is a dangerous, hard-to-colonize, defensible location.
