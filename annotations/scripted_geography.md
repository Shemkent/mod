# Scripted Geography
**Stage:** complete
**Keywords:** `scripted_geography`, `location`, `province_definition`, `area`, `region`, `sub_continent`, `continent`, `is_in_scripted_geography`, `has_presence_in`, `every_location_in_scripted_geography`, `any_location_in_scripted_geography`, `every_area_in_scripted_geography`

> **System type: Data/Reference**

## Overview
Scripted geographies are named, reusable groupings of geographic entities — mixing locations, province definitions, areas, regions, subcontinents, and continents in any combination. Once defined, a scripted geography can be referenced via `scripted_geography:key` to iterate all locations or areas inside it, test membership, or check country presence. They are the primary tool for defining custom geographic regions that span non-contiguous areas or that cross standard hierarchy boundaries (e.g. a canal route, a mission target zone).

## Vanilla File Locations

| File | Content |
|---|---|
| `in_game/common/scripted_geography/00_event_scripted_geography.txt` | Event-specific groupings: canal routes, expedition zones, etc. |
| `in_game/common/scripted_geography/01_europe.txt` | Named regional groupings for Europe (scotland_geography, england_geography, netherlands_geography, etc.) |

## Block Structure

```pdxscript
geography_key = {
    location          = { location_a  location_b }     # individual locations
    province_definition = { sea_province_a }           # province definitions (sea zones, etc.)
    area              = { north_borneo_area  south_borneo_area }
    region            = { my_region }
    sub_continent     = { my_subcontinent }
    continent         = { continent:europe }
}
```

All member types are optional; mix freely. Members are resolved at load — areas expand to their constituent locations at access time.

## Key Fields Reference

| Field | Purpose | Notes |
|---|---|---|
| `location` | Include specific locations by key | Individual map locations |
| `province_definition` | Include province definitions (sea zones, straits) | Used where no named location exists |
| `area` | Include all locations in an area | Expands transitively |
| `region` | Include all locations in a region | Expands transitively |
| `sub_continent` | Include all locations in a subcontinent | Expands transitively |
| `continent` | Include all locations in a continent | Broadest scope; entries use the `continent:` namespace prefix (e.g. `continent:europe`), unlike other member types which use bare keys |

## Script Integration

**Iterate locations:**
```pdxscript
scripted_geography:geography_key = {
    every_location_in_scripted_geography = { <effects/triggers> }
    any_location_in_scripted_geography   = { <triggers> }
}
```

**Iterate areas:**
```pdxscript
scripted_geography:geography_key = {
    every_area_in_scripted_geography = { }   # only areas directly listed or contained in listed regions/etc.
}
```

**Iterate present countries:**
```pdxscript
scripted_geography:geography_key = {
    every_present_country = { }   # countries with at least one location in this geography
}
```

**Membership test:**
```pdxscript
scope:target_location = { is_in_scripted_geography = scripted_geography:geography_key }
# Also works on: province_definition, area, region, sub_continent, continent
```

**Country presence test:**
```pdxscript
scope:target_country = { has_presence_in = scripted_geography:geography_key }
```

**Localised display:**
```
[ShowScriptedGeographyName( Arg0 )]   # in localisation strings; Arg0 = the scripted_geography scope
```
Requires a localisation key matching the geography key: `geography_key: "Display Name"`

## Modding Notes
- Scripted geographies are purely additive — modders can add new `.txt` files with new keys without touching vanilla files.
- Duplicate keys: last-loaded definition wins; there is no merge. Namespace your keys to avoid collisions.
- Member lists are resolved at load — adding a location to an area after the geography is defined does not retroactively include it; reloading is required.
- Sea zones and straits use `province_definition` rather than `location` since they have no named location entry.
- Localisation is optional; omitting it means `[ShowScriptedGeographyName]` will display the raw key.

## Example

```pdxscript
# Custom geography grouping two areas and a specific canal location
grand_canal_geography = {
    location = { hangzhou  jiangdu  xuzhou }
    area = {
        north_china_plain_area
    }
}

# Usage in a mission trigger
scope:target_location = {
    is_in_scripted_geography = scripted_geography:grand_canal_geography
}
```
