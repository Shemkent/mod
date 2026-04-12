# Scripted Relations
**Stage:** complete
**Keywords:** `relation_key`, `type`, `relation_type`, `uses_diplo_capacity`, `block_when_at_war`, `break_on_war`, `break_on_becoming_subject`, `disallow_war`, `military_access`, `fleet_basing_rights`, `called_in_defensively`, `called_in_offensively`, `favors_to_first`, `gold_to_first`, `offer_enabled`, `cancel_enabled`, `will_expire_trigger`, `wants_to_give`, `giving_modifier_scale`, `offer_effect`

> **System type: Gameplay**

## Overview
Scripted relations define bilateral diplomatic relationships that countries can hold with each other — alliances, guarantees, trade agreements, military access, espionage pacts, and more. Each relation specifies its type, directionality, mechanical effects (access rights, war behaviour, monthly resource flows), visibility and availability conditions, AI evaluation weights, and lifecycle effects. One file per relation is the convention; all `.txt` files in the folder are loaded. Relations are toggled on/off through `country_interactions` or scripted effects.

## Vanilla File Locations

| File | Content |
|---|---|
| `in_game/common/scripted_relations/readme.txt` | Authoritative field reference |
| `in_game/common/scripted_relations/alliance.txt` | Mutual alliance with war-joining, military access, favour flows |
| `in_game/common/scripted_relations/guarantee.txt` | One-way security guarantee |
| `in_game/common/scripted_relations/military_access.txt` | One-way military passage rights |
| `in_game/common/scripted_relations/trade_access.txt` | Market trade access |
| `in_game/common/scripted_relations/embargo_nation.txt` | Trade embargo |
| … *(31 content files total)* | One file per relation type |

## Block Structure

```pdxscript
relation_key = {
    # --- Identity ---
    type          = diplomacy          # diplomacy / subject / union
    relation_type = mutual             # mutual / oneway

    # --- Diplomatic slot usage ---
    uses_diplo_capacity          = mutual   # none / mutual / giving / receiving
    diplomatic_capacity_cost     = <script_value>

    # --- Lifecycle triggers ---
    block_when_at_war            = yes/no
    break_on_war                 = yes/no
    break_on_becoming_subject    = yes/no
    break_on_not_spying          = yes/no
    annulled_by_peace_treaty     = yes/no
    annullment_favours_required  = <int>

    # --- Access & war rights ---
    disallow_war          = yes/no
    embargo               = yes/no
    military_access       = yes/no
    fleet_basing_rights   = yes/no
    food_access           = yes/no
    lifts_fog_of_war      = yes/no
    lifts_trade_protection = yes/no
    is_exempt_from_sound_toll  = yes/no
    is_exempt_from_isolation   = yes/no
    block_building             = yes/no
    called_in_defensively = none/mutual/giving/receiving
    called_in_offensively = none/mutual/giving/receiving

    # --- Monthly resource flows (script values; scope:first / scope:second) ---
    favors_to_first              = <script_value>
    favors_to_second             = <script_value>
    gold_to_first                = <script_value>
    gold_to_second               = <script_value>
    trade_to_first               = <script_value>
    trade_to_second              = <script_value>
    institution_spread_to_first  = <script_value>
    institution_spread_to_second = <script_value>

    # --- Costs ---
    diplomatic_cost              = <price>
    war_declaration_cost         = <price>
    buy_price                    = <price>
    monthly_ongoing_price_first_country  = <price>
    monthly_ongoing_price_second_country = <price>

    # --- Visibility / availability triggers ---
    visible          = { <triggers> }
    offer_visible    = { <triggers> }
    request_visible  = { <triggers> }
    cancel_visible   = { <triggers> }
    break_visible    = { <triggers> }
    offer_enabled    = { <triggers> }
    request_enabled  = { <triggers> }
    cancel_enabled   = { <triggers> }
    break_enabled    = { <triggers> }
    will_expire_trigger = { <triggers> }
    should_ai_offer_trigger = { <triggers> }

    # --- AI evaluation ---
    wants_to_give             = { <ai_evaluation> }
    wants_to_receive          = { <ai_evaluation> }  # oneway only
    wants_to_give_diplo_chance    = { <diplo_evaluation> }
    wants_to_receive_diplo_chance = { <diplo_evaluation> }
    wants_to_keep             = { <ai_evaluation> }
    wants_to_keep_diplo_chance    = { <diplo_evaluation> }
    show_break_alert          = yes/no

    # --- Modifier scale (applied while relation is active) ---
    giving_modifier_scale    = { <script_math> }   # scope:first = giver, scope:second = receiver
    receiving_modifier_scale = { <script_math> }
    mutual_modifier_scale    = { <script_math> }

    # --- Lifecycle effects ---
    offer_effect            = { <effects> }
    request_effect          = { <effects> }
    cancel_effect           = { <effects> }
    break_effect            = { <effects> }
    offer_declined_effect   = { <effects> }
    request_declined_effect = { <effects> }
    expire_effect           = { <effects> }

    # --- Passive modifiers (applied while relation is active) ---
    # Key naming conventions — no block wrapper needed:
    # <modifier_key>             → applied to both (mutual)
    # giving_<modifier_key>      → applied to the giving country
    # receiving_<modifier_key>   → applied to the receiving country
    # opinion_<modifier_key>     → opinion bonus for mutual
    # opinion_giving_<modifier_key>
    # opinion_receiving_<modifier_key>
    # trust_<modifier_key>       → trust bonus for mutual
    # trust_giving_<modifier_key>
    # trust_receiving_<modifier_key>

    # --- UI ---
    sound         = "<sound_gfx>"
    mutual_color  = <color>
    giving_color  = <color>
    receiving_color = <color>
    is_ongoing    = yes/no
    texture_file  = "<path>"
    progress      = <script_value>
}
```

