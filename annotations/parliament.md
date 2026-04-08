# Parliament
**Stage:** complete
**Keywords:** parliament_types, parliament_agendas, parliament_issues, assembly, council, estate_parliament, agenda, issue, on_accept, on_debate_passed, modifier_when_in_debate

> **System type: Gameplay**

## Overview

Parliament is the legislative/estate negotiation system. It has three layers: a **parliament type** (the structural form — assembly, council, estate parliament, or none), **parliament agendas** (periodic demands that estates bring to the floor), and **parliament issues** (the specific policy motions that are debated and voted on). Countries with `has_a_parliamentary_system = yes` modifier can enact parliament agendas; IOs (like the HRE) have their own parliament type and agendas. Estates and IO special-status members participate in agendas; debate outcomes grant or deny modifiers.

## Vanilla File Locations

| Path | Purpose |
|---|---|
| `in_game/common/parliament_types/readme.txt` | Parliament type fields |
| `in_game/common/parliament_types/00_default.txt` | Country parliament types (no_parliament, assembly, council, estate_parliament) |
| `in_game/common/parliament_types/01_international_organization.txt` | IO parliament types |
| `in_game/common/parliament_agendas/readme.txt` | Agenda field reference |
| `in_game/common/parliament_agendas/00_common.txt` | Cross-estate agendas (add privilege, etc.) |
| `in_game/common/parliament_agendas/01_societal_values.txt` | Societal-value-pushing agendas |
| `in_game/common/parliament_agendas/02_institutions.txt` | Institution-spreading agendas |
| `in_game/common/parliament_agendas/03_buildings.txt` | Construction agendas |
| `in_game/common/parliament_agendas/10_hre_agendas.txt` | HRE-specific IO agendas |
| `in_game/common/parliament_issues/readme.txt` | Issue field reference |
| `in_game/common/parliament_issues/[7 files]` | Per-estate and IO issue definitions |

Full file list in `_file_index.csv`.

## Block Structure

### Parliament type
```
parliament_type_key = {
    type = country/international_organization   # Required.

    potential = { <triggers> }     # Whether this type is visible (root = country/IO).
    allow     = { <triggers> }     # Whether it can be adopted.
    locked    = { <triggers> }     # Whether it is locked.

    modifier  = { <modifiers> }    # Applied to the country/IO while this type is active.
}
```

### Parliament agenda
```
agenda_key = {
    type    = country/international_organization
    estate  = <estate_type_key>           # Which estate brings this agenda (country only); multiple allowed.
    special_status = <special_status_key> # IO membership status (IO only); multiple allowed.
    importance = <scripted_value>         # Weight impact on parliament issue; default 1.

    potential  = { <triggers> }   # When this agenda can appear.
    allow      = { <triggers> }   # When this agenda is enabled.
    chance     = { <scripted_value_block> } # Relative weight for selection.

    on_accept  = { <effects> }    # Effects when the agenda is accepted.
    can_bribe  = { <triggers> }   # When bribing this agenda is allowed.
    on_bribe   = { <effects> }    # Effects on the briber when estate is bribed.

    ai_will_do = { <scripted_maths> }  # AI weight for accepting/rejecting.
}
```

### Parliament issue
```
issue_key = {
    type    = country/international_organization
    estate  = <estate_type_key>       # Estate this issue is tied to (country only).
    special_status = <special_status_key>  # IO special status (IO only).

    potential         = { <triggers> }   # Root = country/IO.
    allow             = { <triggers> }   # Root = country/IO.
    selectable_for    = { <triggers> }   # Root = country; scope:recipient = IO.

    modifier_when_in_debate = { <modifiers> }  # Applied while this issue is being debated.

    chance     = { <chance_block> }     # Probability this issue is selected.
    on_debate_start  = { <effects> }    # Fires when debate begins.
    on_debate_passed = { <effects> }    # Fires when the issue passes a vote.
    on_debate_failed = { <effects> }    # Fires when the issue fails.

    wants_this_parliament_issue_bias = <scripted_maths>  # AI preference score.
}
```

## Key Fields Reference

| Field | Purpose | Key constraint |
|---|---|---|
| `type` (all) | `country` or `international_organization` | Determines scope |
| `estate` | Which estate is associated (agendas/issues) | Multiple entries OR-combined |
| `modifier` (parliament type) | Country/IO modifier while this type is active | Standard modifier syntax |
| `modifier_when_in_debate` (issue) | Temporary modifier during active debate | Removed when debate ends |
| `importance` (agenda) | Scales the agenda's influence on parliament issue weighting | Default 1 |
| `on_accept` (agenda) | Executed when estate demand is met | Root = country/IO |
| `on_debate_passed` / `on_debate_failed` (issue) | Outcome effects | Root = country/IO |

## Modding Notes

- Parliament type determines the structural modifiers (e.g. which estates can participate, `has_a_parliamentary_system`). Switching parliament type is done through laws or reforms.
- Agendas and issues are separate: agendas are estate demands (fulfilled by clicking accept); issues are broader policy debates that apply a temporary modifier and have pass/fail outcomes.
- The `type = country` agendas reference `scope:target` as the estate type; `type = international_organization` agendas reference `scope:target` as the special status.
- `importance` on agendas scales how much that agenda affects the weighting of parliament issues — higher importance agendas push their associated issues more strongly to the floor.
- New parliament types require modifying laws/reforms to switch a country into them via `has_a_parliamentary_system` modifier flag.

## Example

A basic estate agenda (add a privilege to the nobles):
```
pa_add_privilege = {
    type   = country
    estate = nobles_estate
    estate = clergy_estate
    estate = burghers_estate
    estate = peasants_estate
    importance = 1.5

    potential = {
        "estate(scope:target)" = {
            num_privileges < 6
            num_possible_privileges > 0
        }
    }

    on_accept = {
        "estate(scope:target)" = {
            random_possible_privilege = {
                root = { grant_estate_privilege = prev }
            }
        }
    }

    can_bribe = { can_bribe_default_trigger = yes }
    on_bribe  = { on_bribe_default_effect = yes }
}
```
`scope:target` resolves to the estate type at runtime. `importance = 1.5` means this agenda has 50% more influence on the parliament issue queue than a standard agenda.
