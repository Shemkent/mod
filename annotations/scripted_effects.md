# Scripted Effects
**Stage:** complete
**Game version:** 1.1.10
**Keywords:** scripted_effects, reusable effects, arguments, $variable$, custom_description, hidden_effect

> **System type: Scripted Logic**

## Overview

Scripted effects are named, reusable effect blocks. They allow complex or repeated effect logic to be defined once and called anywhere in the script with `my_effect = yes` or `my_effect = { arg = value }`. They support parameterized arguments and scope references.

---

## File Locations

| Path | Purpose |
|---|---|
| `in_game/common/scripted_effects/` | All scripted effect definitions |
| `in_game/common/scripted_effects/readme.txt` | Format reference |

### File organisation (by scope/theme)

| File | Scope/theme |
|---|---|
| `country_effects.txt` | Country-scoped effects (government, economy, characters) |
| `character_effects.txt` | Character-scoped effects |
| `location_effects.txt` | Location-scoped effects |
| `unit_effects.txt` | Army/navy unit effects |
| `religious_effects.txt` | Religion/conversion effects |
| `global_effects.txt` | Global/multi-scope effects |
| `on_action_effects.txt` | Effects reused in on_action callbacks |
| `prisoner_effects.txt` | Prisoner handling effects |
| `situation_effects.txt` | Situation-related effects |
| `international_organization_effects.txt` | IO effects |
| `country_gold_effects.txt` | Gold income/expense effects |
| `00_event_illustration_effects.txt` | Event illustration helper effects |

---

## Block Structure

### Simple effect
```
my_effect = {
    add_prestige = 10
    add_stability = 1
}
```
**Usage:** `my_effect = yes`

### Parameterized effect
```
my_argument_effect = {
    add_prestige = $value$
}
```
**Usage:** `my_argument_effect = { value = 12 }`

### Scoped argument effect
```
my_scoped_effect = {
    $target$ = {
        add_prestige = $value$
    }
}
```
**Usage:** `my_scoped_effect = { target = scope:my_scope  value = 13 }`

### Type-switched via argument concatenation
```
my_effect_hello = { add_stability = 5 }
my_effect_goodbye = { add_stability = -5 }
my_effect = { my_effect_$type$ = yes }
```
**Usage:** `my_effect = { type = hello }`

---

## Key Fields Reference

| Field / Syntax | Purpose | Key constraint |
|---|---|---|
| `effect_name = yes` | Argument-free call syntax — invokes the effect with no parameters | Effect body must not reference any `$variable$` tokens |
| `effect_name = { key = value }` | Parameterized call syntax — passes one or more named arguments | All `$key$` tokens in the effect body must be supplied |
| `$variable$` | Placeholder substituted as plain text at parse time | Not typed; the engine performs string substitution before parsing the result |
| `hidden_effect` | Wraps inner effects and suppresses their individual tooltip entries | All inner effects are hidden from the UI; pair with `custom_tooltip` |
| `custom_tooltip` | Inserts a single clean tooltip entry replacing the hidden content | Takes a localisation key; use alongside `hidden_effect` |
| `custom_description` | Wrapper block that replaces inner effect tooltips with a single description | `text` key must differ from the enclosing effect's own key (see Modding Notes) |

---

## Examples

### 1. Reusable character creation
```
create_newborn_ruler_child = {
    custom_tooltip = new_child_for_ruler
    hidden_effect = {
        create_character = {
            dynasty = root.ruler.dynasty
            culture = root.ruler.culture
            religion = root.ruler.religion
            father = root.ruler
            age = 0
        }
    }
}
```
`hidden_effect` suppresses tooltip entries for the inner effects; `custom_tooltip` provides a single clean description.

### 2. Parameterized advance unlock
```
unlock_advance_effect = {
    custom_description = {
        text = unlock_advance_effect   # vanilla exception: effect key and description text key match here;
                                       # note the engine warning in Modding Notes still applies in general
        value = advance_type:$type$
        set_variable = {
            name = unlocked_advance_$type$
            value = yes
        }
    }
}
```
**Usage:** `unlock_advance_effect = { type = castle_advance }`

---

## Modding Notes

- All `.txt` files in `scripted_effects/` are loaded — add new files as needed.
- Arguments (`$arg$`) are text replacements, not typed variables. The engine substitutes the string at parse time.
- Prefer `hidden_effect` + `custom_tooltip` for complex effects with many sub-steps that should show as one tooltip entry.
- Scripted effects are called like any other effect: `my_effect = yes` (no args) or `my_effect = { key = value }` (with args).
- Do **not** use the same key for both the effect and its `custom_description` — the engine cannot distinguish them. Use a distinct `text` key: `custom_description = { text = my_effect_text ... }`, not `text = my_effect`. The vanilla `unlock_advance_effect` is a known exception where matching keys are used.
