# Employment Systems
**Stage:** complete
**Keywords:** employment_system, priority, country_modifier, ai_will_do, building_potential_profit, building_index, building_category, infrastructure_category, government_category, capitalism, equality, first_come_first_serve, societal_value_monthly_move, societal_value_minor_monthly_move

> **System type: Gameplay**

## Overview
Employment systems determine the order in which buildings across a country receive workers (pops). When total labour demand exceeds supply, the system's `priority` script value ranks buildings — higher scores are staffed first. Each system also grants a country-level modifier while it is active and has an `ai_will_do` block that governs AI selection. In vanilla four systems exist: `equality`, `first_come_first_serve`, `capitalism`, and `capitalism_prioritising_infrastructure`. The system chosen nudges societal values and shapes which sectors (military, manufacturing, agriculture) get staffed under labour scarcity.

## Vanilla File Locations
- `in_game/common/employment_systems/00_default.txt` — all employment system definitions (one file, 4 systems)
- `in_game/common/employment_systems/readme.txt` — field documentation
- Full file list in `_file_index.csv`.

## Block Structure
```pdx
<employment_system_id> = {
    country_modifier = {
        <modifier_key> = <value>    # modifier applied to the country while this system is active
    }

    priority = {
        value = <script value>      # root = building; higher score = staffed first
        multiply = <float>          # optional: multiplier on the value (e.g. -1 to invert)
        if = {                      # optional: conditional additive bonus for a subset of buildings
            limit = { <trigger> }
            add = <float>
        }
    }

    ai_will_do = {
        <script value>              # root = country; score for AI choosing this system
    }
}
```

## Vanilla Systems
| ID | Priority logic | Country modifier |
|---|---|---|
| `equality` | All buildings equal priority (value = 1) | Monthly push toward communalism |
| `first_come_first_serve` | Oldest buildings filled first (lower `building_index` → higher score after `× −1`) | Monthly push toward traditional economy |
| `capitalism` | Highest-profit buildings filled first (`building_potential_profit`) | Monthly push toward capital economy |
| `capitalism_prioritising_infrastructure` | Profit-based, but infrastructure/government buildings get +10,000 boost | Minor push toward both capital economy and communalism |

## Key Fields Reference
| Field | Purpose | Key constraint |
|---|---|---|
| `priority` | Script value (root = building) returning a sort score | Higher = staffed earlier; the engine sorts all buildings by this value before assigning workers |
| `country_modifier` | Modifier applied to the country while this system is active | Only one system is active at a time; switching removes the old modifier |
| `ai_will_do` | Score for AI selecting this system | AI picks the highest-scoring system; vanilla uses −10 inertia (another system must score ≥10 higher to trigger a switch) plus a societal value term — all four vanilla systems use a convergence pattern: the AI selects a system when societal values are opposite to what it promotes |

## Modding Notes
- **Only one system is active per country at a time.** Switching systems (via laws or events) immediately removes the old `country_modifier` and applies the new one.
- **`priority` runs per building**, not per location. Root is the building being evaluated. Use building-scope script values and conditions — `building_potential_profit`, `building_index`, `building_category`, etc.
- **`ai_will_do` uses societal values** as the primary driver with a convergence pattern: each vanilla system is selected by the AI when the country's societal values are opposite to what the system promotes, nudging the country toward equilibrium. The `−10` inertia base creates a threshold — another system must score at least 10 points higher via societal alignment before the AI will switch. This prevents rapid oscillation. Per-system AI selection direction: `equality` selected when leans individualist; `first_come_first_serve` when leans capital economy; `capitalism` when leans traditional economy; `capitalism_prioritising_infrastructure` when leans traditional economy and individualist simultaneously.
- **`priority` supports conditional bonuses** via `if/limit/add` blocks, allowing a subset of buildings (e.g. those in `infrastructure_category` or `government_category`) to receive a large flat bonus. `capitalism_prioritising_infrastructure` uses `add = 10000` to ensure infrastructure buildings are always staffed before profit-based ones.
- **Adding a new employment system** is purely additive — define the block, add localization, and surface the choice via a law or interaction. When the new system is set as a country's active system, the engine uses its `priority` block to rank buildings.
- **Cross-system:** the `country_modifier` from the active employment system stacks with all other country modifiers (laws, government reforms, etc.). Societal value modifiers (e.g. `monthly_towards_capital_economy`) interact with the cultures and societal values systems.

## Example
`capitalism` — staffs the most profitable buildings first:

```pdx
capitalism = {
    country_modifier = {
        monthly_towards_capital_economy = societal_value_monthly_move
    }

    priority = {
        value = building_potential_profit   # script value: expected profit of this building
    }

    ai_will_do = {
        add = {
            desc  = "inertia"
            value = -10                     # base inertia discourages switching
        }
        add = {
            desc  = "societal_values"
            value = societal_value:capital_economy_vs_traditional_economy
            multiply = -1                   # inverts value: traditional-leaning (negative) → positive score contribution
        }
    }
}
```

When capitalism is active, labour fills the highest-earning buildings first, leaving low-profit agriculture understaffed under scarcity. The AI switches to capitalism when the societal value `capital_economy_vs_traditional_economy` is negative (country leans traditional): `multiply = -1` inverts this to a positive score contribution, making capitalism the convergence choice that pushes the country back toward a capital economy.
