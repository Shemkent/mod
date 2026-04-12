# Cabinet Actions
**Stage:** complete
**Keywords:** `cabinet_actions`, `ability`, `country_modifier`, `location_modifier`, `province_modifier`, `progress`, `is_finished`, `on_activate`, `select_trigger`

> **System type: Gameplay**
> **Cluster:** Government & Politics

## Overview
Cabinet actions represent ongoing tasks assigned to cabinet ministers (Administrative, Diplomatic, or Military). While a minister is assigned to an action, its modifiers apply continuously — scaled by `1 + (effective ability + cabinet efficiency) × 0.05` — until a `progress` threshold is reached, an `is_finished` trigger fires, or the assignment ends. The system uses the same `select_trigger` machinery as country interactions and resolutions, allowing actions to target provinces, locations, countries, areas, or characters. Each cabinet slot can hold one active action at a time.

## Vanilla File Locations
`in_game/common/cabinet_actions/` — 65 files. Full list in `_file_index.csv`.

## Block Structure

```pdx
my_cabinet_action = {
    ability = adm                  # adm / dip / mil — determines which minister handles this

    allow_multiple = yes           # [optional] allow parallel instances on different targets

    forbid_for_automation = yes    # [optional] AI will not pick this action

    potential = {                  # when this action is visible in the cabinet UI
        modifier:allow_x = yes
    }

    allow = {                      # when this action can be assigned (greyed out if false)
        num_locations > 0
    }

    select_trigger = {             # [repeatable] interactive target selection stage
        looking_for_a = location
        source = actor
        source_flags = only_actual_locations
        target_flag = target
        name = "select_province"
        none_available_msg_key = "no_provinces_msg"
        column = { data = name }
        column = { data = development }
        visible = { has_owner = yes  owner ?= scope:actor }
        enabled = { integration_level = core }
        map_color = { lerp = { min_color = define:NMapColors|MAP_COLOR_LOW
                               max_color = define:NMapColors|MAP_COLOR_HIGH
                               factor = { value = development  divide = 100 } } }
    }

    country_modifier = {           # modifier applied to the country while active
        global_migration_speed_modifier = 0.5
    }

    location_modifier = {          # modifier applied to the selected target location while active
        local_monthly_development = 0.0025
    }

    province_modifier = {          # modifier applied to all locations in the target province/area
        local_pop_assimilation_speed = 0.02
    }

    societal_values = 0.1          # [optional] monthly push toward a societal value while active

    progress = {                   # [optional] script value determining completion progress per tick
        scope:target = {
            value = province_average_development
        }
    }

    is_finished = {                # [optional] trigger; action ends when this fires
        scope:actor = {
            cultural_unity >= 1.0
        }
    }

    on_activate = {                # [optional] effect fired when action is assigned
        scope:target = { set_variable = { name = promoting_culture  value = scope:target_culture } }
    }

    on_fully_activated = {         # [optional] effect fired when action completes via progress
    }

    on_deactivate = {              # [optional] effect fired when action ends for any reason
    }

    ai_will_do = {                 # [optional] AI scoring for whether to assign this action
        value = 10
    }

    map_marker = {                 # [optional] visual indicator on the map
        show_for_owner = yes
    }
}
```

## Key Fields Reference

| Field | Purpose | Notes |
|---|---|---|
| `ability` | Which minister type handles the action | `adm` / `dip` / `mil` |
| `allow_multiple` | Permit multiple simultaneous instances | One per different target; default: no |
| `forbid_for_automation` | Blocks AI from auto-assigning | Use when the action requires human judgment |
| `potential` | Visibility condition | Scope: actor = acting country |
| `allow` | Enabledness condition | Shown greyed out when false |
| `select_trigger` | Target selection stage | Repeatable; same pipeline as country_interactions and resolutions |
| `country_modifier` | Ongoing country-level modifier | Applied while action is assigned |
| `location_modifier` | Ongoing location-level modifier | Applies to scope:target location |
| `province_modifier` | Ongoing modifier for all locations in target area/province | Used for area-wide actions like `integrate_area` |
| `societal_values` | Societal value monthly push rate | Flat float value |
| `progress` | Completion speed script value | Action ends when threshold is reached |
| `is_finished` | Completion trigger | Alternative to `progress`; action ends when this fires |
| `on_activate` | Effect on assignment | Scope: actor, target, etc. |
| `on_fully_activated` | Effect on completion via progress | |
| `on_deactivate` | Effect on end (any cause) | |
| `ai_will_do` | AI scoring for assigning this action | Complement to `forbid_for_automation` — guides rather than blocks |
| `map_marker` | Map UI indicator | `show_for_owner = yes` shows pin for owner |

## Modding Notes
- `select_trigger` syntax is identical to `country_interactions` and `resolutions` — all three share the same selection pipeline. See those annotations for the full parameter reference.
- `ability` must match a defined cabinet minister type. Assigning an action to the wrong minister type will silently fail.
- **Modifier scaling**: modifiers are not flat — they are scaled by `1 + (effective ability + cabinet efficiency) × 0.05`. A minister with high ability delivers meaningfully stronger modifiers.
- Two completion mechanisms exist: `progress` (threshold reached over time) and `is_finished` (trigger fires). Many actions use only one; some use both. Omitting both means the action runs until manually cancelled.
- `country_modifier`, `location_modifier`, and `province_modifier` can coexist. Location modifier targets `scope:target`; province modifier targets all locations in the selected area/province.
- `forbid_for_automation = yes` blocks the AI entirely; `ai_will_do` provides a score for AI evaluation instead. Use `forbid_for_automation` for map-selection actions the AI cannot reliably evaluate.
- `on_activate` fires on assignment (before any progress); useful for setting variables or applying one-time setup effects.
- Modifiers are removed immediately when the action ends or is replaced.
- `potential` controls whether the action appears in the UI; gate behind unlocked modifiers or policies.

## Example

```pdx
cab_diplomatic_corps = {
    ability = dip

    potential = {
        modifier:allow_cabinet_diplomatic_corps = yes
    }

    country_modifier = {
        diplomatic_capacity_modifier = 0.2
        monthly_diplomats = 0.2
    }
}
```
*Simple diplomatic cabinet action with no target selection — applies country-wide bonuses while the diplomatic minister is assigned.*

```pdx
develop_province = {
    ability = adm
    allow_multiple = yes

    potential = { num_locations > 0 }

    select_trigger = {
        looking_for_a = province
        source = actor
        target_flag = target
        name = "develop_province_select_province"
        column = { data = name }
        column = { data = development }
        visible = {
            NOT = { cabinet_already_performing_same_task_on_target = {
                cabinet_action = develop_province  interaction_target = target } }
        }
        enabled = { any_location_in_province = { integration_level = core  count = all } }
    }

    location_modifier = { local_monthly_development = 0.0025 }

    progress = { scope:target = { value = province_average_development } }
}
```
*Multi-instance administrative action: each assigned minister develops a different province; progress scales with the province's existing development.*
