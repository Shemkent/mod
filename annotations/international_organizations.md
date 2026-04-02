# International Organizations
**Stage:** complete
**Keywords:** international_organization, io_, special_status, payment, land_ownership_rule, parliament_type, resolution, join_defensive_wars_always, join_offensive_wars_always, leader, modifier, leader_modifier, non_leader_modifier, target_modifier, has_parliament, unique, has_target, variables, payments_implemented, special_statuses_implemented

> **System type: Gameplay**

## Overview
International organizations (IOs) are shared multi-country entities that sit above bilateral diplomacy — the HRE, the Papacy, merchant leagues, military alliances, jihad organizations, and many others. Each IO type defines membership rules, leader selection, war-joining behaviour, modifiers for members and leader, optional land ownership, an optional parliament, and monthly payments between members. Three companion folders extend the type definition: `_payments/` (monthly resource transfers), `_special_statuses/` (per-country titles and roles within an IO, such as HRE Elector or Emperor, with their own triggers, modifiers, and lifecycle effects), and `_land_ownership_rules/` (IO-held territory rules). Modders creating new military leagues, tribute systems, or trade federations work primarily in these four folders.

## Vanilla File Locations
- `in_game/common/international_organizations/` — one `.txt` per IO type (~35 files + readme)
- `in_game/common/international_organization_payments/` — payment definitions (hre.txt, tatar_yoke_payment.txt, etc.)
- `in_game/common/international_organization_payments/readme.txt`
- `in_game/common/international_organization_land_ownership_rules/` — land attachment rules (~8 files + readme)
- `in_game/common/international_organization_special_statuses/` — special status definitions (~10 files + readme)
- Full file list in `_file_index.csv`.

## Block Structure

