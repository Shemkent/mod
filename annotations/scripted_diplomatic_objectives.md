# Scripted Diplomatic Objectives
**Stage:** complete
**Keywords:** `scripted_diplomatic_objectives`, `ai_tick_frequency`, `actor_trigger`, `recipient_trigger`, `recipient_priority`, `recipient_list_builder`, `pause_trigger`, `cancel_trigger`, `improve_relation`, `improve_relations_limit`, `defensive_support`, `antagonise`, `destroy`, `country_interactions`, `country_relations`, `time_limit`, `max_allowed`

> **System type: AI**

## Overview
Scripted diplomatic objectives are named AI behaviour templates that instruct a country's AI to pursue a sustained diplomatic strategy toward a filtered set of target countries. Once active, the AI repeatedly checks the objective and acts on its configured stances — such as improving relations, antagonising a rival, or supporting a target defensively — until the cancel trigger fires or the time limit expires. Vanilla uses this to drive HRE Emperor election campaigns and Japanese clan espionage.

## Vanilla File Locations

| File | Content |
|---|---|
| `in_game/common/scripted_diplomatic_objectives/hre_emperorship.txt` | HRE election objectives: improve with electors, antagonise rivals |
| `in_game/common/scripted_diplomatic_objectives/japanese_clan_spying.txt` | Japanese clan spy/antagonise objective |

## Block Structure

```pdxscript
objective_key = {
    ai_tick_frequency = <int|script_value>    # days between re-evaluations

    actor_trigger = {                         # ROOT = actor; must pass for objective to activate/persist
        <triggers>
    }

    recipient_trigger = {                     # scope:recipient = candidate; scope:actor = actor
        <triggers>                            # filters which countries become targets
    }

    recipient_priority = <float|script_value> # higher = prioritised over other objectives (and other targets)

    recipient_list_builder = {                # narrows candidate pool; if absent, all reachable countries used
        <list builder>
    }

    pause_trigger = {                         # scope:recipient, scope:actor — pause without cancelling
        <triggers>
    }

    cancel_trigger = {                        # scope:recipient, scope:actor — cancel the objective
        <triggers>
    }

    # Relation stances (what the AI should do toward the recipient)
    improve_relation         = yes/no
    improve_relations_limit  = <int|script_value>  # opinion target; omit = improve to max
    defensive_support        = yes/no              # AI tries to defend recipient
    antagonise               = yes/no              # AI uses hostile interactions/relations
    destroy                  = yes/no              # AI tries to destroy recipient

    country_interactions = {                  # encourage (yes) or disable (no) specific interactions
        <interaction_key> = yes/no
    }

    country_relations = {                     # encourage (yes) or disable+cancel (no) specific relations
        <relation_key> = yes/no
    }

    time_limit   = <days|script_value>        # 0 or absent = unlimited
    max_allowed  = <int|script_value>         # max simultaneous objectives of this type per actor; 0 = unlimited
}
```

## Key Fields Reference

| Field | Purpose | Notes |
|---|---|---|
| `ai_tick_frequency` | Days between re-checks | Lower = more responsive, higher CPU cost |
| `actor_trigger` | Activates/sustains the objective | Checked against ROOT (the acting country) |
| `recipient_trigger` | Filters valid target countries | Checked with `scope:recipient` and `scope:actor` |
| `recipient_priority` | Sorting weight | Higher priority objectives and recipients are acted on first |
| `recipient_list_builder` | Narrows candidate list | Performance optimisation — use to pre-filter before `recipient_trigger` |
| `pause_trigger` | Suspends objective without cancelling | Useful for temporary blocks (e.g. at war); objective resumes automatically when the condition clears |
| `defensive_support` | Tell AI to defend the recipient | AI will join wars in the recipient's defence |
| `cancel_trigger` | Ends the objective | Objective is removed; AI stops pursuing it |
| `improve_relation` | Tell AI to improve relations | `improve_relations_limit` sets the opinion ceiling |
| `antagonise` | Tell AI to use hostile actions/relations | Combines with `country_interactions` for specifics |
| `destroy` | Tell AI to eliminate the recipient | Strongest hostile stance |
| `time_limit` | Max duration in days | Safety valve to prevent stuck objectives |
| `max_allowed` | Cap simultaneous objectives of this type | Prevents AI spreading too thin |

## Modding Notes
- This is a purely AI system — it produces no visible UI and does not fire events. Effects are entirely through AI decision-making.
- `recipient_list_builder` is a performance field: use it to avoid checking every country in the world against `recipient_trigger` each tick.
- Objectives stack: an actor can hold multiple objectives of different types simultaneously (up to `max_allowed` per type).
- Cross-system dependency: `country_interactions` entries reference interaction keys defined in `in_game/common/country_interactions/` (see [country_interactions.md](country_interactions.md)); `country_relations` entries reference relation type keys defined engine-side (alliance, guarantee, etc.).

## Example

```pdxscript
# Hostile objective: antagonise a rival and disable alliance formation
rival_suppression = {
    ai_tick_frequency = 180
    actor_trigger = {
        has_rival = yes
    }
    recipient_trigger = {
        scope:actor = { is_rival_of = scope:recipient }
    }
    recipient_priority = 150
    antagonise         = yes
    country_interactions = {
        offer_alliance = no    # disable; AI will not offer alliance to this country
    }
    country_relations = {
        alliance = no          # disable+cancel any existing alliance with recipient
    }
    cancel_trigger = {
        NOT = { scope:actor = { is_rival_of = scope:recipient } }
    }
    max_allowed = 3
}

# Friendly objective: HRE Emperor campaigns to improve relations with electors
hre_improve_with_electors = {
    ai_tick_frequency    = 360
    actor_trigger        = {
        is_leader_of_international_organization = international_organization:hre
    }
    recipient_trigger    = {
        international_organization:hre = {
            country_has_special_status = { type = special_status:elector  country = scope:recipient }
        }
        NOT = { scope:recipient = scope:actor }
    }
    recipient_priority   = 100
    improve_relation     = yes
    improve_relations_limit = 150
    max_allowed          = 7
}
```
