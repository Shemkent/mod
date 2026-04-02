# Subject Types
**Stage:** complete
**Keywords:** subject_type, subject_pays, level, overlord_modifier, subject_modifier, annexation_speed, annexation_min_opinion, diplo_chance_accept_subject, diplo_chance_accept_overlord, join_offensive_wars_always, join_defensive_wars_always, has_limited_diplomacy, great_power_score_transfer, institution_spread_to_overlord, institution_spread_to_subject, diplomatic_capacity_cost_scale

> **System type: Gameplay**

## Overview
Subject types define every distinct overlord–subject relationship in the game (vassal, march, tributary, dominion, trade company, HRE member, etc.). A subject type block specifies who can offer or receive the relationship, how wars are joined, how fast annexation proceeds, what modifiers each side gains, and how the AI weighs the deal. Subject types are the backbone of EU5's hierarchy system: they determine map colour, autonomy level, diplomatic capacity cost, and institution spread. Modders add subject types to introduce new vassal variants or colonial relationships, or modify existing ones to adjust payment, autonomy, or join-war behaviour.

## Vanilla File Locations
- `in_game/common/subject_types/` — one `.txt` file per subject type (16 files + `readme.txt`)
  - `vassal.txt`, `march.txt`, `tributary.txt`, `dominion.txt`, `trade_company.txt`, `hre.txt`, etc.
- `in_game/common/subject_military_stances/` — unmapped; separate stances that modify subject military behaviour
- Full file list in `_file_index.csv`.

## Block Structure
```pdx
<subject_type_id> = {
    # --- Identity ---
    level                 = <int>      # 0 (loose) → 3 (tight); drives annexation and autonomy
    color                 = <color_key>
    subject_pays          = <price_key>  # references a price script for monthly payment
    annullment_favours_required = <int>

    # --- Visibility and creation triggers ---
    visible_through_diplomacy  = { <trigger> }   # root=overlord, scope:target=subject
    enabled_through_diplomacy  = { <trigger> }
    visible_through_treaty     = { <trigger> }   # + scope:recipient, scope:war
    enabled_through_treaty     = { <trigger> }
    creation_visible           = { <trigger> }   # root=overlord
    subject_creation_enabled   = { <trigger> }   # root=overlord, scope:target_province=location
    release_country_enabled    = { <trigger> }   # root=overlord, scope:target=potential subject

    # --- War joining (root=subject_type, scope:actor=caller, scope:recipient=callee, scope:target=enemy) ---
    join_offensive_wars_always    = { <trigger> }
    join_offensive_wars_auto_call = { <trigger> }
    join_offensive_wars_can_call  = { <trigger> }
    join_defensive_wars_always    = { <trigger> }
    join_defensive_wars_auto_call = { <trigger> }
    join_defensive_wars_can_call  = { <trigger> }

    # --- Annexation ---
    can_be_annexed              = yes/no          # defaults yes
    annexation_speed            = <script value>  # per month; root/scope:actor=annexer
    annexation_min_years_before = <int>
    annexation_min_opinion      = <int>
    annexation_stall_opinion    = <int>

    # --- Modifiers ---
    overlord_modifier = { <modifier block> }
    subject_modifier  = { <modifier block> }
    great_power_score_transfer = <float>   # fraction of subject's GPS added to overlord

    # --- Institution spread ---
    institution_spread_to_overlord = <script_value_key>
    institution_spread_to_subject  = <script_value_key>

    # --- Diplomacy flags ---
    has_limited_diplomacy           = yes/no
    diplomatic_capacity_cost_scale  = <float>   # multiplier on how much diplo capacity this costs
    subject_can_cancel              = yes/no
    overlord_can_cancel             = yes/no
    minimum_opinion_for_offer       = <int>
    annulled_by_peace_treaty        = yes/no
    can_be_force_broken_in_peace_treaty = yes/no
    overlord_can_enforce_peace_on_subject = yes/no

    # --- Ruler / culture / religion inheritance ---
    has_overlords_ruler    = yes/no
    has_overlords_religion = yes/no
    only_overlord_culture               = yes/no
    only_overlord_or_kindred_culture    = yes/no
    only_overlord_court_language        = yes/no

    # --- Overlord build/recruit rights ---
    can_overlord_recruit_regiments = yes/no
    can_overlord_build_ships       = yes/no
    can_overlord_build_roads       = yes/no
    can_overlord_build_buildings   = yes/no
    can_overlord_build_rgos        = yes/no

    # --- Misc ---
    food_access              = yes/no
    fleet_basing_rights      = yes/no
    use_overlord_laws        = yes/no
    use_overlord_map_color   = yes/no
    use_overlord_map_name    = yes/no
    overlord_share_exploration  = yes/no
    overlord_protects_external  = yes/no  # defaults yes
    overlord_protects_other_subjects = yes/no  # defaults no
    will_join_independence_wars = yes/no
    can_change_rank          = yes/no
    can_change_heir_selection = yes/no
    allow_declaring_wars     = { <trigger> }   # root=subject_type

    # --- War cost ---
    war_score_cost   = { <script value> }
    base_antagonism  = { <script value> }
    monthly_favor_gain = { <script value> }

    # --- AI acceptance weights ---
    diplo_chance_accept_subject = {
        base = <int>
        current_strength = <float>
        border_distance  = <float>
        # ... (many named factors; see readme.txt)
    }
    diplo_chance_accept_overlord = { ... }
    ai_wants_to_be_overlord = { <effect script> }
    ai_wants_to_be_subject  = { <effect script> }

    # --- Lifecycle effects ---
    on_enable  = { <effect> }   # root=subject, scope:future_overlord=overlord
    on_disable = { <effect> }   # root=subject, scope:former_overlord=overlord
    on_monthly = { <effect> }   # root=subject
}
```

