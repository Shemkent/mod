# Location Ranks
**Stage:** complete
**Keywords:** `location_ranks`, `rank_modifier`, `country_modifier`, `allow`, `max_rank`, `build_time`, `is_established_city`

> **System type: Gameplay**
> **Cluster:** Geography & Map

## Overview
Location ranks define the settlement upgrade tiers — `rural_settlement` (base), `town`, and `city`. Each tier applies a permanent `rank_modifier` to the location and a `country_modifier` per location at that rank owned. Upgrades are gated by an `allow` trigger (typically population) and require a construction project. The base rank (`rural_settlement`) has no `allow` block — it is the starting state, not built to. Only one rank carries `max_rank = yes`; locations cannot be upgraded beyond it.

## Vanilla File Locations
`in_game/common/location_ranks/` — 1 file:
- `00_default.txt` — `city`, `town`, `rural_settlement` (in that order — higher ranks first)

## Block Structure

```pdx
city = {
    rank_modifier = {                     # permanent modifiers applied to the location at this rank
        local_burghers_desired_pop = 1.0
        local_laborers_desired_pop = 1.0
        local_soldiers_desired_pop = 0.25
        local_nobles_desired_pop = 0.05
        local_clergy_desired_pop = 0.1
        local_max_control = 0.1
        local_population_capacity = 100
        local_population_capacity_modifier = 0.25
        local_food_capacity = 500
        local_construction_speed = 0.25
        local_integration_speed_modifier = -0.5
        local_pop_promotion_speed_modifier = 0.25
        free_building_levels = 100
        local_peasant_enfranchisment = 0.25
        max_constructions_at_same_time = 1
    }

    country_modifier = {                  # modifiers applied to owning country per location at this rank
        monthly_doom = 0.03
        fort_limit = 1
    }

    allow = {                             # upgrade eligibility trigger; root = location
        location_rank = location_rank:town  # must already be at the previous rank
        population >= 30
        modifier:cannot_upgrade_location = no
    }

    max_rank = yes                        # this rank cannot be upgraded further
    is_established_city = yes             # marks as a full urban centre (used by triggers/effects)

    build_time = 730                      # days to complete the upgrade project
    construction_demand = build_city_demand  # goods consumed during construction

    color = color_rank_kingdom            # named color for map/UI display
    frame_tier = 3                        # UI frame level for location icon
    show_in_label = yes                   # show rank name in location tooltip label
}
```

The base rank has no `allow` block (it is the starting state) and typically has RGO and food modifiers:

```pdx
rural_settlement = {
    rank_modifier = {
        local_max_rgo_size_modifier = 1.0
        local_monthly_food_modifier = 0.1
        local_population_growth = 0.001
        local_peasant_enfranchisment = -0.2
    }
    country_modifier = { monthly_doom = 0.005 }
    color = color_rank_county
    frame_tier = 1
    show_in_label = no
    # no allow block — this is the base rank
}
```

## Key Fields Reference

| Field | Purpose | Notes |
|---|---|---|
| `rank_modifier` | Permanent modifiers at this rank | Applied directly to the location; stacks with buildings |
| `country_modifier` | Per-rank-location modifiers to owning country | `fort_limit`, `monthly_doom`, etc. |
| `allow` | Upgrade eligibility | Root = location; base rank omits this entirely |
| `max_rank` | Ceiling flag | Only one rank should have this |
| `is_established_city` | Marks as urban centre | Referenced by `is_established_city` triggers |
| `build_time` | Upgrade duration in days | |
| `construction_demand` | Goods consumed during build | References `common/goods_demand/` |
| `free_building_levels` | Extra building slots granted | City grants 100, town 25, rural_settlement 0 |
| `local_integration_speed_modifier` | Integration speed at this rank | Negative on city/town — larger settlements are harder to integrate |
| `local_pop_promotion_speed_modifier` | Pop type promotion speed | Only defined on `city` |
| `local_max_rgo_size_modifier` | RGO capacity | Only defined on `rural_settlement` |
| `color` | Map/UI color | Named color reference |
| `frame_tier` | UI icon frame level | Integer; 1 = smallest |
| `show_in_label` | Show rank in tooltip | `no` for base rank |

## Modding Notes
- The file comment is explicit: **higher ranks must come first**. The engine reads entries top-to-bottom and treats anything below a rank as worse. Place `city` above `town` above `rural_settlement`.
- The base rank (`rural_settlement`) has no `allow` block — it is the implicit starting state of all locations. Do not add an `allow` block to the base rank.
- `allow` for non-base ranks should check the previous rank is held (`location_rank = location_rank:town` before upgrading to city), ensuring the chain is stepped through in order.
- `rank_modifier` and `country_modifier` are separate: one buffs the location, the other the owning country per settlement of that rank.
- `free_building_levels` in `rank_modifier` grants extra building slots — the large city value (100) effectively uncaps constructions.
- `local_integration_speed_modifier = -0.5` on city means larger settlements are harder to culturally integrate — a deliberate design tradeoff.
- Cross-references: `buildings` (free_building_levels interacts with building caps), `goods_demand` (construction_demand), population system.

## Example

```pdx
town = {
    rank_modifier = {
        local_burghers_desired_pop = 0.2
        local_laborers_desired_pop = 0.2
        local_population_capacity = 20
        local_food_capacity = 250
        local_integration_speed_modifier = -0.25
        free_building_levels = 25
        local_peasant_enfranchisment = 0.1
        # ... (additional pop type and control modifiers)
    }

    country_modifier = { monthly_doom = 0.01 }

    allow = {
        population >= 5
        modifier:cannot_upgrade_location = no
        # no prior-rank check — any location with 5+ pop can become a town
    }

    color = color_rank_duchy
    is_established_city = yes
    build_time = 365
    construction_demand = build_town_demand
    frame_tier = 2
    show_in_label = yes
}
```
*The intermediate tier: modest capacity bonuses and 25 free building slots, upgradeable from any rural settlement with 5+ population. Note `town` does not require checking the prior rank — only `city` does.*
