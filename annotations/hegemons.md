# Hegemons
**Stage:** complete
**Keywords:** hegemons, gain, lose, modifier, economic_hegemon, naval_hegemon, military_hegemon, diplomatic_hegemon, cultural_hegemon

> **System type: Gameplay**

## Overview

Hegemons are world-power designations automatically awarded to the great power that dominates a specific domain (economy, navy, military, diplomacy, culture). A country gains a hegemon status when it leads all other great powers on the relevant metric and loses it when another great power exceeds it by a threshold. Only non-subject great powers are eligible. Each hegemon status grants a unique `modifier` block.

## Vanilla File Locations

| Path | Purpose |
|---|---|
| `in_game/common/hegemons/0_economic_hegemon.txt` | Economic hegemon (highest income) |
| `in_game/common/hegemons/1_naval_hegemon.txt` | Naval hegemon (strongest navy) |
| `in_game/common/hegemons/2_military_hegemon.txt` | Military hegemon (largest army) |
| `in_game/common/hegemons/3_diplomatic_hegemon.txt` | Diplomatic hegemon (most allies/subjects) |
| `in_game/common/hegemons/4_cultural_hegemon.txt` | Cultural hegemon (highest prestige/culture) |

Full file list in `_file_index.csv`.

## Block Structure

```
hegemon_key = {
    gain = { <triggers> }   # Conditions that must ALL pass for a country to gain this status.
                            # Root = country.
    lose = { <triggers> }   # Conditions that, if passed, cause the country to lose this status.
                            # Root = country.
    modifier = { <modifiers> }  # Applied to the country while the hegemon status is held.
}
```

## Key Fields Reference

| Field | Purpose | Key constraint |
|---|---|---|
| `gain` | Trigger block for acquiring hegemony | Root = country; must be great power and non-subject |
| `lose` | Trigger block for losing hegemony | Typically checks if another great power exceeds by 10% |
| `modifier` | Modifiers applied during hegemony | Standard modifier syntax |

## Modding Notes

- All vanilla hegemons require `is_great_power = yes` and `is_subject = no` in the `gain` block.
- `lose` typically includes a 10% buffer above the metric threshold to prevent rapid flickering (e.g. `monthly_income_trade_and_tax > root.monthly_income_trade_and_tax * 1.1`).
- Hegemon modifiers stack with all other country modifiers (rank, reforms, laws, etc.).
- Multiple countries cannot hold the same hegemon status simultaneously — the first country satisfying `gain` that is not already a hegemon of that type receives it.
- New hegemon types are fully scriptable; add a new file to `hegemons/`.

## Example

The economic hegemon:
```
economic_hegemon = {
    gain = {
        is_great_power = yes
        is_subject = no
        NOT = {
            any_other_great_power = {
                monthly_income_trade_and_tax > root.monthly_income_trade_and_tax
            }
        }
    }

    lose = {
        OR = {
            any_other_great_power = {
                monthly_income_trade_and_tax > {
                    value    = root.monthly_income_trade_and_tax
                    multiply = 1.1
                }
                is_subject = no
            }
            is_subject = yes
        }
    }

    modifier = {
        food_consumption_modifier         = -0.1
        allow_diplomacy_force_divert_trade = yes
        allow_cabinet_reduced_paperwork    = yes
    }
}
```
The gain condition checks that no other great power has a higher income; the lose condition requires a rival to exceed it by 10% to prevent thrashing. The modifier grants unique diplomatic abilities (force trade diversion) only available to economic hegemons.