## Key Fields Reference
| Field | Purpose | Key constraint |
|---|---|---|
| `level` | Autonomy tier (0=loose, 3=tight) | Controls default annexation eligibility and UI ordering |
| `subject_pays` | Monthly payment from subject to overlord | References a named price script, not a raw value |
| `diplomatic_capacity_cost_scale` | How much this relation costs the overlord's diplo capacity | 0.2 for tributary (cheap), 1.0 for vassal/march |
| `annexation_min_opinion` | Minimum subject opinion of overlord for annexation to begin | Typically 150–175 for tight subjects |
| `annexation_stall_opinion` | Opinion floor below which annexation pauses | Set below `min_opinion`; defaults to ~125 |
| `great_power_score_transfer` | Fraction of subject's GPS credited to overlord | 0.25 for vassal/tributary, 0.5 for march/dominion |
| `institution_spread_to_overlord/subject` | Named script value for how fast institutions propagate | Use engine-defined keys (`monthly_institution_spread_mild`, `_severe`, `_weak`) |
| `join_offensive_wars_always` | Subject joins every offensive war automatically | Vassals always join unless they are themselves a subject of the callee |
| `has_limited_diplomacy` | Restricts subject's independent diplomatic actions | yes for vassal/march/dominion, no for tributary |
| `has_overlords_ruler` | Subject shares the overlord's ruler | Used for dominion (personal union-like); triggers regency on ruler death |
| `diplo_chance_accept_subject` | Named factor weights for AI acceptance of becoming subject | `base` is the starting score; most factors adjust by float per unit |
| `can_be_annexed` | Whether the subject can be annexed at all | `no` for march, tributary — these are perpetual relationships |

## Modding Notes
- **Level is the primary autonomy signal.** Level 0 (tributary) has full self-governance; level 3 (dominion) shares ruler and religion. New subject types should pick a level consistent with their intended autonomy.
- **`subject_pays` is a price script key**, not a raw number. Define the payment amount in a price script (or reference an existing one like `subject_pays_vassal`) to control monthly gold/resource flow.
- **`diplomatic_capacity_cost_scale` is not a script value** (for performance reasons). It must be a plain float literal.
- **`diplo_chance_accept_subject`** named factors are engine-defined tags (not arbitrary script). Only the documented factor names work; unknown keys are silently ignored.
- **`on_enable` / `on_disable` lifecycle effects** allow cultural/language conversions on subject creation (see `dominion.txt` which force-converts the subject's culture and court language). Be careful with irreversible effects in `on_enable`.
- **`visible_through_treaty` is only checked in peace deal contexts**, not the diplomacy screen. Both `visible_through_diplomacy` and `visible_through_treaty` should be set for subject types available via peace treaty.
- **`can_be_force_broken_in_peace_treaty`** controls whether an enemy can demand the subject relationship be broken. Defaults to yes — set no for types that must be permanent (e.g. `hre.txt`).
- **Map colour** is controlled by `color` (a named colour key) and `use_overlord_map_color`. Setting the latter to `yes` makes the subject blend into the overlord's map territory.
- **Cross-system:** subject types reference international organization membership in their `visible_through_diplomacy` triggers (e.g. Japanese shogunate exceptions). Changes to `international_organizations/` data can silently break subject type triggers.

## Example
`march.txt` — a militarised subject that grants the overlord full recruitment rights in exchange for strong combat modifiers:

```pdx
march = {
    subject_pays = subject_pays_march
    color        = subject_march
    level        = 1
    annullment_favours_required = 10

    join_offensive_wars_always = { always = yes }
    join_defensive_wars_always = { always = yes }

    has_overlords_ruler   = no
    can_be_annexed        = no         # marches are perpetual; no annexation path
    overlord_can_cancel   = yes

    can_overlord_recruit_regiments = yes
    can_overlord_build_ships       = yes
    can_overlord_build_roads       = yes
    can_overlord_build_buildings   = yes
    can_overlord_build_rgos        = yes

    has_limited_diplomacy          = yes
    diplomatic_capacity_cost_scale = 1.0

    great_power_score_transfer     = 0.5
    institution_spread_to_overlord = monthly_institution_spread_severe
    institution_spread_to_subject  = monthly_institution_spread_severe

    strength_vs_overlord = -1          # cannot rebel against overlord

    subject_modifier = {
        discipline       = 0.05
        global_defensive = 0.10
        fort_maintenance_cost = -0.10
        fort_limit       = 1
    }
    # ... diplo_chance and ai_ fields omitted for brevity
}
```

The march is level 1 (looser than vassal at level 2), cannot be annexed, but fully joins all wars and grants the overlord total build/recruit access in march territory.
