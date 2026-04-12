# Subject Military Stances
**Stage:** complete
**Keywords:** `subject_military_stances`, `is_default`, `chase_unit_*_priority`, `defend_*_priority`, `carpet_siege_*_priority`

> **System type: AI**
> **Cluster:** Military & War

## Overview
Subject military stances are named sets of AI priority weights that govern how a subject country's military behaves when directed by its overlord. Each stance is a flat list of float priorities covering unit movement, engagement, sieging, logistics, and recruitment. The overlord can assign a stance to each subject, tuning whether a vassal hunts enemies aggressively, defends its own lands, or stays passive. One stance must be marked `is_default`. Five vanilla stances cover the main archetypes: `normal_military_stance` (default), `aggressive_military_stance`, `supportive_military_stance`, `passive_military_stance`, and `defensive_military_stance`.

## Vanilla File Locations
`in_game/common/subject_military_stances/` — 2 files:
- `readme.txt` — field reference
- `00_default.txt` — vanilla stances

## Block Structure

```pdx
my_stance = {
    is_default = yes                              # [optional] fallback stance if none assigned

    # Movement & engagement
    chase_unit_own_location_priority = 10        # chase enemies in own (+ subject/overlord) territory
    chase_unit_overlord_location_priority = 10
    chase_unit_subject_location_priority = 10
    chase_unit_friendly_location_priority = 8    # chase in other allied territory
    chase_unit_neutral_location_priority = 8
    chase_unit_enemy_location_priority = 8
    chase_navy_priority = 5
    hunt_army_priority = 8
    hunt_navy_priority = 8

    # Defence
    defend_own_territory_priority = 10
    defend_ally_territory_priority = 5

    # Stacking
    merge_units_priority = 10

    # Piracy & repatriation
    hunt_pirates_priority = 0
    repatriate_ships_priority = 10
    repatriate_troops_priority = 10

    # Support
    support_armies_priority = 5
    support_sieges_priority = 5

    # Recruitment
    maintain_military_levels_priority = 4

    # Sieging
    carpet_siege_own_locations_attacking_priority = 2
    carpet_siege_own_locations_defending_priority = 4
    carpet_siege_enemy_locations_priority = 4

    # Naval
    blockade_port_priority = 2
    blockade_strait_priority = 3

    # Order
    suppress_rebel_priority = 1

    # Logistics
    support_sieges_priority = 5             # priority for supporting ongoing sieges

    army_logistics_priority = 2
    navy_logistics_priority = 2
}
```

## Key Fields Reference

| Field | Purpose | Notes |
|---|---|---|
| `is_default` | Fallback stance | Exactly one stance must have this |
| `chase_unit_*_priority` | Urgency of chasing enemies by location ownership | Higher = more aggressive pursuit |
| `hunt_army/navy_priority` | Active seek-and-destroy priority | |
| `defend_own/ally_territory_priority` | Defensive patrol priority | Set to 0 in aggressive stance |
| `merge_units_priority` | Stack consolidation | High = units prefer to clump |
| `support_sieges_priority` | Priority for joining and supporting ongoing sieges | Absent from `normal_military_stance`; omitted field defaults to 0 |
| `carpet_siege_*_priority` | Sieging behaviour by context | Attacking/defending wars differ |
| `blockade_strait_priority` | Only active when side has naval supremacy | |
| `maintain_military_levels_priority` | Recruitment urgency | |

## Modding Notes
- All values are relative float weights; 0 disables that behaviour entirely, 10 is high priority.
- The five vanilla stances (`normal_military_stance`, `aggressive_military_stance`, `supportive_military_stance`, `passive_military_stance`, `defensive_military_stance`) cover the main archetypes; new stances can specialise further (e.g. pirate-hunters, garrison-only vassals).
- `support_sieges_priority` is absent from `normal_military_stance` but present in `supportive_military_stance` — omitting a field sets it to 0.
- Subject stance assignment is done from subject_types or via scripted effects.
- Cross-references: `subject_types` (stances are assigned per subject type or per-country override).

## Example

```pdx
# Abbreviated — omitted fields default to 0
aggressive_military_stance = {
    chase_unit_own_location_priority = 0
    chase_unit_neutral_location_priority = 7
    chase_unit_enemy_location_priority = 8
    hunt_army_priority = 8
    defend_own_territory_priority = 0
    defend_ally_territory_priority = 0
    merge_units_priority = 10
    support_sieges_priority = 0
    carpet_siege_enemy_locations_priority = 4
    # ... remaining fields at vanilla defaults
}
```
*Aggressive stance: subject ignores its own territory defence and focuses on chasing and sieging enemies. Own/overlord/subject/friendly territory chase priorities are all 0 (not shown).*
