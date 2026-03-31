# Alert Descriptions
**Stage:** complete
**Keywords:** `alert_key`, `title`, `texture`, `priority`, `game_concept`

> **System type: UI / Presentation**
> This file configures how alerts are *displayed* — icon, color, and tooltip title. It contains no gameplay logic. The trigger conditions that determine *when* each alert fires are defined elsewhere (engine code or scripted triggers).

## Overview
`alert_descriptions` maps named alert IDs to their UI presentation properties: a localization key for the title, an icon texture path, an optional game concept for tooltip linking, and a priority level that controls the alert's color and sort order in the UI. Modders add entries here only when creating new alert IDs that need a display definition — not to change when alerts fire.

## Vanilla File Locations

| File | Content |
|---|---|
| `in_game/common/alert_descriptions/00_default.txt` | All ~80 alert display definitions |

## Block Structure
```
alert_key = {
    title       = "ALERT_LOCALIZATION_KEY"   # Localization key for the alert title tooltip
    texture     = "alerts_icons/icon_name"   # Relative icon path (no extension)
    game_concept = "buildings"               # Optional: links tooltip to a game concept entry
    priority    = orange                     # Alert color/severity (see priority levels below)
}
```

**Priority levels** (observed, approximate severity order):
`blue` (info/tips) → `green` (opportunity) → `yellow` (minor warning) → `orange` (moderate warning) → `red` (critical) → `purple` (special, e.g. parliament in session)

## Key Fields Reference

| Field | Notes |
|---|---|
| `title` | Localization key string — must exist in localization files |
| `texture` | Icon path relative to the game's `gfx/` directory; no file extension |
| `game_concept` | Optional; links to a `game_concepts` entry for tooltip content |
| `priority` | Controls color and UI sort order; use an existing level — custom values are engine-defined |

## Modding Notes

- **Alert trigger logic is not here.** To add a new alert, you also need to define its trigger elsewhere (scripted trigger or engine hook) and add a localization entry.
- `game_concept` is optional and only needed if you want the alert tooltip to include a game concept panel.
- Texture paths use `/` separators and omit the file extension; icons live under `gfx/` in the game directory.
- Load order: `00_` prefix; mod additions should use a new file with a higher prefix. Redefining an existing key overrides it.

## Example

```
has_constructions_missing_input = {
    title        = "ALERT_CONSTRUCTIONS_MISSING_INPUT"
    game_concept = "constructions"
    texture      = "alerts_icons/constructions_missing_input"
    priority     = red
}
```
*Red priority (critical), with a game concept link so the tooltip includes a constructions explanation panel.*
