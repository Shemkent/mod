# Effect Localization
**Stage:** complete
**Keywords:** `effect_key`, `global`, `first`, `third`, `global_past`, `first_past`, `third_past`, `neg`, `_category`, `$VALUE$`

> **System type: UI/Presentation**

## Overview
Effect localization maps effect names to localisation keys that the engine uses when generating human-readable tooltip descriptions for scripted effects. For each effect, up to six variants can be declared: three pronouns (global/no-pronoun, first-person, third-person) each in present and past tense. A `neg` flag handles cases where the effect produces a negative value that should be described positively in text (e.g. "lose 10 gold" uses a positive number for the loss). Groups are declared with `_category = { ... }` blocks (leading underscore). This system contains no gameplay logic — it is purely tooltip text routing.

## Vanilla File Locations

| File | Content |
|---|---|
| `in_game/common/effect_localization/__example.txt` | Syntax examples and category header format |
| `in_game/common/effect_localization/common_effects.txt` | Core effects: kill_character, conditional_effect, etc. |
| `in_game/common/effect_localization/character_effects.txt` | Character-scope effect tooltip keys |
| `in_game/common/effect_localization/country_effects.txt` | Country-scope effect tooltip keys |
| `in_game/common/effect_localization/building_effects.txt` | Building-scope effect tooltip keys |
| … *(31 files total)* | Grouped by domain |

## Block Structure

```pdxscript
# Category group (leading underscore creates a named group)
_category_name = {
    effect_key = {
        global      = LOC_KEY          # no pronoun / generic ("Add 5 Gold")
        global_past = PAST_LOC_KEY     # past tense generic ("Added 5 Gold")
        first       = FIRST_LOC_KEY    # first person ("I gain 5 Gold")
        first_past  = PAST_FIRST_LOC_KEY
        third       = THIRD_LOC_KEY    # third person ("[Name] gains 5 Gold")
        third_past  = PAST_THIRD_LOC_KEY
        neg         = yes              # value in neg entry will be presented as positive
    }
}

# Or flat (no category):
effect_key = {
    global = LOC_KEY
    third  = THIRD_LOC_KEY
}
```

If a requested variant is not defined, the engine falls back to the **last defined** variant in the block.

## Key Fields Reference

| Field | Purpose | Notes |
|---|---|---|
| `global` | Generic / no-pronoun variant | Fallback base; used in tooltips without a specific subject |
| `first` | First-person variant | "I gain…" |
| `third` | Third-person variant | "[Name] gains…" — supports `GetName` loc functions |
| `*_past` | Past-tense suffix for any variant | Used when describing something that already happened |
| `$VALUE$` | Value substitution token | Used in `.yml` loc strings to display the effect's numeric argument; when `neg = yes`, `$VALUE$` is shown as a positive number regardless of the underlying sign |
| `neg` | Invert value sign in text | `yes` → the displayed number is made positive; use for "lose X" effects; only affects numeric effects — does not apply to non-numeric effects |
| `_category` | Group wrapper | Leading underscore; creates a named group for organisation only — no gameplay effect |

## Modding Notes
- Only define the variants you need — the engine falls back to the last defined one. Defining only `global` covers all cases adequately.
- Localisation keys referenced here must be defined in `.yml` files in `localization/`; missing keys display as raw key strings.
- `neg = yes` means the effect value is negated before display — use this when an effect subtracts something and you want the tooltip to read "lose 5 Gold" (showing `5`, not `-5`).
- File names are arbitrary; group by domain for maintainability.
- `_category` blocks are organisational only and have no impact on key resolution.
- This system has no interaction with gameplay; removing or adding entries here only affects tooltip text.

## Example

```pdxscript
# Effect with differentiated pronoun and tense variants
add_prestige = {
    global      = ADD_PRESTIGE_EFFECT           # "Gain $VALUE$ Prestige"
    global_past = PAST_ADD_PRESTIGE_EFFECT       # "Gained $VALUE$ Prestige"
    first       = I_GAIN_PRESTIGE_EFFECT         # "I gain $VALUE$ Prestige"
    first_past  = I_GAINED_PRESTIGE_EFFECT
    third       = THEY_GAIN_PRESTIGE_EFFECT      # "[Name] gains $VALUE$ Prestige"
    third_past  = THEY_GAINED_PRESTIGE_EFFECT
}

# Effect where all pronouns collapse to the same keys (valid but less descriptive)
kill_character = {
    global      = KILL_CHARACTER_EFFECT
    global_past = PAST_KILL_CHARACTER_EFFECT
    first       = KILL_CHARACTER_EFFECT          # same key — pronoun context ignored
    first_past  = PAST_KILL_CHARACTER_EFFECT
    third       = KILL_CHARACTER_EFFECT
    third_past  = PAST_KILL_CHARACTER_EFFECT
}

# Numeric loss effect using neg — $VALUE$ shown as positive in tooltip
spend_gold = {
    global = SPEND_GOLD_EFFECT      # loc string: "Spend $VALUE$ Gold" (VALUE displayed as positive)
    neg    = yes
}
```
