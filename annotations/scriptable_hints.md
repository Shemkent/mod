# Scriptable Hints
**Stage:** complete
**Keywords:** `hint_key`, `priority`, `sort_priority`, `hide`, `icon`, `player_playstyle`, `style`, `hint_tag`

> **System type: UI/Presentation**

## Overview
Scriptable hints are context-sensitive advisor tips shown to the player when specific game conditions are met. Each hint defines a priority trigger (when it becomes relevant), a sort order, an optional hide condition, and one or more playstyle-specific localisation tag variants so that the advice text can be tailored to how the player tends to play (diplomatic, military, administrative). The system is UI-only — hints fire no effects.

## Vanilla File Locations

| File | Content |
|---|---|
| `in_game/common/scriptable_hints/scripted_hints.txt` | All vanilla hint definitions (deficit warnings, capacity alerts, etc.) |
| `in_game/common/scriptable_hints/____info.txt` | Syntax reference |

## Block Structure

```pdxscript
hint_key = {
    priority = {               # script value or trigger block — hint becomes active when this passes/is > 0
        <trigger_or_value>
    }
    sort_priority = <int>      # higher = shown first in the hint list; aligns with alert severity
    hide = { <triggers> }      # suppress the hint even if priority passes (leave empty to never hide)
    icon = <texture_path>      # optional; hint icon

    player_playstyle = {       # one block per playstyle variant
        style = diplomatic     # diplomatic / military / administrative
        hint_tag = <loc_key>   # localisation key for the hint text shown to this playstyle
    }
    player_playstyle = {
        style = military
        hint_tag = <loc_key>
    }
    player_playstyle = {
        style = administrative
        hint_tag = <loc_key>
    }
}
```

## Key Fields Reference

| Field | Purpose | Notes |
|---|---|---|
| `priority` | When the hint is active | Trigger or script value; called per frame — keep triggers cheap |
| `sort_priority` | Ordering in hint panel | Higher = shown first; vanilla uses values like 25, 200 to mirror alert severity |
| `hide` | Suppress hint | Evaluated after `priority`; empty block = never hidden |
| `icon` | Hint icon texture | Optional |
| `player_playstyle` | Playstyle-specific text variant | `style` must be `diplomatic`, `military`, or `administrative`; `hint_tag` is a `.yml` localisation key |

## Modding Notes
- **Performance warning**: `priority` is evaluated per frame. Never use expensive triggers here — simple comparison triggers (`monthly_balance < 0`, `used_cultures_capacity > modifier:cultures_capacity`) only.
- Each hint should have all three `player_playstyle` blocks; if a style is missing the hint may fall back to another style or show no text.
- `hint_tag` localisation keys are independent of the `hint_key` — you must define `<hint_tag>:` entries in a `.yml` file.
- `sort_priority` has no enforced maximum; aligning it with alert severity (see `alert_descriptions` system) makes the hint panel feel consistent.
- Files are additive — modders can add new hint files without touching vanilla content.

## Example

```pdxscript
hint_in_deficit = {
    priority     = { monthly_balance < 0 }
    sort_priority = 25
    hide = {}
    player_playstyle = {
        style    = diplomatic
        hint_tag = hint_in_deficit_diplomatic
    }
    player_playstyle = {
        style    = military
        hint_tag = hint_in_deficit_military
    }
    player_playstyle = {
        style    = administrative
        hint_tag = hint_in_deficit_administrative
    }
}
```
