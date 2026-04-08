# Buildings System
**Stage:** complete
**Game version:** 1.1.10
**Keywords:** building_types, building_categories, construction, modifiers, production_methods, employment, estates, foreign buildings

> **System type: Gameplay**

## Overview

Buildings are location-level constructs that employ pops, consume goods, and apply modifiers. Each building belongs to a category, has one pop type, and uses production methods for maintenance costs. Buildings can have multiple levels, be restricted by location rank (rural_settlement / town / city), and can be conditionally available or automatically removed.

There is no single `buildings/` folder — everything lives under `building_types/` and `building_categories/`.

---

## Vanilla File Locations

| Path | Purpose |
|---|---|
| `in_game/common/building_types/` | All building definitions, split into themed files |
| `in_game/common/building_categories/00_default.txt` | Category tags (name-only containers) |
| `in_game/common/building_types/readme.txt` | Authoritative attribute reference |

### building_types files (by theme)

| File | Contents |
|---|---|
| `common_buildings.txt` | Infrastructure, rural resource extractors (quarry, mill, farm…) |
| `town_buildings.txt` | Town-tier civic buildings |
| `capital_buildings.txt` | Capital-only government buildings |
| `forts.txt` | Fort upgrade chain (stockade → castle → bastion → star_fort → fortress) |
| `coastal_forts.txt` | Coastal/port defensive buildings |
| `port_buildings.txt` | Port infrastructure |
| `trade_buildings.txt` | Marketplace, merchants' quarters |
| `market_buildings.txt` | Market center buildings |
| `religion_buildings.txt` | Churches, temples, mosques |
| `culture_buildings.txt` | Libraries, academies, cultural buildings |
| `estate_buildings.txt` | Estate-owned buildings (nobles, clergy, burghers, etc.) |
| `foreign_buildings.txt` | Buildings built in foreign-owned locations (trade_office, embassy) |
| `trade_company_buildings.txt` | Buildings exclusive to trade company territory |
| `unique_buildings.txt` | Culture/reform-gated special buildings (is_special = yes) |
| `production_*.txt` | Industry buildings for specific goods (glass, cloth, wine, etc.) |
| `rural_buildings.txt` | Rural-specific buildings |
| `manpower_buildings.txt` | Military manpower buildings |
| `pirate_buildings.txt` | Pirate reform-specific buildings |
| `plantation_buildings.txt` | Colonial plantation buildings |
| `situation_buildings.txt` | Situation-triggered buildings |
| `event_only_buildings.txt` | Buildings only placeable via event effects |
| `00_unique_buildings_to_make_obsolete.txt` | Legacy obsolescence entries |

---

## Block Structure

```
building_key = {
    # --- Identity & constraints ---
    category          = <building_category>     # Required. Groups building in UI and logic.
    pop_type          = <pop_type>              # Pop type employed. E.g. laborers, burghers, soldiers.
    max_levels        = <scripted_int|int>      # Maximum buildable levels. Root = location.
    employment_size   = <scripted_float|float>  # Employed pop size per level (1 = 1000 people).

    # --- Location rank filters (yes/no) ---
    rural_settlement  = yes/no
    town              = yes/no
    city              = yes/no

    # --- Ownership & foreign placement ---
    is_foreign        = yes/no          # Can be built in foreign-owned locations.
    in_empty          = empty|owned|any # empty = unowned only, owned = owned only, any = either.
    stronger_power_projection = yes/no  # Requires stronger PP than location owner (foreign buildings).
    need_good_relation = yes/no         # Requires good relations with location owner (foreign buildings).

    # --- Availability triggers ---
    location_potential = { <triggers> } # "Visible" check. Root = location.
    country_potential  = { <triggers> } # "Visible" check. Root = country.
    allow              = { <triggers> } # "Enabled" check. Root = location, scope:actor = builder.
    can_destroy        = { <triggers> } # Can the building be manually destroyed.
    remove_if          = { <triggers> } # Auto-removes when true. Root = building.
    is_indestructible  = yes/no         # Cannot be destroyed by button or effect (remove_if still works).

    # --- Estate association ---
    estate             = <estate_type>  # Only this estate can build it.
    forbidden_for_estates = yes         # No estate can build it; only the country can.

    # --- Costs & timing ---
    build_time         = <scripted_int|int>         # Days to build.
    construction_demand = <scripted_block>          # Goods consumed during construction.
    price              = <price>                    # Gold cost.
    destroy_price      = <price>                    # Gold cost to demolish.
    increase_per_level_cost = <float>               # % cost increase per additional level (0.5 = +50%).

    # --- Production methods ---
    possible_production_methods = { <pm_keys> }    # Named PM slots (from production_methods/).
    unique_production_methods = {                  # Inline PMs (not in separate files).
        pm_key = {
            <good> = <float>    # Input good (consumed per tick).
            produced = <good>   # Output good (if any).
            output = <float>    # Output quantity.
            category = <pm_category>
        }
    }

    # --- Modifiers ---
    modifier              = { <modifiers> }  # Applied to location, scaled by level × goods access.
    raw_modifier          = { <modifiers> }  # Applied to location, NOT scaled.
    capital_modifier      = { <modifiers> }  # Applied to location only if in the capital.
    capital_country_modifier = { <modifiers> } # Applied to country only if built in the capital.
    market_center_modifier = { <modifiers> } # Applied to location only if in a market center.

    # --- Lifecycle events ---
    on_built    = { <effects> }
    on_destroyed = { <effects> }

    # --- Misc ---
    pop_size_created    = <float>      # Foreign buildings: creates a pop of this size from capital.
    obsolete            = <building_key> # This building makes the named one obsolete when built.
    expensive           = yes          # AI/UI flag for high-cost buildings.
    automation_build_allowed = yes/no  # Whether automation can build this.
    ai_forbid_shutdown  = yes/no       # AI won't disable this building.
    AI_ignore_available_worker_flag = yes/no  # AI builds even if pop type unavailable.
    important_for_AI    = yes/no       # AI uses extra performance to find suitable locations.
    important_for_UI    = yes/no       # Appears in high-priority UI alerts.
    is_special          = yes          # Marks culture/reform-gated unique buildings.
    custom_tags         = { <strings> } # Custom identifier strings for scripting.
    graphical_tags      = { <strings> } # Hints for 3D model selection.
}
```

