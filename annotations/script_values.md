# Script Values
**Stage:** complete
**Keywords:** `script_value_name`, `add`, `subtract`, `multiply`, `divide`, `modulo`, `value`, `max`, `min`, `round`, `ceiling`, `floor`, `if`, `fixed_range`, `integer_range`

> **System type: Scripted Logic**

## Overview
Script values are named numbers (or formulas that resolve to numbers) referenceable anywhere in script that accepts a numeric argument. They come in two forms: **static values** (a bare constant, evaluated once at load) and **formulas** (a calculation block evaluated fresh each time at the current scope). Formulas support arithmetic, conditionals, randomness, scope traversal, and list aggregation. Any formula can also be written inline — there is no requirement to name it first. This system is the backbone of all parameterizable math in EU5 script.

## Vanilla File Locations

| File | Content |
|---|---|
| `in_game/common/script_values/_script_values.info` | Authoritative syntax reference |
| `in_game/common/script_values/define_values.txt` | Constants mirroring defines (ABDICATE_PRESTIGE_HIT, HIGH_WAR_EXHAUSTION, etc.) |
| `in_game/common/script_values/diplomatic_values.txt` | Diplomacy thresholds and costs |
| `in_game/common/script_values/research_values.txt` | Advance/institution delta helpers |
| `in_game/common/script_values/unit_values.txt` | Unit combat parameters |
| `in_game/common/script_values/monthly_income.txt` | Income formula helpers |
| `in_game/common/script_values/building_caps.txt` | Building count limits |
| … *(18 content files total)* | Domain-specific constants and formulas |
| `main_menu/common/script_values/default_values.txt` | Main-menu scope constants |

## Block Structure

**Static value** (constant):
```pdxscript
value_name = 50          # integer
value_name = 0.25        # fixed-point decimal
```

**Formula** (evaluated at scope):
```pdxscript
value_name = {
    value    = <number|script_value|scope.property>  # set absolute value
    add      = ...   # add to running total
    subtract = ...
    multiply = ...
    divide   = ...   # avoid divide-by-zero
    modulo   = ...
    max      = ...   # clamp upper bound
    min      = ...   # clamp lower bound
    round    = yes   # round to nearest integer
    ceiling  = yes   # round up
    floor    = yes   # round down

    if = {
        limit = { <triggers> }
        add = 5
    }
    else_if = { limit = { ... }  add = 3 }
    else    = { add = 1 }

    fixed_range   = { min = 1.0  max = 5.0 }   # random decimal
    integer_range = { min = 1    max = 5   }    # random integer
}
```

**Range shorthand** (used at the call site, not in the definition):
```pdxscript
# In an effect that accepts a script value:
add_gold = { 10 50 }                         # picks random int in [10, 50]
add_gold = { lower_sv upper_sv }             # resolves both named values first
# Note: this is call-site syntax; you cannot use inline formulas inside a range shorthand
```

**List aggregation** (sum over scope members):
```pdxscript
add_gold = {
    every_child = { add = 1 }   # adds 1 per child
}
```

**Inline formula** (no named definition needed):
```pdxscript
add_gold = {
    value = gold
    multiply = 0.5
}
```

**Scope chaining** (call a named formula on another scope):
```pdxscript
add_gold = { value = mother.example_age }   # resolves example_age in mother scope
```

## Key Fields Reference

| Field | Purpose | Notes |
|---|---|---|
| `value` | Set the running total to a number | Overwrites — use `add` to accumulate; if `value` is omitted entirely, the running total starts at **0** |
| `add` / `subtract` | Increment or decrement | Accepts number, script value, or scope property |
| `multiply` / `divide` | Scale the running total | Operations execute **in order top-to-bottom** |
| `modulo` | Remainder of division | Useful for cycling |
| `max` / `min` | Clamp the running total | Applied at the point they appear in execution order |
| `round` / `ceiling` / `floor` | Rounding | Applied at the point they appear |
| `if` / `else_if` / `else` | Conditional math | `limit` uses standard trigger syntax |
| `fixed_range` | Random decimal in [min, max] | Cannot be inlined inside a range shorthand |
| `integer_range` | Random integer in [min, max] | Same restriction |

## Modding Notes
- **Execution is ordered**: `value = 5  multiply = 2  max = 8  add = 2` → 5×2=10, clamped to 8, +2 = **10**, not 12. Position of `max`/`min` matters.
- **Formulas are evaluated on every use** — keep complex formulas light; static constants have no runtime cost.
- **True/false values**: a bare static constant (e.g. `my_flag = 1`) can serve as a yes/no value in some triggers. A formula block (`my_flag = { add = ... }`) always resolves to a number and cannot be used as a yes/no flag — use a `scripted_trigger` instead.
- **Inline formulas**: write the `{ add = ... }` block directly in any field that accepts a script value; no named definition required.
- **File organisation**: files are arbitrary — split by domain for maintainability. All `.txt` files in the folder are loaded; duplicate names are last-write wins.
- **Scope availability**: formulas resolve at the calling scope; use scope links (`father.`, `capital.`, `scope:x.`) to traverse.
- **main_menu scope**: `main_menu/common/script_values/default_values.txt` defines constants for achievement/scenario evaluation where in-game scopes are unavailable.

## Example

```pdxscript
# Static constant
ABDICATE_PRESTIGE_HIT = -50

# Formula: research gap between two countries
research_delta = {
    value    = scope:actor.num_of_advances_researched
    subtract = num_of_advances_researched
    min      = 0
}
# Usage: add_prestige = research_delta
#         add_prestige = { value = root.research_delta  multiply = 0.5 }

# Inline formula (no separate definition)
add_gold = {
    value    = gold
    multiply = 0.5
    floor    = yes
}
```
