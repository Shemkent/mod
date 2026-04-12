# On Actions
**Stage:** complete
**Keywords:** `on_action_name`, `trigger`, `weight_multiplier`, `events`, `random_events`, `first_valid`, `on_actions`, `random_on_action`, `first_valid_on_action`, `effect`, `fallback`, `chance_to_happen`, `chance_of_no_event`, `delay`

> **System type: Scripted Logic**

## Overview
On actions are named hooks the engine fires at specific game moments (monthly country pulse, character death, ruler succession, battle end, etc.). Each hook can fire events, chain to other on actions, or execute inline effects. Multiple files can define blocks for the same on action name — all definitions are **merged additively** at load, making this the primary safe extension point for modders who want to inject events or effects into existing game moments without overwriting vanilla content.

## Vanilla File Locations

| File | Content |
|---|---|
| `in_game/common/on_action/_hardcoded.txt` | Reference list of all engine-fired on action names and their default scopes |
| `in_game/common/on_action/country_monthly.txt` | Monthly country pulse, papacy events, appanage events |
| `in_game/common/on_action/country_yearly.txt` | Yearly country pulse |
| `in_game/common/on_action/country_biyearly.txt` | Every-two-years pulse |
| `in_game/common/on_action/country_four_yearly.txt` | Every-four-years pulse |
| `in_game/common/on_action/character.txt` | Character birth, death, coming-of-age, marriage, etc. |
| `in_game/common/on_action/character_death_pulses.txt` | On character death: removes heir support, cleans up variables |
| `in_game/common/on_action/location_pulses.txt` | Location-scoped pulses |
| `in_game/common/on_action/religion_flavor_pulse.txt` | Religion-specific event routing |
| `in_game/common/on_action/government_flavor_pulse.txt` | Government-specific event routing |
| … *(19 content files total)* | Each file groups related game moments |

## Block Structure

```pdxscript
on_action_name = {
    trigger = {              # optional — if false, entire block is skipped
        <triggers>
    }

    weight_multiplier = {    # scales this block's weight in random_on_action lists
        base = 1
        modifier = { add = 1  <triggers> }
    }

    events = {               # always-fire events (each fires if its own trigger passes)
        event_id_1
        delay = { days = 365 }     # all events after this line fire with the delay
        event_id_2
        delay = { months = { 6 12 } }   # ranges supported; setting a new delay overrides the previous
        event_id_3
        # Note: delay is local to this block; a separate block for the same on_action
        # added by a mod starts with no delay, independent of this one
    }

    random_events = {        # exactly one event fires (or none)
        chance_to_happen    = 25   # integer 0–100; checked first — if it fails, weighted
                                   # selection is skipped entirely (performance gate)
        chance_of_no_event  = {    # script value evaluated after chance_to_happen passes;
            value = 0              # adds probability that no event fires despite passing the gate
            if = { limit = { <triggers> }  add = 10 }
        }
        100 = event_id_1    # weight : event_id; weighted random selection
        200 = event_id_2
        100 = 0             # explicit "no event" weight — separate from chance_of_no_event;
                            # represents a slot in the weighted pool that fires nothing
    }

    first_valid = {          # fires first event whose own trigger passes
        event_id_1
        event_id_2
        fallback_event       # no trigger = always valid
    }

    on_actions = {           # chain to other on actions (always)
        other_on_action_1
        other_on_action_2
    }

    random_on_action = {     # chain to one on action (weighted)
        100 = on_action_a
        200 = on_action_b
        100 = 0
    }

    first_valid_on_action = { on_action_a  on_action_b }

    effect = {               # inline effects; runs concurrently with events, NOT before
        <effects>
    }

    fallback = another_on_action   # fires only if nothing else in this block runs
}
```

## Key Fields Reference

| Field | Purpose | Notes |
|---|---|---|
| `trigger` | Gate the whole block | Checked first; if false, block is entirely skipped |
| `events` | Always-fire event list | Each event's own trigger is also checked; `delay` suspends all events listed after it |
| `random_events` | Weighted single-event picker | `chance_to_happen` is an integer 0–100 checked first; if it fails, no selection occurs. `chance_of_no_event` is a separate script-value probability evaluated after. `100 = 0` is a third mechanism — a weighted "no event" entry in the same pool |
| `first_valid` | Ordered fallback event list | First event with a passing trigger fires; useful for priority chains |
| `on_actions` | Chain to other hooks | Fires all listed on actions unconditionally |
| `random_on_action` | Weighted single on-action picker | Same mechanics as `random_events` |
| `effect` | Inline effect block | Executes **concurrently** with events — scopes/variables set here do NOT carry into fired events |
| `fallback` | Backup on action | Only fires if the block triggered no events or on actions; avoid infinite fallback loops |
| `delay` | Defer subsequent events | Validates at fire time AND again when delay expires; do not rely on state being unchanged |
| `weight_multiplier` | Adjust weight in parent random lists | Only relevant when this on action is a candidate in a `random_on_action` block |

## Modding Notes
- **Merge behaviour**: multiple definitions of the same on action name are merged — all `events`, `random_events`, `on_actions`, and `effect` blocks accumulate. This makes on_action the **safe injection point** for mod events; never replace vanilla files, just add new blocks.
- **Scope**: each on action fires in a specific scope context defined by the engine (see `_hardcoded.txt` comments). The `effect` block and any fired events share the same default scopes.
- **Concurrency caveat**: `effect = { }` and events fire concurrently. An effect that sets a variable and an event that reads it in the same on action are racing — do not rely on ordering between them.
- **Delayed events re-validate**: if a delay is used, the event's trigger is checked again at expiry. Conditions that were true when the on action fired may no longer hold.
- **Infinite fallback**: a `fallback` that routes to itself or creates a loop will freeze time. Always ensure fallback chains terminate.
- **`_hardcoded.txt`** is the ground truth for *engine-fired* on action names — the engine fires exactly those names at the documented moments. Modders can also define entirely custom on action names and fire them from effects (`fire_on_action = { on_action = my_custom_hook }`); those names do not appear in `_hardcoded.txt` and are equally valid.

## Example

```pdxscript
# Extend the monthly country pulse to fire a custom event
monthly_country_pulse = {
    random_events = {
        chance_to_happen = 3
        100 = mymod.1001    # fires ~3% of the time
        100 = 0
    }
}

# On character death: clean up a variable
on_character_death = {
    effect = {
        if = {
            limit = { has_variable = mymod_special_role }
            remove_variable = mymod_special_role
        }
    }
}
```
