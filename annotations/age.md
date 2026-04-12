# Age System
**Stage:** complete
**Keywords:** `age_[id]`, `year`, `price_stability`, `max_price`, `hegemons_allowed`, `efficiency`, `unique`, `modifier`, `mercenaries`, `victory_card`, `war_score_from_battles`, `months_for_exploration_spread`, `max_ai_privilege_per_estate`, `min_ai_privilege_per_estate`

> **System type: Gameplay**

## Overview
Ages are global time-period definitions that segment the game into six historical eras (Traditions → Renaissance → Discovery → Reformation → Absolutism → Revolutions). Each age sets world-wide economic parameters, military scale modifiers, hegemon availability, victory card targets, and AI estate-privilege behavior. As the game clock advances through each age's `year` threshold, these modifiers replace the previous era's values — making ages the primary lever for tuning how the game feels at different historical periods. Modders can add new ages, shift start years, or tweak individual modifiers to reshape the game's pacing and difficulty curve.

## Vanilla File Locations
All age definitions live in a single file:

| File | Content |
|---|---|
| `in_game/common/age/00_default.txt` | All 6 age blocks in chronological order |

For the full file list filtered by this system, see `_file_index.csv`.

## Block Structure
```
age_[identifier] = {
    year = 1342                        # Calendar year this age begins
    price_stability = 0.08             # Price drift/stabilization coefficient
    max_price = 4                      # Maximum price multiplier for goods

    hegemons_allowed = yes             # Whether any country can pursue hegemony (omit = no)

    months_for_exploration_spread = 1800  # Time for exploration knowledge to spread between countries

    unique = {                         # Age-specific global bonuses (applied on top of modifier)
        diplomatic_capacity_modifier = 0.25
        cultural_tradition_modifier  = 0.25
    }

    modifier = {                       # Standard global modifier block (see modifiers.md)
        # Military scale: expected_army_size_modifier, levy_maintenance_modifier, global_navy_levy_size_modifier
        # War score: global_war_score_cost, lack_of_control_impact_on_warscore, war_score_from_battles
        # Antagonism: antagonism_religion_influence, antagonism_culture_influence, antagonism_government_type_influence, antagonism_societal_value_influence
        # Construction & Subjects: construction_center_max_level, subject_pays_colonial_cost_modifier, subject_loyalty
        expected_army_size_modifier = 2.0
        levy_maintenance_modifier = 0.25
    }

    mercenaries = 0.25                 # Positive = cheaper, negative = expensive/unavailable
    efficiency  = 0.6                  # General efficiency (1.0 = best, 0.1 = worst)
    victory_card = 2                   # Target victory card count for this age
    war_score_from_battles = -0.2      # Modifier on warscore from battles (omit = 0)

    max_ai_privilege_per_estate = { ... }   # AI: upper privilege bound per estate type
    min_ai_privilege_per_estate = { ... }   # AI: lower privilege bound per estate type
}
```

## Key Fields Reference

| Field | Purpose | Notes |
|---|---|---|
| `year` | Age start year | Years must be ascending; applied in file order |
| `price_stability`, `max_price` | Economic params | Stability drops 0.1→0.02; max_price rises 3→5 |
| `hegemons_allowed` | Hegemon gate | Absent (falsy) in Ages 1–2; present from Age 3 |
| `months_for_exploration_spread` | Exploration speed | Falls 1800→300; later ages spread faster |
| `unique`, `modifier` | Global bonuses & modifiers | `unique` is semantic flavor; functionally same as `modifier` |
| `mercenaries` | Mercenary cost | Positive = cheaper (+0.25 Age 2); negative = unavailable (−0.9 Age 6) |
| `efficiency` | Global efficiency | Falls 1.0→0.1 across ages; scales multiple systems |
| `victory_card`, `war_score_from_battles` | Victory conditions | Cards: 0–5; war score penalty increases over time |
| `max/min_ai_privilege_per_estate` | AI estate policy | Both drop to 0 by Age 6 (revolutionary upheaval) |

## Modding Notes

- **Efficiency scaling:** `efficiency` falls 1.0 → 0.8 → 0.6 → 0.4 → 0.2 → 0.1. It is not a binary flag — avoid large jumps or values outside 0–1.
- **Army size doubling pattern:** `expected_army_size_modifier` roughly doubles each age (1→2→4→8→16). Custom ages should fit this curve or the AI will over/under-recruit.
- **Mercenary sign flip:** Mercenaries are *cheaper* in Age 2 (`+0.25`) but increasingly *unavailable* in Ages 5–6 (`-0.40`, `-0.90`). A value approaching `-1.0` effectively removes mercenaries.
- **Hegemon gate:** `hegemons_allowed` is absent (falsy) in the first two ages. If adding an early age, omit it to preserve the intended hegemon unlock timing.
- **AI privilege tuning:** `max_ai_privilege_per_estate` and `min_ai_privilege_per_estate` are performance hints — they control how aggressively the AI re-evaluates estate privileges. Age 6 deliberately sets both to `0` for all estates, representing revolutionary upheaval stripping aristocratic power.
- **`unique` vs `modifier`:** Functionally equivalent; `unique` is a semantic convention for flavor bonuses tied to the age's identity. Both accept the same modifier keys.
- **Load order:** File is prefixed `00_` — a replacement mod should use a higher prefix (e.g. `01_`) to override, or use the same `age_[id]` key in a new file to overwrite specific blocks.
- **Cross-system dependencies:** `modifier` keys like `antagonism_*_influence`, `construction_center_max_level`, and `subject_pays_colonial_cost_modifier` interact with antagonism, buildings, and subject systems respectively.

## Examples

### `age_6_revolutions` (1737) — late game
```
age_6_revolutions = {
    year = 1737
    price_stability = 0.02
    max_price = 5
    hegemons_allowed = yes
    months_for_exploration_spread = 300

    unique = {
        global_war_score_cost = -0.2
        subject_loyalty       = -30
    }

    modifier = {
        expected_army_size_modifier        = 16.0
        levy_maintenance_modifier          = 1.0
        lack_of_control_impact_on_warscore = -1.0
        construction_center_max_level      = 5
        global_navy_levy_size_modifier     = -2.0
        subject_pays_colonial_cost_modifier = 2.0
        antagonism_culture_influence       = 0.1
        antagonism_societal_value_influence = 0.2
        antagonism_government_type_influence = 0.25
    }

    mercenaries = -0.9
    war_score_from_battles = -0.8
    efficiency = 0.1
    victory_card = 5

    max_ai_privilege_per_estate = { nobles_estate = 0  clergy_estate = 0  ... }
    min_ai_privilege_per_estate = { nobles_estate = 0  clergy_estate = 0  ... }
}
```
*Example end state:* Armies ×16, prices volatile, mercenaries gone. All AI estate privileges stripped to 0 (revolutionary upheaval). `antagonism_*_influence` peaks, driving political conflict. Battles & levy costs at maximum; efficiency at minimum.
