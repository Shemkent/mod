# Scripted GUIs
**Stage:** not applicable (v1.1.10)
**Keywords:** `scripted_gui_key`, `scope`, `is_shown`, `is_valid`, `effect`, `saved_scopes`, `notification_key`, `confirm_title`, `confirm_text`, `ai_is_valid`, `ai_chance`, `ai_frequency`

> **System type: UI/Presentation**

## Overview
Scripted GUIs are named script-defined GUI elements — typically buttons — that can be embedded in GUI layout files and driven entirely by script triggers and effects. They bridge the GUI layer and the script layer without requiring engine code changes. The `.info` file documents the full syntax, but Paradox ships **no vanilla scripted GUI definitions** in `in_game/common/scripted_guis/` in v1.1.10; all vanilla use of the system is either absent or handled through code-side GUI definitions.

## Vanilla File Locations

| File | Content |
|---|---|
| `in_game/common/scripted_guis/scripted_guis.info` | Syntax reference — no vanilla definitions |

## Block Structure

```pdxscript
gui_key = {
    scope            = <scope_type>     # entity type this GUI is evaluated in (country, character, etc.)
    is_shown         = { <triggers> }   # whether the element is visible to the player
    is_valid         = { <triggers> }   # whether the element is interactable (clickable)
    effect           = { <effects> }    # fires when the player activates the element
    saved_scopes     = { scope_a  scope_b }  # scopes available inside triggers/effects
    notification_key = <key>            # notification on activation (default: jomini_scripted_gui_confirm)
    confirm_title    = { }              # confirmation dialog title loc
    confirm_text     = { }              # confirmation dialog body loc
    ai_is_valid      = { <triggers> }   # whether AI can use this element (default false)
    ai_chance        = { }              # script value 1–100; probability AI activates it per evaluation
    ai_frequency     = { }              # named value in months between AI evaluations
}
```

## Modding Notes
- Scripted GUI elements must be referenced by `gui_key` in a `.gui` layout file using the `scripted_gui` widget type to appear in-game.
- `is_shown` / `is_valid` follow standard trigger evaluation in the declared `scope`.
- `ai_is_valid` defaults to `false` — AI will ignore the button unless explicitly enabled.
- The folder is safe to populate; the engine loads all `.txt` files. Modders wanting to add custom interactive buttons without C++ changes should use this system.
