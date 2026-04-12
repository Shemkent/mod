# AI Biases (Opinion & Trust Modifiers)
**Stage:** complete
**Keywords:** `opinion_key`, `trust_key`, `antagonism_key`, `value`, `max`, `min`, `months`, `years`, `yearly_decay`, `yearly_gain`

> **System type: AI**

## Overview
The biases system defines named opinion, trust, and antagonism modifier entries that the engine applies automatically at specific diplomatic moments (war, cultural relations, received interactions, startup) or via scripted effects. Each entry is a static value with optional caps, a decay rate, and an optional expiry duration. Unlike auto modifiers (which apply modifier fields to a country), bias entries apply directly to the bilateral opinion/trust/antagonism score between two countries. The naming convention (`opinion_`, `trust_`, `antagonism_` prefix) is not enforced — it is a human-readability convention only and has no engine significance. The engine identifies entries by the scripted effect or hardcoded call that references them, not by the key name.

## Vanilla File Locations

| File | Content |
|---|---|
| `in_game/common/biases/00_opinion_hardcoded.txt` | Engine-applied opinion modifiers (war, culture, language, religion) |
| `in_game/common/biases/01_opinion_scripted_diplomacy.txt` | Opinion modifiers from diplomatic actions (influence, sabotage, trade access, etc.) |
| `in_game/common/biases/02_opinion_subject_types.txt` | Subject relationship opinions |
| `in_game/common/biases/03_opinion_from_events.txt` | Event-fired opinion modifiers |
| `in_game/common/biases/04_trust_hardcoded.txt` | Engine-applied trust modifiers |
| `in_game/common/biases/05_antagonism_hardcoded.txt` | Engine-applied antagonism modifiers |
| `in_game/common/biases/06_free_cities_opinions.txt` | Free city opinion modifiers |
| `in_game/common/biases/07_opinion_from_start_up.txt` | Initial opinion modifiers applied at game start |

## Block Structure

```pdxscript
opinion_key = {
    value        = <int>            # opinion value added (positive or negative)
    max          = <int>            # optional upper cap on this entry's contribution
    min          = <int>            # optional lower cap
    months       = <int>            # optional; entry expires after this many months
    years        = <int>            # optional; alternative to months, expiry in years (common in trust entries)
    yearly_decay = <int>            # optional; value moves toward zero by this amount each year;
                                    # negative yearly_decay causes the value to grow more extreme each year
    yearly_gain  = <float>          # optional; value grows toward max each year (inverse of yearly_decay; common in trust entries)
}
```

The same structure applies to trust and antagonism entries (same fields, different bilateral stat targeted).

## Key Fields Reference

| Field | Purpose | Notes |
|---|---|---|
| `value` | Base opinion/trust/antagonism delta | Positive = friendly; negative = hostile |
| `max` / `min` | Caps on this specific entry's contribution | Does not cap the total score — only this entry |
| `months` | Expiry duration | Entry removed after N months; absent = permanent until removed by effect |
| `years` | Expiry duration in years | Alternative to `months`; common in trust entries |
| `yearly_decay` | Annual movement toward zero | Moves value toward 0 each year; **negative value** causes the entry to worsen (grow more extreme) rather than recover |
| `yearly_gain` | Annual growth toward cap | Value grows by this amount per year toward `max`; inverse of `yearly_decay`; used in trust entries for slow-building relationships |

## Modding Notes
- Bias entries are applied by engine code at hardcoded moments (war, cultural acceptance, subject relations) or via scripted effects: `add_opinion = { target = scope:x  modifier = opinion_key }`, `add_trust = { target = scope:x  modifier = trust_key }`, `add_antagonism = { target = scope:x  modifier = antagonism_key }`. All three use the same `modifier =` parameter name.
- `yearly_decay` and `months` are independent — an entry can both decay and expire; whichever condition is met first removes it.
- Adding new entries here creates named opinion modifier keys usable in `add_opinion`, `add_trust`, and `add_antagonism` effects.
- The specific keys in the hardcoded files (00, 04, 05) that the engine references by name must not be removed or renamed. New keys can be added to these files freely, and new mod files can be added alongside them.
- A negative `yearly_decay` causes the value to compound in the hostile direction over time — use deliberately for escalating penalties, but avoid accidentally setting it negative in positive-opinion modifiers.
- Duplicate keys: last-loaded wins. Files are loaded alphabetically by filename, so a mod file beginning with `99_` will override a vanilla `00_` entry of the same key.
- Opinion, trust, and antagonism are separate bilateral scores; a bias file in this folder applies to whichever score the engine uses it for — the file name distinguishes context, not the block syntax.

## Example

```pdxscript
# Hardcoded: permanent war opinion penalty
opinion_war = {
    value = -200
}

# Scripted diplomacy: decay-based opinion from sabotage
opinion_sabotage_reputation = {
    value        = -50
    yearly_decay = 5
}

# Expiring opinion from an event
opinion_grateful_for_aid = {
    value  = 25
    max    = 50
    months = 60
}
```
Applied in script via:
```pdxscript
add_opinion = { target = scope:benefactor  modifier = opinion_grateful_for_aid }
```
