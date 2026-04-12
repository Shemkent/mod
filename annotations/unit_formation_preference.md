# Unit Formation Preference
**Stage:** complete
**Keywords:** `unit_formation_preference`, `left`, `center`, `right`, `reserves`, `max_frontage`, `army`, `navy`

> **System type: Gameplay**
> **Cluster:** Military & War

## Overview
Unit formation preferences define named battle-formation templates used when deploying armies or navies. Each template specifies weighted unit-type distributions for the left flank, centre, right flank, and reserves, along with a `max_frontage` multiplier per wing. The game uses the default template unless a formation has been explicitly assigned. Modders can define custom formation doctrines for culture-specific or government-specific unit arrangements.

## Vanilla File Locations
`in_game/common/unit_formation_preference/` — 3 files:
- `readme.txt` — minimal syntax hint
- `army.txt` — land formations
- `navy.txt` — naval formations

## Block Structure

```pdx
my_formation = {
    army = yes          # yes = land formation; no = naval formation

    default = yes       # [optional] fallback if no formation is assigned

    left = {
        10 = army_cavalry          # weight = unit_category
        2  = heavy.army_cavalry    # weight = quality.unit_category
        max_frontage = 1.25        # multiplier on frontage slots available in this wing
    }

    center = {
        2  = army_artillery
        20 = army_infantry
        max_frontage = 1.25
    }

    right = {
        10 = army_cavalry
        max_frontage = 1.25
    }

    reserves = {
        100 = army_auxiliary       # units that fill in behind the main line
    }
}
```

## Key Fields Reference

| Field | Purpose | Notes |
|---|---|---|
| `army` | Formation type | `yes` = land formation; `no` = naval formation |
| `default` | Fallback template | One per army (`army = yes`) and one per navy (`army = no`) should be marked default |
| `left` / `center` / `right` | Wing compositions | Weighted lists of `[quality.]unit_category` |
| `reserves` | Reserve line | Weighted list; used to fill gaps behind the main line |
| `max_frontage` | Frontage multiplier per wing | 1.0 = standard width; >1 = wider deployment |

## Modding Notes
- Unit category IDs come from `common/unit_categories/`. Quality prefixes (`light.`, `heavy.`) sub-select within a category.
- Weights are relative — `10 = army_cavalry` in left and `20 = army_infantry` in center means infantry is twice as likely to fill a center slot.
- If a required unit type is unavailable, the game falls back to the next weighted option.
- All vanilla formations use `max_frontage = 1.25`; the engine implications of values outside this range are undocumented.
- How formations are assigned to specific countries is not exposed in vanilla script files; the assignment mechanism is engine-internal or not yet hooked in v1.1.10.
- Cross-references: `unit_types`, `unit_categories`.

## Example

```pdx
cav_inf_cav = {
    army = yes

    left   = { 10 = army_cavalry   max_frontage = 1.25 }
    center = { 2 = army_artillery   20 = army_infantry  max_frontage = 1.25 }
    right  = { 10 = army_cavalry   max_frontage = 1.25 }

    reserves = { 100 = army_auxiliary }
}
```
*Classic cavalry-flank doctrine: cavalry on both wings with artillery-supported infantry in the centre.*
