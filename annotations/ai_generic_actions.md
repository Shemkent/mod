# AI Generic Action Lists
**Stage:** complete
**Keywords:** `list_key`, `potential`, `actions`

> **System type: AI**

## Overview
Generic action AI lists are named conditional sets of generic actions the AI periodically evaluates and may execute. Each list specifies a `potential` trigger (conditions under which the list is considered at all) and an `actions` block listing generic action keys. When the AI runs its decision loop (typically on a monthly cadence), it checks applicable lists, filters by `potential`, and selects from the eligible actions using their individual AI weights. This system separates the question of *which context activates a set of actions* from the individual action definitions in `generic_actions/`.

## Vanilla File Locations

| File | Content |
|---|---|
| `in_game/common/generic_action_ai_lists/catholic_list.txt` | Catholic-specific actions (demand church tax, request cardinal seat, etc.) |
| `in_game/common/generic_action_ai_lists/aspiration_for_liberty_list.txt` | Disaster-context actions |
| `in_game/common/generic_action_ai_lists/colonial_charters_list.txt` | Colonial charter management |
| `in_game/common/generic_action_ai_lists/black_death_list.txt` | Black death response actions |
| `in_game/common/generic_action_ai_lists/columbian_exchange_list.txt` | Post-Columbian exchange actions |
| `in_game/common/generic_action_ai_lists/readme.txt` | Syntax reference and design rationale |
| … *(69 list files total)* | One file per contextual action set |

## Block Structure

```pdxscript
list_key = {
    potential = {          # trigger block; if false, AI skips this list entirely
        <country triggers>
    }
    actions = {            # generic action keys the AI considers from this list
        action_key_1
        action_key_2
        action_key_3
    }
}
```

## Key Fields Reference

| Field | Purpose | Notes |
|---|---|---|
| `potential` | Gate for list evaluation | Country-scope trigger; keeps AI from evaluating irrelevant lists every tick |
| `actions` | Generic action candidates | References keys from `in_game/common/generic_actions/`; each action's own `ai_will_do` determines selection weight |

## Modding Notes
- This system is a **routing layer** — it does not define action logic; it only tells the AI which actions to consider under which conditions. Action logic and AI weights live in `generic_actions/` (see [generic_actions.md](generic_actions.md)).
- Adding a new action to the AI requires both: (1) a list entry here pointing to it and (2) a defined `ai_will_do` block in the action's definition.
- Multiple lists can reference the same action. The vanilla readme does not document whether the AI de-duplicates weights across lists or evaluates each reference independently — test before relying on cross-list weight stacking.
- `global_list` **must not be removed** — any generic action without an explicit list entry is automatically assigned to it. New actions should be given a specific list to avoid bloating the global evaluation pass (the `global_list` header notes it is slow).
- `potential` should be as cheap as possible; it is evaluated each AI decision cycle (approximately monthly). Use tag, religion, or flag checks rather than complex iterators.
- 69 list files is a large set — adding a new list file is the correct approach for new contextual groups; do not add actions to unrelated vanilla lists.

## Example

```pdxscript
# AI considers Catholic church actions only when playing as Catholic
catholic_list = {
    potential = {
        religion = religion:catholic
    }
    actions = {
        demand_church_tax
        request_aid
        placitum_regium
        request_cardinal_seat
    }
}
```
