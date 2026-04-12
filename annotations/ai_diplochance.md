# AI Diplomatic Chance
**Stage:** complete
**Keywords:** `action_key`, `base`, `opinion`, `trust_in_actor`, `rank_difference`, `different_religion`, `different_culture`, `war_exhaustion`, `in_debt`, `yesman`, `enforced_demand`, `relative_strength`, `border_distance`

> **System type: AI**

## Overview
`ai_diplochance` defines the weighted-score tables the AI uses to evaluate acceptance of diplomatic actions — marriage proposals, subject demands, loans, military access, peace offers, call-to-arms, and province trades. Each action has a named block of factor keys mapped to numeric weights; the AI sums all applicable factors to produce a final acceptance score. Positive values push toward acceptance, negative values push toward rejection. Extreme values (±1000 or larger) act as hard gates. Modders can tune individual weights to make the AI more or less receptive to specific diplomatic overtures.

## Vanilla File Locations

| File | Content |
|---|---|
| `in_game/common/ai_diplochance/00_ai_diplochance.txt` | All diplomatic action chance tables |
| `in_game/common/ai_diplochance/ai_diplochance.info` | **Non-functional / CK3 artifact.** Documents a custom localization key format using CK3 scope types (`character`, `faith`, `artifact`, etc.) — unrelated to the EU5 AI diplo system. Can be ignored. |

## Block Structure
```
action_key = {
    base = -25              # Starting score before any factors apply (omit = 0)
    yesman = 10000          # Override: if AI is in "yesman" mode, accept unconditionally

    # Relationship factors: opinion, positive_opinion, negative_opinion,
    #   trust_in_actor, royal_ties, royal_ties_with_target, allied_to_enemy
    opinion = 0.1

    # Difference factors: different_culture, same_culture, different_religion,
    #   different_religion_group, same_religion, culture_view, religion_view, same_court_language
    different_religion = -100

    # Political/power factors: rank_difference, relative_strength, competing_power,
    #   diplomatic_reputation, disloyal_subject, negative_stability
    rank_difference = -10

    # War/military factors: actor_at_war, recipient_at_war, levy_availability,
    #   low_manpower, war_exhaustion, recipient_occupied_beseiged_locations, recipient_civil_war
    war_exhaustion = 10

    # Economic factors: in_debt, price, price_percentage_of_treasury_funds,
    #   good_interest_rate, interest_rate_too_high, too_many_loans, loan_ends_too_soon
    in_debt = 1.0

    # War resolution factors (requestpeace only): warscore, months_at_war, peaceoffer,
    #   desperation, making_gains, demands_made, victory, enforced_demand
    enforced_demand = 10000

    # Geography (trade/access actions): border_distance, province_distance, lacks_border,
    #   capital, core, location_value, base_location_value
    border_distance = -0.1
}
```

## Key Fields Reference

| Factor | Typical range | Notes |
|---|---|---|
| `base` | −100 to 25 | Baseline before other factors; often negative (AI reluctant by default) |
| `yesman`, `enforced_demand` | 10000 | Hard override — effectively forces acceptance |
| `opinion`, `positive_opinion`, `negative_opinion` | −5 to 0.2 | Scaled by actual opinion value; negative_opinion is a separate downward factor |
| `trust_in_actor` | 0.25 to 2 | Scaled multiplier; also affects diplo rep loss in call-to-arms |
| `different_religion` / `different_religion_group` | −20 to −200 | Group difference is harsher than religion alone |
| `rank_difference` | −10 to −20 | Negative: higher-rank targets are harder to accept |
| `relative_strength` | 30 to 40 | AI more likely to comply when much weaker |
| `competing_power` | −100 | Hard block: will not comply with a direct rival power |
| `war_exhaustion`, `in_debt`, `low_manpower` | 0.5 to 10 | Distress factors — AI more pliable when in bad shape |
| `has_truce` / `betrayed_ally` | −10 to −1000 | Treaty/honor constraints; betrayed_ally is near-absolute |
| `border_distance`, `province_distance` | −0.025 to −2 | Fractional — scaled by actual distance value |
| `location_value` / `base_location_value` | ±25 to ±100 | Sign flips between `sell_location` (positive) and `buy_location` (negative) |

## Modding Notes

- **Score is additive:** all applicable factors sum to a final score. There is no explicit threshold visible in these files — the cut-off is handled by engine logic.
- **Hard gates via extreme values:** Use ±1000 or larger to create near-absolute blocks or near-certainties (e.g. `loan_is_insignificant = -1000`, `enforced_demand = 10000`).
- **Sign semantics in trade actions:** `sell_location` and `buy_location` invert signs on the same logical factors (e.g. `location_value = 100` in sell vs. `-100` in buy). This is intentional — a valuable province is a reason to sell high and resist buying.
- **Empty blocks:** `invite_to_international_organization`, `ask_to_join_international_organization`, and `create_international_organization_targetting` have no factors defined — the AI uses only engine defaults for those actions currently.
- **Load order:** file prefixed `00_`; mod overrides should use a higher prefix or redefine the same action key in a separate file.
- **`.info` file:** ignore for modding purposes — it is a CK3 custom-loc artifact and has no effect in EU5.

## Examples

### `callaction_offensive` — offensive call-to-arms (most complex, shows gate pattern)
```
callaction_offensive = {
    enforced_demand = 10000         # Overrides everything if applicable
    has_truce_with_target = -1000   # Hard block: won't join if truced with the war target
    betrayed_ally = -1000           # Hard block: won't join if previously betrayed

    levy_availability = -100        # Near-hard block if levies unavailable
    recipient_at_war   = -100       # Near-hard block if already at war
    recipient_civil_war = -100

    low_manpower       = -20
    negative_stability = -0.2
    war_exhaustion     = -2
    in_debt            = -5

    opinion            = 0.2        # Positive relationships increase willingness
    target_opinion     = -0.5       # Negative view of the war target reduces willingness
    royal_ties         = 0.25
    royal_ties_with_target = -1     # Family ties to the enemy reduces willingness
    promised_land      = 20

    lacks_border       = -10        # No border with the conflict area is a deterrent
    border_distance    = -0.1
    allied_to_enemy    = -20
}
```
*Key pattern:* Stacked hard gates (`-1000`) first eliminate impossible cases; large negatives (`-100`) cover near-impossible cases; then relationship and situational factors fine-tune the remainder.
