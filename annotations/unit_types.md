# Unit Types
**Stage:** complete
**Keywords:** `unit_key`, `category`, `buildable`, `max_strength`, `combat_power`, `maintenance_demand`, `construction_demand`, `age`, `copy_from`, `levy`, `impact`, `combat`, `strength_damage_taken`, `morale_damage_taken`, `bombard_efficiency`, `gfx_tags`, `unit_ability`

> **System type: Gameplay**

## Overview
Unit types define individual military and naval units — their combat stats, terrain modifiers, supply costs, and visual tags. Each unit type belongs to a **category** (defined in `unit_categories/`) that supplies base stats and role defaults; unit types override or extend those defaults. The `copy_from` field copies all fields from another unit type before applying local overrides, enabling inheritance chains (e.g. unique noble cavalry copying a base age template). Unit abilities in `unit_abilities/` define scripted actions units can perform in the field.

## Vanilla File Locations

| File | Content |
|---|---|
| `in_game/common/unit_types/00_age_templates_land.txt` | Base land unit templates per age (infantry, cavalry, artillery, auxiliary) |
| `in_game/common/unit_types/00_age_templates_navy.txt` | Base naval unit templates per age |
| `in_game/common/unit_types/0_knights.txt` | Unique noble cavalry units (levy-only) |
| `in_game/common/unit_types/0_tribal.txt` | Tribal unit types |
| `in_game/common/unit_types/1_uniques_for_age_*.txt` | Age-gated unique unit types (1–6) |
| … *(29 files total)* | Grouped by role/age |
| `in_game/common/unit_categories/` | 8 files — one per category (army_infantry, army_cavalry, etc.) |
| `in_game/common/unit_abilities/` | 16 files — one per scripted unit action |

## Block Structure

**Unit category** (defines base stats for a role):
```pdxscript
army_infantry = {
    is_army       = yes
    assault       = yes
    is_garrison   = yes
    movement_speed = 2.5
    max_strength  = 0          # overridden per unit type
    combat_power  = 1
    frontage      = 1
    initiative    = 1
    combat_speed  = 2
    flanking_ability = 1.0
    supply_weight = 1.0
    food_storage_per_strength  = 1
    food_consumption_per_strength = 1
    damage_taken  = 1.0
    maintenance_demand   = infantry_maintenance
    construction_demand  = infantry_construction
    startup_amount = 20
    ai_weight     = 0.5
}
```

**Unit type** (inherits from category, extends or overrides):
```pdxscript
unit_key = {
    category    = army_cavalry       # required; determines role and base stats
    copy_from   = a_age_5_absolutism_cavalry  # optional; copies all fields from another unit type first

    buildable   = yes/no             # can players recruit this unit; no = levy/estate only
    levy        = yes/no             # provided by levy system
    max_strength = 0.2               # proportion of army stack; 0 = use category default
    combat_power = 4                 # base combat effectiveness

    age         = "age_1_traditions" # age this unit is associated with

    maintenance_demand   = heavy_cavalry_maintenance
    construction_demand  = heavy_cavalry_construction

    # Terrain modifiers (additive on category base):
    impact = { jungle = 0.10  mountains = 0.10 }   # modifier when entering terrain
    combat = { jungle = -0.10 mountains = -0.10 }  # modifier during combat in terrain

    # Combat stat overrides:
    strength_damage_taken  = -0.25   # negative = takes less strength damage
    morale_damage_taken    = -0.25
    bombard_efficiency     = 0.1     # artillery only
    artillery_barrage      = 1       # artillery only

    gfx_tags = { heavy_tag }         # visual/animation tags
}
```

**Unit ability** (scripted field action):
```pdxscript
ability_key = {
    army_only  = yes/no
    navy_only  = yes/no
    toggle     = yes/no              # can be toggled on/off
    duration   = <days>              # -1 = indefinite
    cancel_on_combat     = yes/no
    cancel_on_combat_end = yes/no
    cancel_on_move       = yes/no

    allow          = { <triggers> }  # unit scope; when the ability can be used
    finished_when  = { <triggers> }  # unit scope; when a timed ability completes
    ai_will_do     = { <script_value> }
    ai_will_revoke = { <triggers> }

    modifier       = { <modifier_fields> }  # applied to unit while active
    start_effect   = { <effects> }
    finish_effect  = { <effects> }
    on_entering_location = { <effects> }
}
```

