# Customizable Localization
**Stage:** complete
**Keywords:** `function_key`, `type`, `text`, `trigger`, `localization_key`, `fallback`

> **System type: UI/Presentation**

## Overview
Customizable localization defines named inline functions callable from localisation strings via `[FunctionKey()]`. Each function evaluates a prioritised list of `text` entries and returns the localisation key of the first entry whose trigger passes. A `fallback = yes` entry acts as the unconditional default. This is the mechanism for dynamic UI strings that change based on game state — country-specific names for chivalric orders, ruler titles, event text variations, and more.

## Vanilla File Locations

| File | Content |
|---|---|
| `in_game/common/customizable_localization/00_customizable_localization.txt` | Core functions: GetOrderOfChivalry, GetOrderOfChivalryBonusTooltip, etc. |
| `in_game/common/customizable_localization/01_customizable_event_loc.txt` | Event-context dynamic strings |
| `in_game/common/customizable_localization/character_title.txt` | Dynamic character title strings |
| `in_game/common/customizable_localization/character_address.txt` | Dynamic character address forms |
| `in_game/common/customizable_localization/countries.txt` | Country-specific dynamic names |
| `in_game/common/customizable_localization/common_strings.txt` | Shared generic dynamic strings |
| … *(25 files total)* | Grouped by domain |

## Block Structure

```pdxscript
FunctionKey = {
    type = country       # scope type the function is evaluated in
                         # (country, character, province, etc.)
    text = {
        trigger = {      # conditions under which this text entry is used
            <triggers>
        }
        localization_key = MY_LOC_KEY    # the loc key returned if trigger passes
    }
    text = {
        trigger = { ... }
        localization_key = ANOTHER_LOC_KEY
    }
    text = {
        fallback = yes               # no trigger; always matches — must be last
        localization_key = DEFAULT_LOC_KEY
    }
}
```

Entries are evaluated top to bottom; the first passing `trigger` wins. `fallback = yes` replaces `trigger` in the final entry. If no entry matches and no fallback exists, the raw function key is displayed.

## Key Fields Reference

| Field | Purpose | Notes |
|---|---|---|
| `type` | Scope of evaluation | Determines which triggers and scope links are valid inside `trigger` blocks |
| `text` | A candidate text entry | Contains one `trigger` and one `localization_key` |
| `trigger` | Condition for this entry | Standard trigger block in the declared `type` scope |
| `localization_key` | The loc key returned when this entry matches | Must be defined in a `.yml` file |
| `fallback` | Unconditional match | `yes` — no trigger required; place last |

## Modding Notes
- Function keys are **case-sensitive** in localisation strings: `[GetOrderOfChivalry()]` must match the block name exactly.
- Evaluation is **top-to-bottom, first-match**: order your `text` entries from most specific to most general, with `fallback` last.
- The `type` field determines which scope the triggers run in. `type = country` is the most common; use `type = character` for ruler/character-context strings.
- Localisation keys returned by `localization_key` must exist in `.yml` — they can themselves contain further `[Function()]` calls, enabling chaining.
- Adding a new `text` entry via a mod file that redefines the same function key **replaces** the entire function — there is no merge. To extend a vanilla function, copy the whole block and add your entry before the fallback.
- `type` is required and must be a valid scope type. Omitting it or using an invalid value causes triggers to evaluate in the wrong scope context, silently producing incorrect results — always declare it explicitly.
- File names are arbitrary; all `.txt` files in the folder are loaded.

## Example

```pdxscript
# Returns the name of the currently held chivalric order, or a generic fallback
GetOrderOfChivalry = {
    type = country
    text = {
        trigger          = { has_policy = order_of_saint_george_policy }
        localization_key = order_of_saint_george_policy
    }
    text = {
        trigger          = { has_policy = order_of_the_ermine_policy }
        localization_key = order_of_the_ermine_policy
    }
    text = {
        fallback         = yes
        localization_key = order_of_chivalry_law     # generic name when no specific order held
    }
}
# Used in localisation: "Renew the [GetOrderOfChivalry()]"
```
