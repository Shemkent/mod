# Holy Sites
**Stage:** complete
**Keywords:** holy_site, holy_site_type, location, type, importance, religions, god, avatar, location_modifier, country_modifier, religion_modifier

> **System type: Data/Reference**

## Overview
Holy sites define specific locations that are sacred to one or more religions. Each site references a `type` (from `holy_site_types/`) that provides its modifier template, and an `importance` value (1–5) that scales those modifiers. The modifiers from the type definition are applied to the site's location, to the owning country, and/or proportionally based on whether the dominant religion and owner religion match the site's listed religions. Vanilla ships 16 holy site files (organized by religion) and 1 holy site types file with ~12 type definitions.

## Vanilla File Locations
- `in_game/common/holy_sites/` — 16 `.txt` files by religion/tradition
- `in_game/common/holy_site_types/00_holy_site_types.txt` — all site type modifier templates
- Full file list in `_file_index.csv`.

## Block Structure

### Holy Site Definition
```pdx
<site_id> = {
    location   = <location_key>         # geographic anchor (e.g. jerusalem, rome)
    type       = <holy_site_type_id>    # modifier template from holy_site_types/
    importance = <1-5>                  # scales modifiers from the type; 5 = maximum
    religions  = { <religion_id> ... }  # religions that consider this site holy (repeatable)
    god        = <god_id>               # optional associated deity (from gods/)
    avatar     = <avatar_id>            # optional associated avatar
}
```

### Holy Site Type Definition
```pdx
<site_type_id> = {
    location_modifier = {
        # Applied to the location where the holy site exists
        # Scaled by importance
        local_clergy_desired_pop = <float>
        local_unrest = <float>
        # etc.
    }
    country_modifier = {
        # Applied to the country that owns the location
        # Scaled by importance
        monthly_religious_influence = <float>
        religious_icon_power_modifier = <float>
        # etc.
    }
    religion_modifier = {
        # Applied conditionally: 50% if location's dominant religion is in site's religions list,
        # 50% if the owning country's religion is in the list
        <modifier_key> = <float>
    }
}
```

## Vanilla Site Types
| Type ID | Location effect | Country effect |
|---|---|---|
| `shrine` | Clergy desired pop, unrest reduction | — |
| `mountain` | Food, clergy pop, literacy | — |
| `temple` | Clergy pop, prosperity, control | — |
| `city` | Population capacity, institution growth, clergy pop | — |
| `episcopal_see` | — | Religious icon power |
| `orthodox_church_holy_site` | Clergy pop, prosperity, clergy literacy | Religious icon power |
| `christian_holy_site` | Clergy pop, prosperity, control | Monthly religious influence |

## Key Fields Reference
| Field | Purpose | Key constraint |
|---|---|---|
| `location` | Geographic anchor | Must match a valid location ID from the map database |
| `type` | Modifier template | Must match an ID in `holy_site_types/`; determines what bonuses the site grants |
| `importance` | Modifier scale (1–5) | Higher importance = stronger modifiers from the type; 5 is the maximum vanilla value |
| `religions` | Which faiths consider this site holy | A site can be holy to multiple religions simultaneously |
| `god` | Associated deity | Optional; links the site to a god definition for flavor/effects |
| `location_modifier` (type) | Modifiers applied to the site location | Scaled by the site's `importance` value |
| `country_modifier` (type) | Modifiers applied to the owning country | Scaled by `importance`; active when a country's religion is in the site's `religions` list |
| `religion_modifier` (type) | Conditionally split modifier | Applied in two 50% halves: one when location's dominant religion matches, one when owner's religion matches |

## Modding Notes
- **Adding a new holy site** requires: a definition in any `.txt` file in `holy_sites/`, a valid `location` ID, a `type` that exists in `holy_site_types/`, and at least one religion in `religions`. No other registration is needed — the engine picks up all sites from the folder.
- **Adding a new site type** requires: a definition in `holy_site_types/`, localization, and at least one modifier block. Any of `location_modifier`, `country_modifier`, and `religion_modifier` can be omitted.
- **`importance` scales all modifiers from the type** — it is a multiplier on the type's modifier values, not a simple tier. A type with `local_unrest = -0.05` at importance 3 grants `−0.15` unrest at the location.
- **A site can be holy to multiple religions.** Jerusalem, for example, is listed under Catholic, Orthodox, and Muslim holy site files, each with their own site definition and type. Multiple definitions for the same location are valid — each religion's entry is independent.
- **`religion_modifier`** is split 50/50 between dominant-religion match and owner-religion match. If neither matches, neither half applies. This allows sites to benefit countries whose religion isn't the dominant local faith.
- **Cross-system:** holy sites interact with the `gods/` system (`god` field), the `avatars/` system, the location system (location IDs), and religion definitions (`religions` field). Site modifiers stack with `definition_modifier` from the religion and `religious_aspects` modifiers.

## Example
`jerusalem_the_holy_city_catholic` and `christian_holy_site` type:

```pdx
# holy_sites/catholic.txt
jerusalem_the_holy_city_catholic = {
    location   = jerusalem
    type       = christian_holy_site
    importance = 5               # maximum importance — strongest modifier scaling
    religions  = { catholic bosnian_church bogomilism paulicianism }
}

# holy_site_types/00_holy_site_types.txt
christian_holy_site = {
    location_modifier = {
        local_clergy_desired_pop = 0.02
        local_monthly_prosperity = 0.001
        local_max_control        = 0.01
    }
    country_modifier = {
        monthly_religious_influence = 0.02
    }
}
```

At importance 5, `jerusalem` grants all `christian_holy_site` location modifiers at 5× scale, and the owning country gains `monthly_religious_influence = 0.10`. The site is simultaneously holy to four religions — each has its own entry in its own file, but they all resolve to the same location.
