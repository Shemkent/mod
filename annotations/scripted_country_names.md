# Scripted Country Names
**Stage:** complete
**Keywords:** `country_name_key`, `country_trigger`, `capital_trigger`, `location_trigger`

> **System type: UI/Presentation**

## Overview
Scripted country names define conditional overrides for a country's displayed name. When all three trigger blocks pass simultaneously — country-level conditions, a capital location check, and a check against all owned locations — the game uses the localisation key matching the scripted country name key instead of the country's default name. This is the mechanism that renames Spain's colonial subjects ("New Spain", "New Granada", "Jamaica") based on their capital and territory.

## Vanilla File Locations

| File | Content |
|---|---|
| `in_game/common/scripted_country_names/00_default.txt` | All vanilla conditional country name definitions |

## Block Structure

```pdxscript
name_key = {
    country_trigger = {    # checked against the country itself
        <triggers>
    }
    capital_trigger = {    # checked against the country's capital location; must pass
        <triggers>
    }
    location_trigger = {   # checked against every owned location; all must pass
        <triggers>
    }
}
```

All three blocks are required but may be left empty (`{ }`), which always passes. The localisation key `name_key` must be defined in a `.yml` file for the name to display correctly.

## Key Fields Reference

| Field | Purpose | Notes |
|---|---|---|
| `country_trigger` | Country-scope conditions | Overlord tag, culture, cores, etc. |
| `capital_trigger` | Capital location conditions | Province definition, region, area — narrows where in the world the country must be seated |
| `location_trigger` | All-locations conditions | All owned locations must satisfy this; leave empty if no territorial constraint is needed |

## Modding Notes
- All three triggers must pass simultaneously. An empty block (`{ }`) is equivalent to `always = yes`.
- `location_trigger` applies to **every** owned location — it is not "any location". This makes it a strong territorial filter; use sparingly.
- The displayed name resolves to the localisation key equal to the block name (`name_key:`). Without that localisation entry the key string will be shown raw.
- Multiple entries can match for the same country simultaneously; the game picks one (order in file determines priority — first match wins).
- Files are loaded alphabetically; `00_default.txt` is loaded first, so modder files loaded after can override vanilla entries if they use the same key.

## Example

```pdxscript
# Rename a colonial country to "Jamaica" if its capital is in the Xaymaca province
jamaica_scripted_country_name = {
    country_trigger = { }               # no country-level restriction
    capital_trigger = {
        province_definition = province_definition:xaymaca_province
    }
    location_trigger = {
        province_definition = province_definition:xaymaca_province
    }
}
# Requires localisation: jamaica_scripted_country_name: "Jamaica"
```
