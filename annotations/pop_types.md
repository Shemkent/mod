# Pop Types
**Stage:** complete
**Keywords:** pop_type, pop_types, nobles, clergy, burghers, laborers, soldiers, peasants, tribesmen, slaves, promote_to, promotion_factor, migration_factor, pop_food_consumption, upper, has_cap, grow, literacy_impact, assimilation_conversion_factor, city_graphics, editor, dhimmi_estate, tribes_estate, pop_percentage_impact

> **System type: Data/Reference**

## Overview
Pop types define the distinct population classes that inhabit EU5's locations — nobles, clergy, burghers, laborers, soldiers, peasants, tribesmen, and slaves. Each type controls food consumption, promotion and migration rates, estate assignments, literacy modifiers, and how much the pop contributes to city graphics. The pop type system interacts with goods demand (via `demand_add`/`demand_multiply` in `goods/`), literacy-driven local modifiers, estate assignment, and promotion chains. Modders adjusting pop balance, adding new pop categories, or tuning economic output work directly in this file.

## Vanilla File Locations
- `in_game/common/pop_types/00_default.txt` — all pop type definitions (one file, 8 types)
- Full file list in `_file_index.csv`.

## Block Structure
```pdx
<pop_type_id> = {
    # --- Display ---
    color       = <color_key>
    city_graphics = <float>         # contribution to city visual scale (higher = bigger city look)

    # --- Editor ---
    editor = <float>                # starting proportion of this pop type in the editor

    # --- Food ---
    pop_food_consumption = <float>  # food units consumed per 1000 pops per month

    # --- Social mobility ---
    promote_to   = <pop_type_id>    # repeatable; which types this pop can promote into (omit if none)
    promotion_factor = <float>      # base speed of upward promotion (omit if none, e.g. slaves, tribesmen)
    migration_factor = <float>      # willingness to migrate between locations

    # --- Growth ---
    grow = yes                      # this pop type can naturally grow (not just from promotion)

    # --- Estate assignment (repeatable; multiple lines for different conditions) ---
    # Each key maps this pop type to an estate when a condition is met.
    # The value is a trigger block evaluated at assignment time.
    <estate_type_id>_estate = { <trigger> }   # e.g. dhimmi_estate = { is_dhimmi = yes }
    <estate_type_id>_estate = { <trigger> }   # e.g. nobles_estate = {}  (empty trigger = always)

    # --- Social tier ---
    upper   = yes                   # marks as upper-class (nobles, clergy, burghers)
    has_cap = yes                   # subject to population cap mechanics

    # --- Tribal flag ---
    tribal_rules = yes              # enables special tribal promotion factors (tribesmen only)

    # --- Trade ---
    counts_towards_market_language = yes/no  # this pop type counts toward the dominant market language calculation (burghers only in vanilla)

    # --- Assimilation ---
    assimilation_conversion_factor = <float>  # how quickly this pop converts/assimilates

    # --- Literacy modifiers (applied per literate pop of this type in the location) ---
    literacy_impact = {
        <modifier_key> = <float>    # local modifier applied based on literate pop proportion
    }

    # --- Population percentage modifiers (applied based on share of total pops) ---
    pop_percentage_impact = {
        <modifier_key> = <float>
    }

    # --- Goods demand (defined in goods/ files, not here — referenced by pop type ID) ---
    # demand_add / demand_multiply entries in goods definitions reference these IDs
}
```

## Pop Types in Vanilla
The "Tier" column is an editorial grouping — there is no tier field in the game files.

| Type | Tier | Can grow | Can promote to | Estate |
|---|---|---|---|---|
| `nobles` | upper | no | — | nobles_estate / dhimmi_estate / tribes_estate |
| `clergy` | upper | no | — | clergy_estate / dhimmi_estate / tribes_estate |
| `burghers` | upper | no | — | burghers_estate / dhimmi_estate / tribes_estate |
| `laborers` | lower | no | — | peasants_estate / dhimmi_estate / tribes_estate |
| `soldiers` | lower | no | — | peasants_estate / dhimmi_estate / tribes_estate |
| `peasants` | base | yes | nobles, clergy, burghers, laborers, soldiers | peasants_estate / dhimmi_estate / tribes_estate |
| `tribesmen` | tribal | yes | peasants | tribes_estate / cossacks_estate |
| `slaves` | base | no | — | peasants_estate / dhimmi_estate |

