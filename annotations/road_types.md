# Road Types
**Stage:** complete
**Keywords:** `road_types`, `level`, `movement_cost`, `market_access`, `proximity`, `build_time_per_unit_distance`, `price_per_unit_distance`

> **System type: Gameplay**
> **Cluster:** Geography & Map

## Overview
Road types define the infrastructure tiers that countries can construct between locations. Each tier improves unit movement speed and market access at increasing cost and build time; higher tiers also reduce effective capital distance via `proximity` (absent on the lowest tier). Roads are built along edges between locations; the cost and time scale with the geographic distance traversed. Four vanilla tiers progress from gravel tracks to railroads. Mods can add intermediate tiers or replace existing ones.

## Vanilla File Locations
`in_game/common/road_types/` — 2 files:
- `readme.txt` — field reference
- `00_generic.txt` — all four vanilla road types

## Block Structure

```pdx
my_road = {
    level = 2                          # integer; higher = better/later road; controls has_latest_road_to triggers

    movement_cost = -0.3               # modifier to unit movement speed (negative = faster)
    market_access = -0.2               # modifier to market access distance penalty (negative = better access)
    proximity = 5                      # reduction to effective distance-to-capital for connected locations

    build_time_per_unit_distance = 20  # months to build per unit of geographic distance
    price_per_unit_distance = build_paved_road          # references common/prices/
    construction_demand = build_paved_road_demand       # references common/goods_demand/
    maintenance_demand = maintain_paved_road_demand     # [optional] ongoing goods drain

    color = map_paved_road             # named color for the road map mode overlay
    spline_style_id = 1                # visual spline style index (renderer asset reference)

    enabled = {                        # [optional] trigger; root = country attempting to build
        has_technology = railroad_technology
    }
}
```

## Key Fields Reference

| Field | Purpose | Notes |
|---|---|---|
| `level` | Upgrade tier integer | Used by `has_latest_road_to` and upgrade logic; must be unique |
| `movement_cost` | Speed modifier on locations with this road | Negative offset (e.g. `-0.3`); combined with topography's positive multiplier |
| `market_access` | Market distance penalty modifier | Negative = better connectivity |
| `proximity` | Capital distance reduction | Flat subtraction; absent on `gravel_road` (lowest tier only) |
| `color` | Named color for the road map mode overlay | Must match a defined named color (e.g. `map_paved_road`) |
| `build_time_per_unit_distance` | Months per geographic unit | Scales with map distance |
| `price_per_unit_distance` | Price reference per unit distance | See `common/prices/` |
| `construction_demand` | Goods consumed during construction | See `common/goods_demand/` |
| `maintenance_demand` | Ongoing goods drain | Present on all four vanilla types; omitting may default to no maintenance but is untested |
| `enabled` | Build condition trigger | Root = acting country |
| `spline_style_id` | Renderer asset index | Must match a defined spline style in gfx |

## Modding Notes
- `level` values must be a strict ordering — `has_latest_road_to` compares levels numerically.
- `movement_cost` on roads is a negative offset; topography uses positive multipliers (e.g. `mountains = 2`). The two combine additively — a road does not negate terrain cost entirely.
- `proximity` is a flat subtraction from effective capital distance; high values can make distant provinces act as if adjacent to the capital.
- `maintenance_demand` links to `common/goods_demand/` entries — missing entries will log an error.
- `enabled` blocks can gate high tiers behind technology or modifiers, matching historical unlock curves.
- Cross-references: `goods` (prices), `goods_demand`, `topography` (movement cost interaction).

## Example

```pdx
railroad = {
    level = 4
    movement_cost = -0.8
    market_access = -0.4
    proximity = 15
    build_time_per_unit_distance = 50
    price_per_unit_distance = build_railroad
    construction_demand = build_railroad_demand
    maintenance_demand = maintain_railroad_demand
    color = map_railroad
    spline_style_id = 3
}
```
*The highest road tier: massive movement bonus, strong market integration, and significant capital proximity gain — but expensive and slow to build.*