---

## Key Fields Reference

| Field | Purpose | Key constraint |
|---|---|---|
| `category` | Groups the building in UI and logic; determines which estate-related rules apply | Required; must match a key in `building_categories/` |
| `pop_type` | Pop type employed by the building | Required; must be a valid pop type key |
| `max_levels` | Maximum buildable levels; accepts a scripted int or literal | Usually references a `script_values/` constant |
| `employment_size` | Employed pop size per level (1.0 = 1000 people) | Accepts scripted float; scales linearly with levels |
| `rural_settlement` / `town` / `city` | Boolean flags controlling which location ranks can host this building | At least one must be `yes`; omitting all prevents construction anywhere |
| `is_foreign` | Allows the building to be constructed in foreign-owned locations | Typically paired with `stronger_power_projection` and/or `need_good_relation` |
| `location_potential` / `country_potential` | "Visible" gates — building is hidden if these fail; root = location or country respectively | Failing potential hides the building entirely (not just greyed out) |
| `allow` | "Enabled" gate — building is visible but locked if this fails; root = location, `scope:actor` = builder | Use `?=` when owner may be absent to avoid crashes |
| `remove_if` | Auto-removes the building when the block evaluates true; root = building | Engine evaluates monthly; safe to use `?=` for nullable scopes |
| `modifier` | Modifiers applied to the location, scaled by `building_level × goods_access` | Do NOT put `fort_level` or `propagating_zone_of_control` here — use `raw_modifier` |
| `raw_modifier` | Modifiers applied to the location, NOT scaled by level or goods access | Required for `fort_level` and `propagating_zone_of_control` |
| `build_time` / `price` | Days to construct and gold cost respectively | Both accept scripted values; prefer `script_values/` constants over literals |
| `possible_production_methods` | List of named PM keys (from `production_methods/`) selectable for this building | PM keys must exist in the named PM files |
| `unique_production_methods` | Inline PM definitions specific to this building; same syntax as named PMs | Inline PMs are not available to other buildings |
| `on_built` / `on_destroyed` | Effect blocks fired when the building is constructed or demolished | Root = location at time of trigger |
| `estate` | Restricts construction to the named estate | Use `forbidden_for_estates = yes` to bar all estates (country-only construction) |
| `obsolete` | When this building is built, the named building is automatically removed | Used for upgrade chains (e.g. fort tiers); one entry per obsoleted building |

---

## Building Categories

Defined in `building_categories/00_default.txt`. Categories are pure name-tags — no sub-fields. They group buildings for UI filtering and can be checked in scripted triggers.

```
rgo_building_category = {}
basic_industry_category = {}
weapons_industry_category = {}
consumer_goods_category = {}
government_category = {}
infrastructure_category = {}
religious_category = {}
cultural_category = {}
trade_category = {}
naval_category = {}
military_category = {}
defense_category = {}
village_category = {}
colonial_category = {}
estate_category = {}
```

---

## Key Mechanics

### Modifier scaling
`modifier` and `capital_modifier` values are multiplied by `(building_level × goods_access)`. Use `raw_modifier` for flat values that should not scale — primarily used for `fort_level` and `propagating_zone_of_control`.

