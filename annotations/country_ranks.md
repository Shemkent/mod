# Country Ranks
**Stage:** complete
**Keywords:** country_ranks, rank_modifier, level, ai_level, allow, language_power_scale, character_ai_cooldown, diplomacy_ai_cooldown

> **System type: Gameplay**

## Overview

Country ranks are the prestige tiers that classify a country's size and power (duchy, kingdom, empire, etc.). Each rank applies a `rank_modifier` block to all countries of that tier, sets diplomatic capacity, fort limits, and other scale-sensitive values. Ranks are evaluated dynamically — a country's rank can change as it grows or shrinks. The `level` integer determines ordering; `ai_level` governs AI scope (higher = interacts with more countries).

## Vanilla File Locations

| Path | Purpose |
|---|---|
| `in_game/common/country_ranks/readme.txt` | Field reference |
| `in_game/common/country_ranks/00_default.txt` | All rank definitions |

Full file list in `_file_index.csv`.

## Block Structure

```
rank_key = {
    color = <color_key>                # Map color for this rank tier.

    rank_modifier = { <modifiers> }    # Modifiers applied to all countries of this rank.

    level            = <int>           # Numeric tier (higher = larger/stronger). Used for comparisons.
    ai_level         = <int>           # AI scope of interaction (1–4). Defaults to 1.
    language_power_scale = <float>     # Multiplier on court language power. Default 1.
    character_ai_cooldown   = <float>  # How often character interactions are evaluated. Default 1.
    diplomacy_ai_cooldown   = <float>  # How often country interactions are evaluated. Default 1.

    victory_card = yes/no              # Whether this rank counts for victory cards.

    allow = { <triggers> }             # Condition for a country to hold this rank.
}
```

## Key Fields Reference

| Field | Purpose | Key constraint |
|---|---|---|
| `rank_modifier` | Modifiers applied to all countries at this rank | Standard modifier syntax |
| `level` | Numeric tier; used in trigger `country_rank >= country_rank:X` | Higher = more powerful tier |
| `ai_level` | Caps how many countries AI interacts with diplomatically | Values 1–4; higher = broader AI |
| `language_power_scale` | Multiplies language power gained from being court language | Float; 2.0 for empire |
| `character_ai_cooldown` | Modifies character interaction frequency | < 1 = more frequent |
| `diplomacy_ai_cooldown` | Modifies diplomatic interaction frequency | < 1 = more frequent |
| `allow` | Trigger block a country must satisfy to hold this rank | Root = country |
| `victory_card` | Whether this rank grants a victory card entry | Relevant for score tracking |

## Modding Notes

- Ranks are checked dynamically — `allow` determines if a country qualifies each tick.
- `rank_modifier` stacks with all other modifiers (government type, reforms, laws). It is the primary source of per-rank scaling (diplomat count, fort limit, diplomatic capacity).
- When inserting a new rank between vanilla ranks, set `ai_level` to the rough vanilla equivalent to avoid breaking AI scoping.
- Triggers like `country_rank = country_rank:rank_empire` compare rank keys directly; `country_rank >= country_rank:rank_kingdom` uses level integers.

## Example

The `rank_empire` definition (abbreviated):
```
rank_empire = {
    color = color_rank_empire

    rank_modifier = {
        max_diplomats          = 2
        diplomatic_range       = 1000
        monthly_prestige       = 0.2
        num_possible_rivals    = 4
        fort_limit             = 2
        cultures_capacity      = 2
        government_size        = 1
        diplomatic_capacity    = 3
        allowed_alliance       = yes
        allowed_guarantee      = yes
        num_local_governors    = 1
    }

    level           = 4
    victory_card    = yes
    ai_level        = 4
    language_power_scale = 2
}
```
The empire tier grants the most diplomatic and military headroom. `ai_level = 4` means the AI considers the widest set of potential partners and rivals.
