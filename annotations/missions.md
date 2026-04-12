# Missions
**Stage:** complete
**Keywords:** `mission_key`, `task_key`, `repeatable`, `player_playstyle`, `visible`, `enabled`, `abort`, `chance`, `ai_will_do`, `on_potential`, `on_start`, `on_abort`, `on_completion`, `on_post_completion`, `on_monthly`, `requires`, `final`, `bypass`, `duration`, `modifier_while_progressing`, `on_persistent_completion`

> **System type: Gameplay**

## Overview
Missions are country-level objective chains presented to the player as a set of tasks to complete in sequence. Each mission pack bundles one or more missions; each mission contains task nodes with prerequisite links, completion triggers, and reward effects. Up to a fixed number of mission chains (`POTENTIAL_MISSION_COUNT` — see `____Info.txt` for the current value) are shown at one time, selected by weighted `chance`. The `mission_task_defs/` folder is an empty stub — all task definitions live directly inside mission files.

## Vanilla File Locations

| File | Content |
|---|---|
| `in_game/common/missions/____Info.txt` | Authoritative field reference |
| `in_game/common/missions/generic_conquer_province_mission_pack.txt` | Generic military conquest mission |
| `in_game/common/missions/generic_capital_economy_mission_pack.txt` | Capital economy development |
| `in_game/common/missions/generic_colonize_explore_mission_pack.txt` | Colonisation/exploration |
| `in_game/common/missions/generic_trade_mission_pack.txt` | Trade focus |
| `in_game/common/missions/generic_infrastructure_mission_pack.txt` | Infrastructure/buildings |
| … *(11 content files total)* | One file per mission pack |
| `in_game/common/mission_task_defs/` | **Empty stub** — tasks live inside mission files |

## Block Structure

```pdxscript
mission_key = {
    header          = ""            # header image path
    icon            = ""            # icon path for mission button
    repeatable      = yes/no        # default no; can the mission recur after completion
    player_playstyle = military     # optional hint: diplomatic / military / administrative

    visible = { <triggers> }        # shown in mission list at all
    enabled = { <triggers> }        # mission can be selected by player; NOT the completion condition
    abort   = { <triggers> }        # mission auto-aborts when this passes while active

    chance      = <script_value>    # weight for appearing in the potential mission pool
    ai_will_do  = { <script_value> }# AI priority for selecting this mission

    on_potential   = { <effects> }  # fires when mission enters the potential pool; mission scope created here
                                    # and persists until abort/completion — use for setup variables
    on_start       = { <effects> }  # fires when player selects the mission; prefer on_potential for
                                    # scope setup that must exist before on_start is reachable
    on_abort       = { <effects> }  # fires on abort (when abort trigger passes)
    on_completion  = { <effects> }  # fires on full mission completion (all final tasks done)
    on_post_completion = { <effects> } # fires after mission removed from country

    # --- Task nodes (nested inside the mission block) ---
    task_key = {
        icon      = ""
        requires  = { other_task_key }  # prerequisite tasks; omit for starting tasks
        final     = yes/no              # completing this task completes the mission

        visible   = { <triggers> }
        highlight = { <triggers> }      # scope:province — highlight map provinces on hover
        enabled   = { <triggers> }      # task completion condition
        bypass    = { <triggers> }      # task skipped if this passes

        ai_will_do = { <script_value> }
        duration   = <days>             # optional timer; instant if absent

        on_monthly             = { <effects> }  # fires each month while task is active
        on_start               = { <effects> }  # fires when timed task starts
        on_persistent_start    = { <effects> }  # fires when timed task starts, always
        modifier_while_progressing = { <modifiers> }  # applied to country during task

        on_completion          = { <effects> }  # fires on completion; skipped if task_rewards_disabled
        on_persistent_completion = { <effects> } # fires on completion, always
        on_bypass              = { <effects> }
    }
}
```

## Key Fields Reference

