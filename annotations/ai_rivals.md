# AI Rivals
**Stage:** complete
**Keywords:** `tag`, `enabled`, `ai_rule`

> **System type: AI**

## Overview
Rival criteria are tag-specific rules that restrict which countries the AI will choose as rivals. Each entry is keyed to a country tag and contains either an `enabled` trigger block (conditions a potential rival target must meet for *the keyed country's* AI to select them) or an `ai_rule` block (tags that *the keyed country's* AI will never rival). Players are unaffected — they can rival any country regardless of these entries.

## Vanilla File Locations

| File | Content |
|---|---|
| `in_game/common/rival_criteria/readme.txt` | Syntax reference |
| `in_game/common/rival_criteria/europe.txt` | Tag-specific rival restrictions for European countries (PAP, CAS, POR, ARA, etc.) |

## Block Structure

```pdxscript
TAG = {
    enabled = {          # triggers checked against the potential rival country
        <country triggers>
        # root = TAG (the defined country)
        # this = potential rival
    }
}

TAG = {
    ai_rule = {          # tags this country's AI should never rival
        # standard trigger logic (AND/OR/NOT) is fully supported here
        tag = OTHER_TAG
        OR = { tag = TAG_A  tag = TAG_B }
    }
}
```

Both `enabled` and `ai_rule` can be combined in one block. If a potential rival does not meet `enabled`, the AI will not pick them. If `ai_rule` matches a candidate, the AI skips them regardless of other criteria.

## Key Fields Reference

| Field | Purpose | Notes |
|---|---|---|
| `enabled` | Filter on potential rival targets | `root` = the defined TAG; `this` = potential rival; standard country triggers |
| `ai_rule` | Explicit block list | Tags listed here are never rivalled by this country's AI |

## Modding Notes
- This system is **AI-only** — player rivalry choices are unrestricted.
- Entries are keyed to country tags; only one entry per tag is valid (last-loaded wins for duplicate tags).
- If `enabled` is absent, the keyed country's AI has no restriction on target selection (any country meeting the engine's general rival criteria is eligible). `enabled` and `ai_rule` can coexist in one block; `ai_rule` is evaluated after `enabled` — a country blocked by `ai_rule` is never rivalled even if it passes `enabled`.
- `ai_rule` supports standard PDXScript trigger logic (AND/OR/NOT) — it is not a flat tag list.
- `!=` in `enabled` is shorthand for `NOT = { ... = ... }` and is valid PDXScript trigger syntax.
- `ai_rule` is used for diplomatic-narrative reasons (e.g. preventing Castile from rivalling Portugal so the Iberian union route stays open through diplomacy). `ai_rule` is one-directional — if you want bilateral protection, both countries need entries (`CAS` blocking `POR` and `POR` blocking `CAS`).
- No entry is needed for most tags — the AI will rival any country that meets general criteria. Only add entries when you need to override default AI behaviour for a specific tag.
- At 1.1.10 `europe.txt` is the only data file. To add entries for non-European tags, create a new file (e.g. `asia.txt`) in the same folder; the engine loads all `.txt` files in `rival_criteria/`.

## Example

```pdxscript
# Papacy only rivals countries of a different religion or excommunicated rulers
PAP = {
    enabled = {
        OR = {
            religion != c:PAP.religion
            is_excommunicated = yes
        }
    }
}

# Prevent Castile from rivalling Portugal or Aragon (push diplomatic Iberian union)
CAS = {
    ai_rule = {
        OR = { tag = POR  tag = ARA }
    }
}
```