### International Organization Type
```pdx
<io_type_id> = {
    # --- Structure ---
    unique              = yes/no        # only one instance can exist globally
    has_target          = yes/no        # IO has an enemy target country
    has_leader_country  = yes/no        # IO has a designated leader (default no)
    has_enemies         = yes/no        # IO can have enemy countries

    # --- Leader ---
    leader_type                  = none/country/character
    leader_title_key             = "LOC_KEY"
    use_regnal_number            = yes/no
    override_ruler_title         = yes/no
    title_is_suffix              = yes/no
    leader_change_trigger_type   = none/rulerchange/timed
    leader_change_method         = rotation/vote/lottery/score/none
    months_between_leader_changes = <int>
    leadership_election_resolution = <resolution_key>
    disband_if_no_leader         = yes/no
    leader = { <effect — add_to_list = leaders on candidate scopes> }
    can_lead_trigger = { <trigger> }    # root = potential leader
    leader_score = { <script value> }   # root = potential leader, recipient = IO

    # --- Membership triggers ---
    can_join_trigger     = { <trigger> }   # root = joiner, actor = leader, recipient = IO
    can_leave_trigger    = { <trigger> }
    auto_leave_trigger   = { <trigger> }
    auto_disband_trigger = { <trigger> }
    create_visible_trigger  = { <trigger> }
    create_enabled_trigger  = { <trigger> }
    invite_visible_trigger  = { <trigger> }
    invite_enabled_trigger  = { <trigger> }
    join_visible_trigger    = { <trigger> }
    join_enabled_trigger    = { <trigger> }
    can_target_trigger      = { <trigger> }   # root = potential enemy
    can_be_enemy_trigger    = { <trigger> }   # root = enemy, recipient = IO
    potential_target_trigger = { <trigger> }  # for alert UI

    # --- War joining (root = IO, scope:actor/recipient = caller/callee) ---
    join_defensive_wars_always    = { <trigger> }
    join_defensive_wars_auto_call = { <trigger> }
    join_defensive_wars_can_call  = { <trigger> }
    join_offensive_wars_always    = { <trigger> }
    join_offensive_wars_auto_call = { <trigger> }
    join_offensive_wars_can_call  = { <trigger> }
    only_leader_country_joins_defensive_wars = yes/no   # performance optimisation
    only_leader_country_joins_offensive_wars = yes/no
    can_declare_war = { <trigger> }   # attacker = attacker, defender = defender, recipient = IO

    # --- Access rights ---
    has_military_access  = { <trigger> }    # root = IO, actor = asker, recipient = owner
    fleet_basing_rights  = { <trigger> }
    gives_military_access_to_all_when_at_war = yes/no   # optimised bulk access
    gives_food_access_to_members = yes/no
    can_recruit_regiments_in_members = yes/no
    can_build_ships_in_members       = yes/no
    can_build_roads_in_members       = yes/no
    can_build_buildings_in_members   = yes/no
    can_build_rgos_in_members        = yes/no

    # --- Modifiers (scale blocks — modifier fields multiplied by a script-value scale; root = country, scope:recipient = IO) ---
    modifier                        = { <modifier block> }  # all members
    leader_modifier                 = { <modifier block> }  # leader only
    non_leader_modifier             = { <modifier block> }  # non-leader members
    target_modifier                 = { <modifier block> }  # enemy target

    # --- Parliament ---
    has_parliament       = yes/no
    parliament_type      = <parliament_type_key>
    resolution_widget    = <gui_widget_key>
    max_active_resolutions = <int>
    can_initiate_policy_votes = { <trigger> }   # root = country, recipient = IO

    # --- Annexation within IO ---
    allow_member_annexation = yes/no
    annexation_min_years_before = { <script value> }
    can_annex_members    = { <trigger> }
    can_annex_visible    = { <trigger> }
    annexation_speed     = { <script value> }

    # --- Finance ---
    payments_implemented = { <payment_id> ... }
    <currency_type>      = yes/no   # treasury holding; valid types include gold, manpower — check modifier definitions for the full list

    # --- Special statuses ---
    special_statuses_implemented = { <special_status_id> ... }

    # --- Internal variables ---
    variables = {
        <var_name> = {
            format         = "<display_string>"
            change_format  = "<display_string>"
            monthly_change = { <script value> }
            start          = { <script> }
            min            = <float>
            max            = <float>
            hidden         = yes/no
            monthly_change_hidden = yes/no
        }
    }

    # --- Land ownership ---
    land_ownership_rule = <land_ownership_rule_key>

    # --- Lifecycle ---
    on_creation  = { <effect> }   # root = IO, actor = creator, target = target
    on_disband   = { <effect> }
    on_joined    = { <effect> }   # root = country, scope:recipient = IO
    on_left      = { <effect> }
    monthly_effect = { <effect> } # root = IO

    # --- Expulsion ---
    expel_members_who_are_targets_of_other_members = yes/no
    expel_members_who_target_the_leader            = yes/no
    expel_members_who_are_attackers_at_war_with_other_members = yes/no
    expel_members_who_are_defenders_at_war_with_other_members = yes/no

    # --- Opinion / trust ---
    opinion_bonus   = <float>
    opinion_trust   = <float>
    min_opinion     = <float>   # IO disbands if violated — use cautiously
    min_trust       = <float>
    antagonism_towards_leader_modifier = <float>
    antagonism_modifier_for_taking_land_from_fellow_member   = <float>
    antagonism_modifier_for_taking_land_from_member_as_outsider = <float>
    no_cb_price_modifier_for_fellow_member = <float>

    # --- Map ---
    show_on_diplomatic_map  = yes/no
    show_as_overlord_on_map_trigger = { <trigger> }
    map_color_override          = { <script color> }
    secondary_map_color_override = { <script color> }
    leader_color   = ...
    member_color   = ...
    target_color   = ...

    # --- UI ---
    background_texture = <path>
    should_show_ruler_history = yes/no
    show_strength_comparison_with_target = yes/no
    show_leave_message = yes/no
    custom_name = <customizable_localization_key>
    disband_message_trigger = { <trigger> }

    # --- Other ---
    subject_limited              = yes/no   # can limited-diplomacy subjects participate?
    can_invite_countries         = yes/no
    use_laws_as_join_reason      = yes/no
    annulled_by_peace_treaty     = yes/no
    annullment_favours_required  = <int>    # note: "annullment" is the vanilla spelling (double-l); match it exactly
    has_dynastic_power           = yes/no
    declare_war_on_target_casus_belli = <cb_id>

    # --- AI ---
    ai_desire_to_join              = { <script value> }
    ai_desire_to_allow_new_member  = { <script value> }
    ai_desire_to_attack_other_members = { <script value> }
    ai_issue_voting_bias           = { <script value> }
}
```

### Payment Definition
```pdx
<payment_id> = {
    get_payer_list = { <effect — add_to_local_variable_list = { name = payers target = this }> }
    get_payee_list = { <effect — add_to_local_variable_list = { name = payees target = this }> }
    price             = <price_key>
    price_multiplier  = { <script value> }    # root = IO
    uses_maintenance  = yes/no
    maintenance_modifier = {                  # repeatable; penalises/rewards based on slider
        potential_trigger = { <trigger> }
        scale = { <script value> }
        <modifier fields>
    }
    proportion_for_payer = { <script value> } # root = payer country, scope:recipient = IO
    proportion_for_payee = { <script value> }
    min_slider_value  = <float>               # floor of maintenance slider (default 0.5)
}
```

### Special Status Definition
```pdx
<status_id> = {
    priority          = <int>         # higher = shown first in UI
    max_countries     = { <script value> }   # root = IO
    can_bestow_trigger = { <trigger> }       # root = country, scope:recipient = IO
    auto_bestowal_trigger = { <trigger> }
    auto_rescind_trigger  = { <trigger> }
    on_bestowed_effect = { <effect> }
    on_rescinded_effect = { <effect> }
    modifier          = { <modifier block> }          # applied to status-holder
    leader_modifier   = { <modifier block> }          # applied to leader, scaled by count
    map_color         = <scripted color>
    special_status_power = { <script value> }         # voting power in IO parliament
    leader = yes/no                   # this status designates the leader
    elector = yes/no                  # this status participates in elections
}
```