## Key Fields Reference

| Field | Purpose | Notes |
|---|---|---|
| `category` | Role and base stat source | Required; determines `is_army`/`is_navy`, movement, supply defaults |
| `copy_from` | Inheritance template | Copies all fields from the named unit at load time; explicitly listed local fields then override. Not a live reference — changing the source does not retroactively update copies |
| `buildable` | Recruitable by player | `no` = cannot be manually recruited; combine with `levy = yes` for levy-exclusive units |
| `levy` | Provided by levy system | `yes/no`; independent of `buildable` — both must be set explicitly. `buildable = no` + `levy = yes` is the standard levy pattern |
| `max_strength` | Size proportion of army stack | 0 in category definitions means "dynamically sized by engine"; unit types should set a concrete non-zero value |
| `age` | Age association | Used for age-gated availability and template organisation |
| `frontage` | Combat front slots occupied | Category-level; higher frontage = unit takes more space in the battle line |
| `initiative` | Priority in combat turn order | Category-level; higher = acts earlier each combat round |
| `startup_amount` | Initial army composition count | Category-level hint for starting armies; not a hard cap |
| `impact` | Terrain movement modifier | Additive on category base; keyed to terrain type strings |
| `combat` | Terrain combat modifier | Additive on category base; keyed to terrain type strings |
| `gfx_tags` | Visual/animation tag set | Links unit to GFX definitions (e.g. `heavy_tag`, `light_tag`); valid tags are defined in GFX files — check `gfx/` for available tags; unknown tags are ignored |
| `finished_when` *(ability)* | Timed ability completion condition | Unit scope; ability ends when this trigger passes |
| `on_entering_location` *(ability)* | Location-entry effect | Fires each time the unit moves into a new location while ability is active |

## Modding Notes
- **Category defines role** — `is_army`, `assault`, `is_garrison`, flanking, supply weight are all category-level; unit types cannot change these without changing the category.
- `buildable = no` + `levy = yes` is the standard pattern for levy-only units. The two fields are independent — `buildable = no` alone means a unit cannot be built or levied; only `levy = yes` opts it into the levy system.
- Terrain keys in `impact`/`combat` must match valid terrain type keys — using an unknown terrain key is silently ignored.
- Unit abilities are wired to units through GUI layout files (not in unit type script); the `army_only`/`navy_only` flags determine broad availability, and the `allow` trigger in each ability restricts it further. To restrict an ability to a specific unit type, add a unit-type check inside `allow` (e.g. `unit_type = unit_type:my_unit`).
- Adding new unit types requires corresponding localisation and GFX entries; missing loc produces raw key display.

## Example

```pdxscript
# Age template — not directly buildable, used as copy_from source
a_age_3_discovery_cavalry = {
    category     = army_cavalry
    buildable    = no    # not recruited directly; this is a template for copy_from
    # levy = yes is NOT set here — this is not a levy unit either;
    # it exists purely as an inheritance base for unique units below
    max_strength = 0.2
    combat_power = 4
    age          = "age_3_discovery"
    maintenance_demand   = standard_cavalry_maintenance
    construction_demand  = standard_cavalry_construction
}

# Unique unit copying and overriding the age template
a_gendarmerie = {
    category   = army_cavalry
    copy_from  = a_age_6_revolutions_cavalry
    buildable  = no
    levy       = yes
    strength_damage_taken = -0.25
    morale_damage_taken   = -0.25
    maintenance_demand    = heavy_cavalry_maintenance
    construction_demand   = heavy_cavalry_construction
    combat = { grasslands = 0.10  wetlands = -0.10 }
    gfx_tags = { heavy_tag }
}

# Unit ability — toggle that boosts morale damage while active
forced_march = {
    army_only  = yes
    toggle     = yes
    duration   = -1                     # indefinite until toggled off
    cancel_on_combat = yes

    allow = { location = { is_land = yes } }

    modifier = {
        movement_speed = 0.25
        morale_damage_taken = 0.15      # penalty while marching hard
    }

    start_effect  = { }
    finish_effect = { }
}
```
