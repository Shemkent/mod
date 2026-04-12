# Societal Values
**Stage:** complete
**Keywords:** societal_value, left_modifier, right_modifier, opinion_importance_multiplier, age, allow, monthly_towards, societal_value_monthly_move, societal_value_minor_monthly_move, societal_value_importance_modifier, bias_for

> **System type: Gameplay**

## Overview
Societal values are bidirectional sliders that represent a country's political, economic, military, and cultural orientation. Each value is a named axis with two opposing ends — `left_modifier` applies while the country leans left; `right_modifier` applies while it leans right. Modifiers from both sides are blended proportionally to the slider position, so a country at the midpoint receives half the effect from each side. Some values are gated behind a specific age (`age` field) or a country trigger (`allow` block). Values are moved monthly by laws, traits, employment systems, events, and `monthly_towards_*` modifiers. In vanilla, 16 values are defined across several thematic clusters.

## Vanilla File Locations
- `in_game/common/societal_values/00_default.txt` — all 14 value definitions (one file)
- Full file list in `_file_index.csv`.

## Block Structure
```pdx
<value_id> = {
    # --- Availability ---
    age   = <age_id>        # optional; value only exists from this age onward
    allow = { <trigger> }   # optional; country trigger; value is hidden/inactive if not met

    # --- Modifiers ---
    left_modifier  = { <modifier block> }   # applied at full strength at the left extreme
    right_modifier = { <modifier block> }   # applied at full strength at the right extreme
    # Both modifiers are blended by slider position; midpoint = 50% each

    # --- Opinion ---
    opinion_importance_multiplier = <float>   # scales how much this value affects AI opinion of others
}
```

## Vanilla Values
| Value ID | Left end | Right end | Notes |
|---|---|---|---|
| `centralization_vs_decentralization` | Centralized (crown power, control) | Decentralized (estate loyalty, subject loyalty) | |
| `traditionalist_vs_innovative` | Traditionalist (cultural tradition, stability) | Innovative (literacy, institutions) | |
| `spiritualist_vs_humanist` | Spiritualist (conversion, clergy) | Humanist (tolerance, assimilation) | |
| `aristocracy_vs_plutocracy` | Aristocracy (discipline, nobles) | Plutocracy (trade, burghers) | |
| `serfdom_vs_free_subjects` | Serfdom (raw output, levy, nobles) | Free subjects (pop promotion, enfranchisement) | |
| `mercantilism_vs_free_trade` | Mercantilism (protection, home market) | Free trade (merchants, global trade) | Age 4+ |
| `belligerent_vs_conciliatory` | Belligerent (war score, CB speed, aggression) | Conciliatory (diplomacy, reputation) | |
| `quality_vs_quantity` | Quality (tactics, morale) | Quantity (frontage, food) | |
| `offensive_vs_defensive` | Offensive (siege, assault) | Defensive (forts, combat speed) | |
| `land_vs_naval` | Land (RGO, land trade) | Naval (maritime, sea trade, colonies) | |
| `capital_economy_vs_traditional_economy` | Capital (production, banking) | Traditional (raw output, population, food) | |
| `individualism_vs_communalism` | Individualism (morale) | Communalism (estates, pop stability) | |
| `outward_vs_inward` | Outward (colonialism, diplomacy) | Inward (control, cultural tradition) | Age 3+ |
| `sinicized_vs_unsinicized` | Sinicized (governance, trade) | Unsinicized (prestige, stability) | `allow`: NOT chinese culture group AND one of: subject of `c:CHI`, OR capital in east/south-east Asia sub-continent, OR member of Middle Kingdom IO |
| `absolutism_vs_liberalism` | Absolutism (crown power, privileges) | Liberalism (cultures capacity, parliament) | Age 5+ |
| `mysticism_vs_jurisprudence` | Mysticism (morale, conversion) | Jurisprudence (research, cabinet) | `allow`: requires `allow_mysticism_vs_jurisprudence` modifier |

