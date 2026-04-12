# Modifiers System
**Stage:** complete
**Game version:** 1.1.10
**Keywords:** modifiers, country_modifier, location_modifier, province_modifier, auto_modifiers, scripted_modifiers, scaled, raw_modifier

> **System type: Scripted Logic**

## Overview

"Modifiers" in EU5 is an umbrella term covering several distinct systems. This guide clarifies what each system is and where to find more detail.

---

## 1. Inline Modifiers (most common)

The `modifier = {}` block appears throughout the script — in buildings, traits, laws, reforms, estate privileges, and more. These are key-value pairs that buff or debuff game stats while the parent object is active.

**Scopes:**
| Block | Applied to |
|---|---|
| `country_modifier` | The owning country |
| `location_modifier` | The specific location |
| `province_modifier` | The province |
| `modifier` | Context-dependent (building → location, trait → character, etc.) |
| `raw_modifier` | Location, not scaled by goods access (buildings only) |
| `capital_modifier` | Location, only when in the capital |
| `capital_country_modifier` | Country, only when the building is in the capital |
| `market_center_modifier` | Location, only when in a market center |

**Where they appear:** buildings, laws/policies, government reforms, estate privileges, traits, disasters, auto_modifiers, scripted effects (via `add_country_modifier`), events.

**Scaled vs flat:**
- `modifier` on buildings is scaled by `building_level × goods_access`.
- `raw_modifier` on buildings is never scaled.
- `country_modifier` on laws/reforms/privileges scales during the implementation period (0→100%).

---

## 2. auto_modifiers

**Folder:** `in_game/common/auto_modifiers/`
**Annotation:** [auto_modifiers.md](auto_modifiers.md)

Auto-modifiers are conditional modifiers that the engine evaluates continuously. They apply a modifier block to a scope (country, location, dynasty, IO) whenever a trigger condition is met — no effect needed. Think of them as "always-on" conditional modifiers.

---

## 3. scripted_modifiers

**Folder:** `in_game/common/scripted_modifiers/`

Scripted modifiers are **not** country/location modifiers. They are parameterized weight/value calculations used in `random_list` weights and similar scripted math contexts. They support `modifier`, `opinion_modifier`, `compare_modifier` sub-blocks and take `$PARAMETER$` arguments.

**Usage example:**
```
random_list = {
    10 = { my_weight_modifier = { SCALE = 0.5 } }
    5  = { }
}
```

---

## 4. static_modifiers

**Note:** The `in_game/common/static_modifiers/` folder does **not exist** in v1.1.10. Static modifier functionality (hardcoded engine modifiers) is not directly scriptable.

---

## 5. add_country_modifier / remove_country_modifier (effects)

Country modifiers can be applied dynamically via effects:
```
add_country_modifier = {
    modifier = my_modifier_key   # References a named modifier defined elsewhere
    years = 10                   # Duration; -1 = permanent
    mode = add_and_extend        # Optional: add_and_extend extends if already present
}
remove_country_modifier = my_modifier_key
```
Named modifiers referenced by `add_country_modifier` are standalone modifier blocks defined inline in events, on_actions, or scripted effects — they are not defined in a dedicated folder. The modifier block sits alongside the effect code that first applies it.

---

## Summary Table

| System | Folder | Purpose |
|---|---|---|
| Inline modifiers | (everywhere) | Buff/debuff via key-value pairs in any script block |
| auto_modifiers | `auto_modifiers/` | Continuous conditional modifiers, no effect needed |
| scripted_modifiers | `scripted_modifiers/` | Weight/value calculations for random lists; NOT country modifiers |
| static_modifiers | (absent in v1.1.10) | Hardcoded engine modifiers |
| add_country_modifier | (effect) | Dynamically apply/remove named modifiers |

---

## Example

The most common modifier context is a `country_modifier` block inside a law or government reform. The modifier applies to the owning country for as long as the law is active:

```
my_law = {
    ...
    country_modifier = {
        army_morale = 0.05
        global_tax_modifier = -0.1
        stability_cost_modifier = 0.15
    }
    ...
}
```

For system-specific modifier keys (which stats are valid in each context) see the individual system annotations listed in Cross-References below.

---

## Cross-References

- Building modifiers: [buildings.md](buildings.md)
- Law/reform/privilege modifiers: [laws.md](laws.md), [government_reforms.md](government_reforms.md), [estates.md](estates.md)
- Trait modifiers: [traits.md](traits.md)
- auto_modifiers: [auto_modifiers.md](auto_modifiers.md)