## Key Fields Reference

| Field | Purpose | Notes |
|---|---|---|
| `type` | Scope of valid partners | `diplomacy` = any country; `subject` = subject only; `union` = union partners only |
| `relation_type` | Directionality | `mutual` = both sides equal; `oneway` = giver/receiver asymmetry |
| `uses_diplo_capacity` | Diplomatic slot consumption | `none/mutual/giving/receiving` |
| `block_when_at_war` | Prevents forming the relation while at war | Does not break an existing relation; use `break_on_war` for that |
| `break_on_war` | Auto-breaks if parties go to war | Most friendly relations use `yes` |
| `break_on_becoming_subject` | Auto-breaks if one party becomes a subject | Important edge case — subjects gain a new overlord relationship that can conflict |
| `disallow_war` | Prevents declaration of war between parties | Used by alliances and truces |
| `called_in_defensively/offensively` | War-joining scope | `none/mutual/giving/receiving` |
| `favors_to_first/second` | Monthly favour flows | Script values resolved in `scope:first` / `scope:second` |
| `giving_/receiving_modifier_scale` | Scale passive modifiers while active | `scope:first` = giver |
| `will_expire_trigger` | Auto-expiry condition | Checked periodically; fires `expire_effect` when true |
| `wants_to_give/keep` | AI decision weight | Negative result → AI tries to cancel/break |

## Modding Notes
- One file per relation is convention but not enforced — all `.txt` files are loaded.
- Passive modifiers use flat key naming (`giving_<modifier>`) rather than a block — see readme.txt for the full naming convention.
- `type = subject` relations are only available between overlord and subject; attempting to form them between non-subject countries silently fails.
- `wants_to_give_diplo_chance` uses a predefined set of named diplo-chance values (full list in readme.txt) rather than arbitrary triggers — this is a different evaluation system from standard triggers.
- Duplicate relation keys: last-loaded wins; no merge.
- `scope:first` / `scope:second` are bound throughout lifecycle effects and modifier scaling; `scope:first` = offering/giving country, `scope:second` = receiving country.

## Example

```pdxscript
# alliance.txt (simplified)
alliance = {
    type          = diplomacy
    relation_type = mutual
    uses_diplo_capacity       = mutual
    break_on_war              = yes
    break_on_becoming_subject = yes
    disallow_war              = yes
    military_access           = yes
    fleet_basing_rights       = yes
    lifts_fog_of_war          = yes
    called_in_defensively     = mutual
    called_in_offensively     = mutual

    favors_to_first = {
        value  = 1
        add    = "scope:first.relative_military_strength(scope:second)"
        divide = 12
    }
    institution_spread_to_first  = monthly_institution_spread_mild
    institution_spread_to_second = monthly_institution_spread_mild

    offer_enabled = { alliance_can_exist = yes }
    cancel_enabled = {
        scope:actor = { NOT = { is_fighting_war_together_with = scope:recipient } }
    }
    will_expire_trigger = { alliance_can_exist = no }
    show_break_alert = yes
}
```
