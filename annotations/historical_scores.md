# Historical Scores
**Stage:** complete
**Keywords:** `historical_scores`, `tag`, `score`

> **System type: UI/Presentation**
> **Cluster:** UI / Presentation

## Overview
Historical scores define the reference leaderboard entries shown in the end-game score screen. Each entry names a famous historical ruler or moment, associates it with a country tag, and assigns a target score. The game compares the player's final score against these benchmarks to contextualise their achievement. This is pure display data — no triggers, effects, or gameplay logic.

## Vanilla File Locations
`in_game/common/historical_scores/` — 1 file:
- `the_best_and_worst.txt` — the full reference list (Napoleon at 40 000 down to Hiawatha at 10 000)

## Block Structure

```pdx
hs_napoleon = {
    tag = FRA_revolutionary_republic   # country tag for display (flag shown on leaderboard)
    score = 40000                      # reference score value
}
```

## Key Fields Reference

| Field | Purpose | Notes |
|---|---|---|
| `tag` | Country tag for flag display | Any valid country tag; can be a formable nation tag |
| `score` | Reference score threshold | Integer; entries are ranked top-to-bottom |

## Modding Notes
- Entries have no effect on gameplay; they are purely cosmetic leaderboard references.
- The vanilla file is sorted by descending `score` value; the engine likely orders the leaderboard by score rather than file position.
- `tag` can reference tags that only exist as formable nations (e.g. `FRA_revolutionary_republic`, `JAP_tokugawa`) — the game uses their flag even if the country never existed in the playthrough.
- Scores are arbitrary integers; scale them to match the scoring system if you change how scores are calculated.
- Safe to add or remove entries freely.

## Example

```pdx
hs_suleyman  = { tag = TUR   score = 38000 }
hs_charles   = { tag = HRE   score = 36000 }
hs_mansa_musa = { tag = MAL  score = 30000 }
```
