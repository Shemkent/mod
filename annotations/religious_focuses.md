# Religious Focuses
**Stage:** complete
**Keywords:** religious_focus, potential, allow, monthly_progress, modifier_while_progressing, modifier_on_completion, effect_on_completion, ai_will_do, religious_focuses

> **System type: Gameplay**

## Overview
Religious focuses are religion-specific research projects — analogous to the advances system but scoped to individual faiths. A country researches a focus over time (governed by `monthly_progress`), receiving `modifier_while_progressing` during the process and `modifier_on_completion` permanently afterward. Focuses are assigned to religions via a `religious_focuses {}` list in the religion's definition block. Vanilla uses focuses for Nahuatl/Aztec religion (`nahuatl.txt`). The system is fully scriptable and can be added to any religion.

## Vanilla File Locations
- `in_game/common/religious_focuses/` — 2 `.txt` files (1 readme + 1 definition file for Nahuatl focuses)
- Focuses are registered on religions via `religious_focuses { }` in `religions/` files
- Full file list in `_file_index.csv`.

## Block Structure
```pdx
<focus_id> = {
    # --- UI availability ---
    potential = { <trigger> }   # root = country; controls visibility in the UI (unused in vanilla)
    allow     = { <trigger> }   # root = country; controls whether the focus can be started

    # --- AI weighting ---
    ai_will_do = { <script value> }   # how much the AI wants to research this focus

    # --- Research speed ---
    monthly_progress = { <script value> }   # root = country; amount of progress per month

    # --- Modifiers ---
    modifier_while_progressing = {
        <modifier_key> = <value>   # applied to the country while research is in progress
    }
    modifier_on_completion = {
        <modifier_key> = <value>   # applied permanently once research completes
    }

    # --- One-time effect ---
    effect_on_completion = {
        <effect>   # root = country; fired once when research finishes
    }
}
```

To make focuses available, register them on the religion:
```pdx
# in religions/<religion>.txt
<religion_id> = {
    # ...
    religious_focuses = {
        <focus_id>
        <focus_id>
    }
}
```

## Key Fields Reference
| Field | Purpose | Key constraint |
|---|---|---|
| `potential` | UI visibility trigger | Root = country; focus is hidden if this fails; not used in any vanilla focus |
| `allow` | Research start trigger | Root = country; focus is shown but greyed out if this fails; used in vanilla to enforce sequential ordering |
| `ai_will_do` | AI research priority | Script value; all vanilla focuses define this — omitting it may cause the AI to never pursue the focus |
| `monthly_progress` | Research speed | Script value; root = country; higher = faster completion |
| `modifier_while_progressing` | Temporary modifiers during research | Active only while the focus is being researched; removed on completion |
| `modifier_on_completion` | Permanent modifiers after completion | Applied permanently once research finishes; cannot be removed without scripted effects |
| `effect_on_completion` | One-time effect on completion | Root = country; use for unique unlocks, event triggers, or flag assignments |
| `religious_focuses` (on religion) | Registers focuses for a religion | IDs listed here become available to countries of that religion |

## Modding Notes
- **Focuses must be registered on a religion** via the `religious_focuses { }` block in `religions/`. Defining a focus without listing it there makes it invisible to all countries.
- **Focuses can be heavily sequenced via `allow`.** Vanilla's Nahuatl focuses require completing all 7 prerequisite focuses before `adopt_ometeotl` is available — the `allow` trigger checks `has_completed_religious_focus` for each. Calling focuses "independent" is misleading for the vanilla implementation; treat them as optionally sequential, with `allow` as the ordering mechanism.
- **`modifier_while_progressing` is temporary.** It is removed when research completes (succeeded or abandoned). Use it for costs or drawbacks during the research period.
- **`modifier_on_completion` is permanent** and stacks with other religion/country modifiers. Plan the full modifier stack carefully.
- **`effect_on_completion`** fires exactly once. Use scripted variables or flags to track completion state for `allow` triggers in dependent focuses.
- **Cross-system:** religious focuses are structurally similar to `advances` (see `advances.md`). They interact with the religion system (registration), the modifier system (both modifier blocks), and the scripted effects/triggers system (`effect_on_completion`, `allow`/`potential`).
