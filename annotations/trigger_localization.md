# Trigger Localization
**Stage:** complete
**Keywords:** `trigger_key`, `global`, `first`, `third`, `global_not`, `first_not`, `third_not`, `_trigger_localization.info`, `$COMPARATOR$`, `$NUM$`, `any_trigger`, `count`, `percent`

> **System type: UI/Presentation**

## Overview
Trigger localization maps trigger names to localisation keys the engine uses when generating human-readable tooltip descriptions for conditions. Each trigger can declare up to three pronoun variants (global, first, third) in positive and negative forms. The system handles value-comparison operators, `any` trigger count/percent variants, scope-comparison triggers, and fallback chaining. Like effect localization, this system contains no gameplay logic — it is purely tooltip text routing. The `.info` file is the most comprehensive documentation in the codebase for this system.

## Vanilla File Locations

| File | Content |
|---|---|
| `in_game/common/trigger_localization/_trigger_localization.info` | Authoritative syntax reference |
| `in_game/common/trigger_localization/common_triggers.txt` | Core boolean triggers (is_adult, is_married, etc.) |
| `in_game/common/trigger_localization/country_triggers.txt` | Country-scope trigger tooltip keys |
| `in_game/common/trigger_localization/character_triggers.txt` | Character-scope trigger tooltip keys |
| `in_game/common/trigger_localization/area_triggers.txt` | Area-scope trigger tooltip keys |
| … *(41 files total)* | Grouped by domain |

## Block Structure

**Basic trigger:**
```pdxscript
trigger_key = {
    global = LOC_KEY           # generic / no pronoun ("Is an adult")
    first  = FIRST_LOC_KEY     # first person ("I am an adult")
    third  = THIRD_LOC_KEY     # third person ("[Name] is an adult")

    # Negative overrides (used inside NOT = { } context):
    global_not = NOT_LOC_KEY
    first_not  = NOT_FIRST_LOC_KEY
    third_not  = NOT_THIRD_LOC_KEY
}
```

Fallback: if a requested variant is absent, the engine uses the **last defined** variant. Defining only `global` covers all pronoun contexts.

**Value comparison trigger** (`dread >= 50`):
```pdxscript
trigger_key = {
    third = THEIR_DREAD_TRIGGER    # uses $COMPARATOR$ and $NUM$ substitution tokens
}
# Specialisations by operator (fall back to generic if absent):
trigger_key_equal            = { third = LOC_EQUAL }
trigger_key_greater_than     = { third = LOC_GT }
trigger_key_greater_or_equal = { third = LOC_GTE }
trigger_key_less_than        = { third = LOC_LT }
trigger_key_less_or_equal    = { third = LOC_LTE }
```

**Any trigger with count/percent** (`any_child = { count >= 5 ... }`):
```pdxscript
any_child = {
    third = ANY_CHILD_THIRD                       # basic "any of"
}
any_child_count_greater_or_equal = {
    third = COUNT_GT_EQ_CHILDREN_THIRD            # "$COUNT$ of [Name]'s children"
}
any_child_all = {
    third = ALL_CHILDREN_THIRD                    # "All of [Name]'s children"
}
```

**NOT inversion rules for `any` triggers:**
```
NOT = { any_child = { ... } }        → resolved as any_child with _all suffix, conditions inverted
NOT = { any_child = { count >= 3 } } → resolved as any_child_count_less_than
```

**Scope comparison trigger** (`betrothed = scope:target`):
```pdxscript
betrothed_equal = {
    third = BETROTHED_EQUAL_THIRD   # subject = LHS before last dot; object = RHS
}
```

## Key Fields Reference

| Field | Purpose | Notes |
|---|---|---|
| `global` | Generic tooltip text | No pronoun; fallback base |
| `first` | First-person text | "I am…" |
| `third` | Third-person text | "[Name] is…"; most commonly needed |
| `*_not` | Negative form | Used inside `NOT = {}` context; if absent, falls back to positive form with negation phrasing from the loc string |
| `$COMPARATOR$` | Operator token | Substituted with "greater than", "equal to", etc. by the engine in value comparison triggers |
| `$NUM$` | Value token | Substituted with the compared number in value comparison triggers |
| `$COUNT$` | Count token | Used in `any_<key>_count_<operator>` variant loc strings |
| `$PERCENT$` | Percent token | Used in `any_<key>_percent_<operator>` variant loc strings; suffix pattern is `any_child_percent_greater_or_equal` etc., matching the count pattern with `percent` replacing `count` |

## Modding Notes
- Fallback chain: if `first` is not defined, the engine uses the last defined variant — defining `global` alone is sufficient for most triggers.
- Positive and negative forms use **two separate mechanisms**: (1) the script field names are always `global_not`, `first_not`, `third_not` — these let you point to a different `.yml` key for the negated form; (2) the `.yml` keys themselves can be named anything, but the vanilla convention is a `NOT_` prefix (e.g. `NOT_IS_ADULT_TRIGGER`). If you omit a `*_not` field, the engine falls back to the last defined variant — it does not automatically negate the positive form.
- Value comparison specialisations are suffix-appended to the trigger key name — they are entirely separate script entries, not sub-fields of the base entry.
- `any` trigger count/percent variants follow the same suffix pattern: `any_child_count_greater_or_equal` is a standalone entry.
- The NOT inversion rules for `any` triggers are automatic — the engine rewrites the trigger in the display layer; you do not write the inverted form in script.
- Custom triggers added by mods need entries here for tooltips to display cleanly; without them the raw trigger key is shown.

## Example

```pdxscript
# Simple boolean trigger with all pronoun variants
is_adult = {
    global = IS_ADULT_TRIGGER         # loc: "Is an adult"
    first  = I_AM_ADULT_TRIGGER       # loc: "I am an adult"
    third  = THEY_ARE_ADULT_TRIGGER   # loc: "[Character.GetName] is an adult"
    # Negative .yml keys (NOT_ prefix convention): NOT_IS_ADULT_TRIGGER, NOT_I_AM_ADULT_TRIGGER, NOT_THEY_ARE_ADULT_TRIGGER
    # Script negative overrides (if you want different keys):
    # global_not = MY_CUSTOM_NOT_KEY
}

# Value comparison — generic + operator specialisation
stability = {
    third = STABILITY_TRIGGER              # "[Country]'s Stability is $COMPARATOR$ $NUM$"
}
stability_greater_or_equal = {
    third = STABILITY_GTE_TRIGGER          # "[Country]'s Stability is at least $NUM$"
}

# Any trigger with count variant
any_location = {
    third = ANY_LOCATION_THIRD             # "Any of [Country]'s Locations:"
}
any_location_count_greater_or_equal = {
    third = LOCATION_COUNT_GTE_THIRD       # "$COUNT$ of [Country]'s Locations"
}
any_location_percent_greater_or_equal = {
    third = LOCATION_PERCENT_GTE_THIRD     # "$PERCENT$% of [Country]'s Locations"
}

# Scope comparison trigger (capital = scope:target)
capital_equal = {
    third = CAPITAL_EQUAL_THIRD            # "[Country]'s Capital is [target]"
}
```
