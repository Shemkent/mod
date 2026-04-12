# Regencies
**Stage:** complete
**Keywords:** regencies, regency, start_effect, end_effect, allow, modifier, internally_assigned, interregnum, consort_regency, nobles_regency, clergy_regency

> **System type: Gameplay**

## Overview

Regencies cover the period between rulers — when a country has no adult ruler and requires a regent or is in interregnum. Each regency definition specifies who can hold the position (`allow`), what happens at start/end, what modifiers apply, and whether the regent is assigned automatically by the engine (`internally_assigned`). Vanilla ships a default interregnum plus regencies for consorts, subjects, overlords, and all four major estates.

## Vanilla File Locations

| Path | Purpose |
|---|---|
| `in_game/common/regencies/readme.txt` | Field reference |
| `in_game/common/regencies/zz_default.txt` | Fallback interregnum |
| `in_game/common/regencies/10_consort_regency.txt` | Consort regency |
| `in_game/common/regencies/11_subject_regency.txt` | Subject country regent |
| `in_game/common/regencies/12_lordship_of_ireland_regency.txt` | HRE/Ireland-specific |
| `in_game/common/regencies/1_nobles_regency.txt` | Nobles estate regent |
| `in_game/common/regencies/2_clergy_regency.txt` | Clergy estate regent |
| `in_game/common/regencies/3_burghers_regency.txt` | Burghers estate regent |
| `in_game/common/regencies/4_peasants_regency.txt` | Peasants estate regent |
| `in_game/common/regencies/9_overlord_regency.txt` | Overlord-appointed regent |
| `in_game/common/regencies/00_fratricide_succesion.txt`, `00_judicial_election.txt`, `00_mamluk_succesion.txt`, `00_papal_election.txt`, `00_republican_election.txt` | Government-specific succession forms |

Full file list in `_file_index.csv`.

## Block Structure

```
regency_key = {
    internally_assigned = yes/no    # Engine assigns this regent automatically (no player choice).

    allow = { <triggers> }          # Root = country. When this regency type applies.

    modifier = { <modifiers> }      # Applied to the country during this regency.

    start_effect = { <effects> }    # Root = country. Fires when regency begins.
    end_effect   = { <effects> }    # Root = country. Fires when regency ends.
}
```

## Key Fields Reference

| Field | Purpose | Key constraint |
|---|---|---|
| `internally_assigned` | Engine auto-assigns regent without UI prompt | Default no |
| `allow` | Determines if this regency type applies | Evaluated in file order; first match wins |
| `modifier` | Country modifier during regency | Standard modifier syntax |
| `start_effect` | One-time effects when regency begins | Root = country |
| `end_effect` | One-time effects when regency ends | Root = country |

## Modding Notes

- Regency types are evaluated in **file order** (sorted by filename) — the first `allow` block that passes is used. The fallback `interregnum` in `zz_default.txt` has `allow = { always = yes }` to catch everything else.
- `internally_assigned = yes` means no character is displayed as regent in the UI; the engine handles assignment internally (used for elections like republic terms).
- File naming controls priority: `00_` prefixed files (fratricidal succession, papal election) are checked first; `zz_default.txt` is always last.
- Add new regency types by creating a new file with an appropriately-ordered name prefix.

## Example

The consort regency:
```
consort_regency = {
    allow = {
        any_character = {
            is_consort = yes
            is_available = yes
            is_adult = yes
        }
    }

    start_effect = {
        random_character = {
            limit = {
                is_consort = yes
                is_available = yes
                is_adult = yes
            }
            save_scope_as = regent
        }
        # ... assign regent to scope:regent
    }

    modifier = {
        stability_investment = stability_investment_penalty
    }
}
```
The `allow` block checks for an eligible consort in court. If one exists, this regency type applies instead of falling through to an estate or the interregnum.