### Location rank gating
`rural_settlement`, `town`, `city` are boolean flags on the building definition itself. If none is set to `yes`, the building cannot be built anywhere. Many buildings allow all three; capital-only buildings typically omit `rural_settlement`.

### Obsolescence chain
`obsolete = <building_key>` removes the named building when this one is constructed. The fort chain is the canonical example:
```
stockade → (castle obsoletes stockade) → (bastion obsoletes castle) → (star_fort obsoletes bastion) → (fortress obsoletes star_fort)
```

### Foreign buildings
Set `is_foreign = yes`. Usually also set `stronger_power_projection` and/or `need_good_relation`. The `allow` trigger scope is `root = location, scope:actor = builder`. Foreign buildings can apply `modifier` to the **building owner's** stats via `building_owner_*` modifier keys.

### Estate buildings
Set `estate = <estate_type>` to restrict building to a specific estate. Use `forbidden_for_estates = yes` to prevent any estate from building it (only the country can). Estate buildings use `estate_category` and often have `estate_build_time` and `estate_construct_building`.

### Capital-only buildings
Use `location_potential = { is_capital = yes }` combined with `remove_if = { is_no_longer_capital = yes }`. Capital modifiers use `capital_modifier` (affects the capital location) or `capital_country_modifier` (affects the whole country).

### Unique / culture-gated buildings
Mark with `is_special = yes`. Gate with `country_potential` (reform, culture, etc.) and `location_potential` (dominant culture, etc.).

---

## Example

### 1. Simple rural resource extractor
```
lumber_mill = {
    is_foreign = no
    max_levels = rural_building_cap
    pop_type = laborers
    employment_size = rural_peasant_produce_employment
    category = rgo_building_category
    rural_settlement = yes
    town = yes
    city = yes
    location_potential = {
        OR = { vegetation = woods  vegetation = forest  vegetation = farmland  vegetation = jungle }
        NOT = { raw_material = goods:lumber }
    }
    build_time = rural_build_time
    unique_production_methods = {
        lumber_mill_maintenance = {
            produced = lumber
            output = 1
            tools = 0.4
            category = building_maintenance
        }
    }
    construction_demand = basic_construction_needs
}
```
Key points: `location_potential` gates it to forested tiles without native lumber; the inline PM declares both input (`tools`) and output (`lumber`).

### 2. Fort with raw_modifier
```
castle = {
    is_foreign = no
    max_levels = 1
    pop_type = soldiers
    forbidden_for_estates = yes
    category = defense_category
    obsolete = stockade
    employment_size = castle_employment
    rural_settlement = yes  town = yes  city = yes
    build_time = medium_fort_building
    expensive = yes
    unique_production_methods = {
        castle_maintenance = { stone = 0.1  tools = 0.05  weaponry = 0.1  tar = 0.025  category = building_maintenance }
    }
    construction_demand = castle_construction
    modifier = { local_garrison_size = 0.5  local_unrest = -0.05  local_nobles_estate_power = 1.0 }
    raw_modifier = { fort_level = 2  local_nobles_desired_pop = 0.02  propagating_zone_of_control = yes }
}
```
`fort_level` and `propagating_zone_of_control` must be in `raw_modifier` — they are not goods-access-scaled values.

### 3. Capital government building with country_potential gate
```
royal_court = {
    is_foreign = no
    pop_type = nobles
    max_levels = 1
    category = government_category
    employment_size = large_noble_employment
    town = yes  city = yes
    expensive = yes
    build_time = large_capital_build_time
    location_potential = { is_capital = yes }
    country_potential = { government_type = government_type:monarchy }
    remove_if = {
        OR = {
            is_no_longer_capital = yes
            location.owner ?= { government_type != government_type:monarchy }
        }
    }
    capital_country_modifier = {
        country_cabinet_efficiency = 0.02
        global_crown_estate_power = 0.10
        monthly_towards_absolutism = societal_value_tiny_monthly_move
    }
    unique_production_methods = {
        royal_court_maintenance = { furniture = 0.15  tools = 0.05  glass = 0.1  fine_cloth = 0.2
                                    books = 0.1  goods_gold = 0.1  marble = 0.1  category = building_maintenance }
    }
    graphical_tags = { palace }
    construction_demand = early_capital_building_construction
}
```
`capital_country_modifier` applies to the whole country (not just the capital location). `remove_if` cleans up the building if the government type changes or the capital moves.

---

## Modding Notes

- To add a new building, create a new `.txt` file in `building_types/` or append to an existing themed file.
- New building categories can be added to `building_categories/00_default.txt` as empty blocks.
- `scripted_values` (e.g. `rural_building_cap`, `guild_max_level`) are defined in `script_values/` — always prefer referencing these over hardcoded integers.
- `construction_demand` blocks are defined in their own files; reuse vanilla ones where possible.
- The `?=` scoping operator in `remove_if` handles the case where the location has no owner (prevents a crash).