## Key Fields Reference
| Field | Purpose | Key constraint |
|---|---|---|
| `left_modifier` | Modifiers when slider leans left | Blended proportionally; full strength at far left, zero at far right |
| `right_modifier` | Modifiers when slider leans right | Blended proportionally; full strength at far right, zero at far left |
| `opinion_importance_multiplier` | Scales opinion impact between countries with divergent values | Higher = this value causes more AI antagonism when countries differ; default appears to be ~1.0 if omitted |
| `age` | Age gate for the value's existence | Value does not appear on the slider panel until the specified age begins |
| `allow` | Country-level trigger for value applicability | Value hidden and inactive for countries not meeting this trigger |
| `monthly_towards_<side_name>` | Modifier key used elsewhere to move this slider | The suffix is the value ID's side name (e.g., `monthly_towards_centralization`, `monthly_towards_decentralization`), not the literal word "left" or "right"; used in traits, employment systems, laws, events |

## Modding Notes
- **Slider blending is continuous.** A country at position 0.75 (leaning left) receives 75% of `left_modifier` and 25% of `right_modifier`. Modifiers from both ends apply simultaneously at reduced strength.
- **Adding a new societal value** requires a new block in `00_default.txt`, localization for both ends and the axis, and surface points (laws, traits, or events that push `monthly_towards_<left_id>` or `monthly_towards_<right_id>`). Without surface points, the slider will never move.
- **`monthly_towards_*` modifier keys** are derived from the value ID's side names. For `centralization_vs_decentralization`, the keys are `monthly_towards_centralization` and `monthly_towards_decentralization`; for `serfdom_vs_free_subjects`, they are `monthly_towards_serfdom` and `monthly_towards_free_subjects`. Verify exact key names against the employment systems source (`employment_systems.md`) or event files that use these modifiers.
- **`age`-gated values** are hidden and have no effect until the specified age begins globally. Countries cannot move the slider before that age. Plan mod content that surfaces these values accordingly.
- **`allow` blocks** (e.g., `sinicized_vs_unsinicized`) make the value conditional on country state. Non-qualifying countries do not see the slider and cannot accumulate value in either direction. The trigger is re-evaluated; if a country later meets the condition, the slider initializes at the midpoint.
- **`opinion_importance_multiplier`** is used by the AI antagonism system. A value of 0.1 (e.g., `land_vs_naval`) means divergence on this axis has a minor effect on AI opinions. Values of 0.5 (e.g., `centralization_vs_decentralization`, `quality_vs_quantity`) create moderate opinion effects. Most values omit this field, implying a game default.
- **Cross-system:** societal value positions are tested via `societal_value:value_id` script values in `ai_will_do` blocks (employment systems, laws). Traits push values via `monthly_towards_*` modifiers. The `bias_for_*_policies` modifier keys feed into the law adoption AI.

## Example
`traditionalist_vs_innovative` — governs cultural tradition vs. institutional adoption:

```pdx
traditionalist_vs_innovative = {
    left_modifier = {                          # Traditionalist
        global_estate_target_satisfaction = small_permanent_target_satisfaction
        cultural_tradition_modifier = 1.0
        stability_cost = -0.20
        embrace_institution_cost_modifier = 2.0  # institutions very expensive
        language_change_threshold_modifier = 1
        societal_value_importance_modifier = 0.2
        institution_importance_modifier = -0.2
        bias_for_scholar_policies = -100
    }
    right_modifier = {                         # Innovative
        global_max_literacy = 10
        cultural_influence_modifier = 1.0
        stability_cost = 0.20
        embrace_institution_cost_modifier = -0.5  # institutions discounted
        language_change_threshold_modifier = -1
        societal_value_importance_modifier = -0.2
        institution_importance_modifier = 0.2
        bias_for_scholar_policies = 100
    }
}
```

A traditionalist country pays half the stability cost but double the institution embrace cost. An innovative country gets literacy bonuses and cheap institutions at the cost of more expensive stability. The `societal_value_importance_modifier` entries cause this slider's own position to amplify or dampen how much other societal values move.