### Land Ownership Rule
```pdx
<rule_id> = {
    modifier               = { <modifier block> }   # applied to IO-owned locations
    can_add_trigger        = { <trigger> }   # root = IO
    can_remove_trigger     = { <trigger> }
    can_add_location_trigger    = { <trigger> }   # root = location, recipient = IO
    can_remove_location_trigger = { <trigger> }
    on_added               = { <effect> }    # root = location, recipient = IO
    on_removed             = { <effect> }
    ai_desire_to_add       = { <script value> }
    owned_location_color   = <scripted color>
}
```

## Key Fields Reference
| Field | Purpose | Key constraint |
|---|---|---|
| `unique` | Only one instance of this type can exist globally | Without this, multiple instances (e.g. leagues) can form |
| `has_leader_country` | Enables a leader mechanism; defaults no | Must be yes for election/rotation/score methods to work |
| `leader_change_method` | How leadership transfers | `vote` requires `leadership_election_resolution`; `score` requires `leader_score` |
| `variables` | Named variables stored on the IO instance | Displayed in the IO panel; can track dynamic state (e.g. imperial authority) |
| `payments_implemented` | Which payment definitions are active at IO creation | Payments can also be toggled via laws/policies after creation |
| `special_statuses_implemented` | Which special statuses are available at IO creation | Status bestowals use `auto_bestowal_trigger` on each member monthly |
| `land_ownership_rule` | Links to a rule in `_land_ownership_rules/`; enables IO to hold territory | Without this, IO has no territorial ownership |
| `only_leader_country_joins_defensive_wars` | Performance optimisation: only leader is checked for war calls | Set yes for large IOs (HRE) to avoid checking every member |
| `min_opinion` / `min_trust` | Floor below which IO disbands | Expensive on large IOs — use with caution |
| `has_enemies` | Whether the IO can have designated enemy countries (distinct from having a `has_target` country) | Enables `add_enemy_to_international_organization` effect and enemy-related triggers |
| `modifier` vs `leader_modifier` vs `non_leader_modifier` | Member-wide vs leader-only vs non-leader modifiers | All are scale blocks: modifier fields are multiplied by a script-value scale attached to each modifier definition |
| `special_status_power` | Voting weight in IO parliament per-status | Only relevant when `has_parliament = yes` |

## Modding Notes
- **IO type vs IO instance:** The files define _types_. To make an IO appear in a game, create an instance via `create_international_organization` in a history file, an on-action, or an event effect. Defining the type alone has no effect.
- **`payments_implemented` and `special_statuses_implemented`** list which payment/status IDs are active at creation time. These IDs must exist in their respective companion folders. Laws and policies can activate more during gameplay.
- **`modifier`, `leader_modifier`, `non_leader_modifier`** are scale blocks, not flat modifier blocks. Each modifier field is multiplied by a script-value scale defined inside the modifier itself. Most vanilla IO modifiers use a flat scale of `1`, making them behave like ordinary modifiers in practice.
- **`auto_bestowal_trigger`** and **`auto_rescind_trigger`** on special statuses fire monthly for every member. Keep them cheap.
- **`has_dynastic_power = yes`** enables influential dynasties to exert power within the IO (used by HRE for Habsburg dominance mechanics). Requires supporting scripted infrastructure.
- **`can_initiate_policy_votes`** controls which members can bring policies to a vote, separate from who can vote. Not the same as `can_lead_trigger`.
- **Cross-system:** IO membership is tested in subject type triggers (see `subject_types.md`), casus belli triggers, laws, and government reforms. Changing IO membership rules can break chains in those systems.
- **Land ownership (`international_organization_land_ownership_rules/`)** only enables the concept of IO-held territory. The actual mechanics (adding/removing locations, what the territory does) are driven by the rule definition and by scripted effects that call `add_location_to_international_organization`.

## Example
Simplified HRE IO block showing key structural patterns:

```pdx
hre = {
    unique              = yes
    has_target          = no
    has_leader_country  = yes
    has_parliament      = yes
    parliament_type     = hre_court_assembly
    has_dynastic_power  = yes

    leader_type                = character   # Emperor is a character, not a country
    leader_title_key           = "HRE_LEADER"
    leader_change_trigger_type = rulerchange
    leader_change_method       = vote
    leadership_election_resolution = hre_election
    disband_if_no_leader       = no          # can survive a vacancy

    only_leader_country_joins_defensive_wars = yes   # performance: only Emperor is checked

    join_defensive_wars_auto_call = {
        scope:target ?= {
            not = { is_member_of_international_organization = root }
        }
        # ... (members auto-called against non-members)
    }

    payments_implemented = { imperial_contribution imperial_treasury_contribution imperial_army_contribution }
    special_statuses_implemented = { emperor elector archbishop_elector free_city imperial_prince ... }

    land_ownership_rule = hre_land_ownership

    show_as_overlord_on_map_trigger = { always = yes }
    show_leave_message = no
    use_laws_as_join_reason = no
}
```

The HRE's `payments_implemented` list references `in_game/common/international_organization_payments/hre.txt`; its `special_statuses_implemented` reference `in_game/common/international_organization_special_statuses/hre.txt`. The parliament type is defined in `common/parliament_types/`.
