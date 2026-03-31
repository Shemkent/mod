# Attribute Columns
**Stage:** complete
**Keywords:** `object_type`, `column_id`, `widget`, `width`, `fixed_height`, `is_constant_width`, `contains_select_target_button`, `single_widget_for_row`, `sort`

> **System type: UI/Presentation**
> Defines the columns shown in sortable selection lists (e.g. the country picker, the character picker). No gameplay logic — purely layout and sort configuration. Changes here require matching GUI widget definitions.

## Overview
Attribute columns configure the visible columns and sort options for every interactive list in the game where the player selects a target (country, character, province, etc.). Each file covers one object type and defines named column entries with their widget reference, width, and optional sort headers. The engine uses these definitions to construct selection UIs; mods can add columns or sort options to existing screens.

## Vanilla File Locations
`in_game/common/attribute_columns/` — ~50 files, one per object type. Numbering determines load order.

Key files: `00_defaults.txt` (shared reusable columns), `06_country.txt`, `09_character.txt`, `05_location.txt`, `11_pop.txt`.

## Block Structure
```
object_type = {                          # e.g. country, character, location
    column_id = {
        widget   = widget_name           # GUI widget key (must exist in gui files)
        width    = 150                   # Column width in pixels
        fixed_height = 32                # Optional; enables row height optimization
        is_constant_width = no           # yes (default) = fixed; no = absorbs remaining space
        contains_select_target_button = yes  # Set if widget includes the selection button
        single_widget_for_row = yes      # Set if one widget covers the full row

        sort = {                         # 0..n sort headers per column
            sort_key            = "gp"   # Unique sort key string
            sort_value = { value = great_power_score }  # For numeric sort
            # OR: sort_text = { ... }    # For text sort
            sort_order          = "Ascending"
            sort_by_tooltip_key = "SORT_BY_GP"
        }
    }
}
```

## Key Fields Reference

| Field | Purpose | Notes |
|---|---|---|
| `widget` | GUI widget reference | Must match a widget defined in `.gui` files |
| `width` / `fixed_height` | Layout dimensions | `fixed_height` improves render performance for long lists |
| `is_constant_width = no` | Fluid column | Absorbs any leftover horizontal space |
| `contains_select_target_button` | Deduplication flag | Prevents engine creating a duplicate selection button |
| `single_widget_for_row` | Full-row widget | Errors if additional columns are added when set |
| `sort` | Sort header | Multiple per column allowed; use `sort_value` (numeric) or `sort_text` (string) |

## Modding Notes
- Each `column_id` must have a matching widget in the game's GUI files — this system is tightly coupled to the GUI layer and cannot be changed independently.
- `00_defaults.txt` defines reusable column types (e.g. `left_click_cost`, `right_click_and_confirm_cost`) that can be referenced in other files.
- Adding new columns to an existing object type is safe if the widget exists. Removing or renaming columns may break screens that reference them.
- `sort_value` uses scripted value syntax; `sort_text` uses scripted text syntax. Both have access to `root` (the listed object) and interaction scopes.

## Example
```
country = {
    great_power_score = {
        widget       = country_great_power_score
        width        = 100
        fixed_height = 32
        sort = {
            sort_key            = "gp"
            sort_value          = { value = great_power_score }
            sort_by_tooltip_key = "COUNTRY_SORT_BY_GREAT_POWER_SCORE"
        }
    }
}
```
*Standard numeric sort column: fixed 100px wide, sorts by the `great_power_score` scripted value.*
