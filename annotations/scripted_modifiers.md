# Scripted Modifiers
**Stage:** not applicable (v1.1.10)
**Keywords:** `scripted_modifiers`, `modifier`, `opinion_modifier`, `compare_modifier`, `$PARAM$`

> **System type: Scripted Logic**

## Overview
Scripted modifiers are named, reusable weight-modifier blocks used in `random_list` and AI weighting contexts — they are **not** country/location modifiers. A scripted modifier bundles one or more `modifier`, `opinion_modifier`, or `compare_modifier` sub-blocks under a single callable name, with optional `$parameter$` substitution. The folder exists and the system is documented in the `.info` file, but Paradox ships **no vanilla definitions** in v1.1.10 — all vanilla weight modifiers are written inline.

## Vanilla File Locations

| File | Content |
|---|---|
| `in_game/common/scripted_modifiers/scripted_modifiers.info` | Syntax reference only — no vanilla definitions |

## Block Structure

```pdxscript
modifier_key = {
    modifier = {
        add = { value = 10  multiply = $SCALE$ }
        $CHARACTER_1$ = { has_trait = ambitious }   # trigger condition on the modified scope
        desc = MODIFIER_LOC_KEY                      # optional tooltip localisation
    }
    opinion_modifier = {
        target      = $CHARACTER_2$
        who         = $CHARACTER_1$
        multiplier  = { value = 0.25  multiply = $SCALE$ }
    }
    compare_modifier = {
        target      = $CHARACTER_1$
        value       = stress
        multiplier  = $SCALE$
    }
}
```

**Invocation:**
```pdxscript
# No parameters:
modifier_key = yes

# With parameters:
modifier_key = {
    CHARACTER_1 = root
    CHARACTER_2 = scope:target
    SCALE       = 0.5
}
```

## Modding Notes
- Scripted modifiers apply in `random_list` weight blocks and AI decision weights, not to country/location stats (use auto_modifiers or inline modifier blocks for those).
- Parameters use `$UPPER_CASE$` convention by analogy with `scripted_effects` and `scripted_triggers`.
- The folder is safe to populate — the engine loads all `.txt` files. Modders can define shared weight modifiers here to reduce duplication across event `random_list` entries.