| Field | Purpose | Notes |
|---|---|---|
| `chance` | Weight in the potential mission pool | Higher = more likely to appear in the offered set |
| `repeatable` | Can mission fire more than once | Default `no`; use variables to gate re-occurrence |
| `ai_will_do` | AI mission/task selection weight | Mission-level: `{ <script_value> }` block; task-level: same — AI uses it to pick between eligible tasks |
| `on_potential` | Setup hook when mission enters pool | Mission scope created here and persists; use to save targeting variables before `on_start` |
| `on_abort` | Cleanup on abort | Fires when `abort` trigger passes while mission is active; mirror any `on_potential` setup here |
| `on_monthly` | Monthly task effect | Task-scope; fires each month while the timed task is active, not throughout the mission |
| `final` | Task that completes the whole mission | At least one task must have `final = yes` |
| `requires` | Prerequisite tasks | Forms the task DAG; omit for root (starting) tasks |
| `bypass` | Auto-skip condition | Task counted as complete without rewards when this trigger passes; use for conditional branches, not ordering |
| `duration` | Timed task length in days | Absent = instant; `on_start` and `on_monthly` only valid for timed tasks |
| `on_completion` vs `on_persistent_completion` | Reward gating | `on_completion` skipped when `task_rewards_disabled` rule is active; `on_persistent_completion` always runs — use latter for counters and cleanup |
| `modifier_while_progressing` | Modifier applied during timed task | Removed on task completion or bypass |
| `on_post_completion` | Post-mission cleanup | Runs after the mission scope is removed; safe for final cleanup |

## Modding Notes
- Missions are defined in packs — one mission or several per file — but each mission stands alone; packs are a file-organisation convention only.
- The mission scope created at `on_potential` persists through the entire lifecycle. Use it to save variables needed across tasks.
- `task_rewards_disabled` (a scripted rule / game rule) suppresses `on_completion` but not `on_persistent_completion` — use `on_persistent_completion` for effects that must always fire (e.g. advancing a counter), and `on_completion` for optional player rewards.
- `bypass` is for conditional skipping, not ordering. Ordering is handled by `requires`. Use `bypass` when task B should be silently skipped if task A was already completed — e.g. two mutually exclusive paths through a mission.
- Duplicate mission keys across files: last-loaded wins. Namespace mission and task keys to avoid collisions.
- `mission_task_defs/` is an empty database stub; do not put content there.

## Example

```pdxscript
# Two-task chain with requires and bypass
build_fleet_mission = {
    repeatable = yes
    chance     = 1000
    visible    = { has_coastline = yes }

    on_potential = {
        # scope created here; persists until abort/completion
        set_variable = { name = fleet_mission_target  value = 0 }
    }
    on_abort = {
        remove_variable = fleet_mission_target
    }

    secure_port_task = {
        # root task — no requires
        enabled = {
            any_owned_non_rural_location = { is_coastal = yes  has_building = shipyard }
        }
        on_persistent_completion = {
            set_variable = { name = fleet_mission_target  value = 1 }
        }
    }

    build_warships_task = {
        requires = { secure_port_task }   # only available after secure_port_task
        final    = yes
        enabled  = { navy_size >= 10 }
        on_completion = { add_prestige = 100 }
        on_persistent_completion = {
            remove_variable = fleet_mission_target
        }
    }
}

# Original simple example
generic_conquer_province = {
    icon       = generic_conquer_province
    repeatable = yes
    player_playstyle = military

    visible = {
        game_has_missions_enabled = yes
        NOT = { has_variable = recently_had_generic_conquer_province_variable }
    }
    enabled = {
        any_neighbor_country = { ROOT = { can_declare_war_on = prev } }
    }
    chance = 3600

    on_start = {
        set_variable = { name = target_province_variable  value = 1 }
    }

    conquer_target_task = {
        final = yes
        enabled = {
            any_owned_province = { has_variable = target_province_variable }
        }
        on_completion = {
            add_prestige = 50
        }
        on_persistent_completion = {
            remove_variable = target_province_variable
        }
    }
}
```