## Key Fields Reference
| Field | Purpose | Key constraint |
|---|---|---|
| `pop_food_consumption` | Food consumed per 1000 pops per month | Ranges from 0.0 (tribesmen — cost no food) to 20.0 (nobles); soldiers consume 5.0, clergy 5.0, burghers 4.0, laborers/peasants/slaves 1.0 |
| `promotion_factor` | Base rate at which this pop promotes upward | Higher = faster promotion; laborers (1.5) highest among non-base types; peasants (0.5); nobles/clergy (0.1); slaves and tribesmen have no `promotion_factor` |
| `migration_factor` | Willingness to move between locations | Lowest for slaves (0.01), soldiers (0.05); laborers and peasants (0.1); burghers have the highest (0.5) |
| `promote_to` | Which pop types this type can upgrade into | Repeatable; only `peasants` and `tribesmen` have multiple targets |
| `grow` | Whether this type grows naturally without promotion | Only base types (`peasants`, `tribesmen`) have this |
| `editor` | Starting proportion in the scenario editor | Does not affect gameplay — editor UI only; slaves and tribesmen have no `editor` entry and cannot be placed via the scenario editor |
| `upper = yes` | Classifies as upper class | Affects goods demand (`demand_multiply.upper`) and estate eligibility |
| `has_cap` | Subject to per-location population caps | Nobles, clergy, burghers, laborers, soldiers, and peasants have this; slaves and tribesmen do not |
| `assimilation_conversion_factor` | Speed of religious/cultural assimilation | 0.0 for tribesmen (resistant); 1.0 for peasants/slaves (fastest) |
| `literacy_impact` | Local modifiers granted proportional to literate pops of this type | Keys must be valid location modifier fields |
| `<estate>_estate` | Assigns this pop to an estate under a condition | Repeatable; condition is a trigger block; empty block `= {}` always assigns |
| `tribal_rules` | Enables tribal-specific promotion logic | Only for `tribesmen`; governs special promotion factors in tribal countries |

## Modding Notes
- **All pop types are in one file.** Adding a new type appends to `00_default.txt`. The new ID then needs demand entries in `goods/` files and estate assignments to be functional.
- **`literacy_impact`** modifiers are applied per literate pop of this type in the location. The higher the literate count, the stronger the effect. Keys must be valid local modifier identifiers.
- **`pop_percentage_impact`** modifiers scale with the pop type's percentage of total location population. Used for tribesmen: `local_distance_from_capital_speed_propagation = -1.0` reduces capital propagation speed as their share grows.
- **Estate assignment** uses repeatable `<estate>_estate = { <trigger> }` blocks. The condition is evaluated to determine which estate this pop type belongs to in a given context. An empty trigger block `= {}` always assigns to that estate.
- **`promote_to`** is one-directional and repeatable. `peasants` can promote to any upper class type. There is no promotion path back down, but population can be reduced via attrition, migration, or events. `slaves` have no `promote_to` and no `promotion_factor` — they are a completely static pop type that can only change via events or player actions.
- **`promotion_factor`** belongs to the promoting type, not the destination. When peasants (factor 0.5) promote to laborers, the peasant's factor of 0.5 governs speed. The laborer's factor of 1.5 would govern speed of any further promotion — but laborers have no `promote_to` target, so this factor is effectively unused.
- **Goods demand** for each pop type is defined in `goods/` files using `demand_add.<pop_type_id>` and `demand_multiply.<pop_type_id>`. Pop types themselves do not define which goods they consume — that lives in the good definition.
- **`grow = yes`** enables natural population growth independent of promotion. Without it, a pop type only gains pops from promotion or events. Only the two base types in vanilla (`peasants`, `tribesmen`) have this set.
- **Upper-class pops** (`upper = yes`) interact with the `demand_multiply.upper` key in good definitions — luxury goods scale demand for all upper types simultaneously via that shorthand.

## Example
`laborers` — a lower-class non-growing pop that promotes quickly and provides production efficiency when literate:

```pdx
laborers = {
    color  = pop_laborers
    editor = 1.0

    assimilation_conversion_factor = 0.5
    pop_food_consumption = 1.0
    city_graphics = 0.05

    has_cap = yes
    promotion_factor = 1.5     # promotes fastest of all non-base types
    migration_factor = 0.1

    dhimmi_estate  = { is_dhimmi = yes }
    tribes_estate  = { is_gaelic_clans = yes }
    peasants_estate = {}       # always in peasants_estate otherwise

    literacy_impact = {
        local_production_efficiency  = medium_production_efficiency_bonus
        local_max_rgo_size_modifier  = 0.1
    }
}
```

Laborers have no `grow = yes` and no `promote_to` — they only gain pops from peasant promotion and are an endpoint with no onward promotion path. The laborer's `promotion_factor` (1.5) is the highest among non-base types but is effectively unused since there is no `promote_to` target. Peasants promoting into laborers do so at the peasant's own `promotion_factor` (0.5). The `literacy_impact` block makes literate laborers improve local production.
