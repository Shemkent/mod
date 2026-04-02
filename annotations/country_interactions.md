# Country Interactions
**Stage:** complete
**Keywords:** country_interaction, type, select_trigger, potential, allow, effect, reject_effect, ai_will_do, diplo_chance, diplomatic_cost, cooldown, ai_tick, price, price_modifier, block_when_at_war, use_enroute, category, show_in_gui_list

> **System type: Gameplay**

## Overview
Country interactions are scripted diplomatic or subject-management actions one country can perform toward another (or on itself). Each block defines who can see and use the action, what must be selected as a target, what it costs, and what happens when accepted or rejected. They include requesting military access, offering alliances, enforcing religion on subjects, demanding money, and breaking unions. Interactions differ from generic actions because they target another country (`type = diplomacy` or `type = subject`) rather than acting on oneself or a location. The `diplomatic_costs/` and `insults/` sub-folders extend this system.

## Vanilla File Locations
- `in_game/common/country_interactions/` — one or a few `.txt` files per interaction category (~70+ files)
- `in_game/common/country_interactions/readme.txt` — field documentation and AI diplo chance tag list
- `in_game/common/diplomatic_costs/` — defines diplomatic cost types referenced by `diplomatic_cost =`
- `in_game/common/insults/` — insult interactions (separate sub-system)
- Full file list in `_file_index.csv`.

## Block Structure
```pdx
<interaction_id> = {
    # --- Type & presentation ---
    type     = subject/diplomacy/union  # scope context for the action
    category = <CATEGORY_LOC_KEY>       # UI grouping on the interaction panel
    sound    = <sound_key>
    show_in_gui_list = no               # omit from auto-generated GUI lists

    # --- Cost & execution ---
    price              = <price_key>    # e.g. price:hire_advisor or scope:target.price
    price_modifier     = { <script value> }  # multiplier on price
    payer              = { <script> }   # who pays (default: actor)
    payee              = { <script> }   # who receives (default: nobody)
    diplomatic_cost    = <cost_id>      # spy network / trust / favor cost type
    diplomatic_cost_modifier = { <script value> }

    # --- Timing ---
    use_enroute        = yes/no         # uses diplomat travel time
    block_when_at_war  = yes/no         # defaults yes
    cooldown = {
        type   = <tag>
        days/weeks/months/years = <int>
    }

    # --- Target selection (repeatable) ---
    select_trigger = {
        looking_for_a   = character/location/province/area/region/country/value/boolean
        target_flag     = <scope_name>  # defaults to target, target_1, target_2...
        source          = actor/recipient/target/world/knowncountries/inrange/rivals/subjects/atwar
        source_ai_override = <source>
        source_flags    = <flags>       # neighbor, adjacent_locations, border, etc.
        source_flags_ai_override = <flags>
        source_global_list = <list_name>
        interaction_source_list    = { <effect> }
        ai_interaction_source_list = { <effect> }
        pre_evaluation_sort_value  = { <script value> }
        pre_evaluation_number_to_evaluate_fully = <int>
        max_targets_for_ui = <int>
        cache_targets              = yes/no
        cache_interaction_source_list = yes/no
        cache_order                = yes/no
        name           = <loc_key>
        allow_null     = yes/no
        allow_self     = yes/no
        move_to_next_section_on_click = yes/no
        column = { data = <col_id> width = <int> icon = <path> }
        default_sort   = <sort_key>
        none_available_msg_key = <loc_key>
        show_why_not_visible = yes/no
        show_why_not_enabled = yes/no
        show_if = { <trigger> }
        visible  = { <trigger> }
        enabled  = { <trigger> }
        selected = { <trigger> }
        map_mode = <map_mode>
        map_color = { <script color> }
    }

    # --- Eligibility ---
    potential = { <trigger> }   # scope:actor = acting country
    allow     = { <trigger> }   # scope:actor, scope:recipient, scope:target...

    # --- AI ---
    accept    = { <script value> }    # AI acceptance score (deprecated; use diplo_chance)
    diplo_chance = {
        base = <int>
        <named_factor> = <float>  # see readme.txt for full factor list
    }
    ai_tick           = never/daily/monthly
    ai_tick_frequency = { <script value> }
    ai_limit_per_check = <int>     # max countries AI targets per check
    ai_will_do        = { <script value> }   # scores how much AI wants to use this; scope:actor, scope:recipient, scope:target
    ai_prerequisite   = { <trigger> }        # cheap early-out for AI (scope:actor only)

    # --- Effects ---
    effect        = { <effect> }         # scope:actor, scope:recipient, scope:target...
    reject_effect = { <effect> }         # fired only when the action has an AI acceptance step

    # --- Messages ---
    show_message          = no
    show_message_to_target = no
    should_execute_price  = no           # suppress price payment (for test/tooltip)
}
```

## Key Fields Reference
| Field | Purpose | Key constraint |
|---|---|---|
| `type` | Determines base scope context | `subject` actions scope into overlord–subject; `diplomacy` into two independent countries |
| `category` | UI grouping key | Must match a loc key defined in the interaction panel GUI |
| `diplomatic_cost` | References a cost type for spy network / trust / favor spending | Defined in `diplomatic_costs/`; distinct from `price` (gold/resource costs) |
| `use_enroute` | Sends a diplomat on a travel delay | Only meaningful for `type = diplomacy` actions |
| `cooldown.type` | Cooldown tag shared across actions using the same tag | Different interactions can share one cooldown pool |
| `diplo_chance` | Named-factor AI acceptance calculation | Factor names are engine-defined; unknown names are silently ignored |
| `accept` | Legacy script-value AI acceptance | Use `diplo_chance` instead; both can coexist |
| `ai_tick` | How often AI evaluates this action | `never` prevents all AI use; `daily` is expensive for actions with many targets |
| `ai_limit_per_check` | Max number of targets AI processes per evaluation | Default 1; set higher for multi-target spamming actions |
| `reject_effect` | Fires only when the target AI declines | Not available for actions without an acceptance step (e.g. `type = owncountry`, which has no target to accept/decline) |
| `show_in_gui_list` | Hides from auto-generated GUI lists | Set `no` for actions driven by specific UI buttons only |
| `select_trigger` | Multi-stage target selection for the action | Repeatable; each block adds a new selection stage (`scope:target`, `scope:target_1`, ...) |
| `payer` / `payee` | Who pays and who receives the price | Default: actor pays, nobody receives. Use `scope:recipient` or a scripted country to route payment |
| `ai_prerequisite` | Cheap per-country early-out before full AI evaluation | Only `scope:actor` is available — use for tag/government checks, not target iteration |
| `show_message` / `should_execute_price` | Debug/test flags to suppress notifications or price payment | Useful when wiring up tooltip-only actions or during development |
| `source_flags` | Narrows the candidate list for performance | Always set `neighbor`, `border`, or other flags where possible to avoid iterating all countries |

## Modding Notes
- **File organisation:** vanilla groups related interactions by theme (estates, IO, religion, economy). There is no technical requirement; interaction IDs must be globally unique.
- **`type = diplomacy`** actions have `scope:actor` (initiator) and `scope:recipient` (target country) available throughout. `scope:target` and `scope:target_1`+ are populated by `select_trigger` stages.
- **`type = subject`** actions set `scope:actor` to the overlord and `scope:recipient` to the subject. The subject is pre-selected by the game, not by a `select_trigger`.
- **`diplo_chance` factors are engine tags**, not script. The full list is in `readme.txt` (100+ factors). Unrecognised tags are silently ignored — always check against the list in `readme.txt`.
- **`ai_tick = never`** permanently disables AI use without removing the action from the player's UI. Use this for player-only interactions or actions that are managed by code-side AI logic.
- **`diplomatic_cost`** is a different concept from `price`. It draws from spy network points, trust, or favors rather than gold/manpower. The cost type is defined in `diplomatic_costs/` and can have its own triggers and tooltip.
- **`cooldown.type`** acts as a shared key. Two different interactions with the same `type` tag share one cooldown counter — useful for preventing rapid sequential use of related actions.
- **`ai_prerequisite`** is evaluated before any target iteration. Use it for cheap country-level checks (tag, government type, at-war status) to avoid running the full AI evaluation on ineligible actors.
- **`payer` / `payee`** default to actor and nobody respectively. To route gold from the actor to the recipient, set `payee = { scope:recipient = { ... } }`. Both fields accept arbitrary script — they can point to estates, IOs, or any country scope.
- **`interaction_source_list`** runs as a script effect to populate the candidate list. It is evaluated per actor–recipient pair, so it can be expensive in AI context; use `ai_interaction_source_list` for a cheaper AI-specific override.
- **`source_flags`** such as `neighbor`, `border`, `subjects`, `atwar` dramatically reduce the candidate pool before `visible`/`enabled` triggers run. Always provide one for country-targeting interactions.

## Example
`enforce_religion` — overlord forces religion conversion on a subject:

```pdx
enforce_religion = {
    type     = subject
    category = CATEGORY_SUBJECT_ACTIONS

    # Empty blocks are shown here for clarity; in practice they are omitted when there are no conditions.
    potential = { }
    allow     = { }

    select_trigger = {
        looking_for_a = country
        interaction_source_list = {
            scope:actor = {
                every_subject = { add_to_list = source }  # only subjects
            }
        }
        target_flag = recipient
        name = "choose_country"
        column = { data = name }
        visible = {
            not = { religion = scope:actor.religion }    # only subjects of a different religion
        }
        enabled = {
            subject_loyalty >= MIN_SUBJECT_LOYALTY_FOR_ACTION
            liberty_desire  <  MAX_SUBJECT_LIBERTY_DESIRE_FOR_ACTION
            modifier:blocked_from_conversion = no
        }
    }

    effect = {
        enforce_religion_effect = { actor = scope:actor  target = scope:recipient }
        scope:recipient = { add_liberty_desire = liberty_desire_ultimate_plus }
    }

    ai_will_do = {
        # Base score is 0 unless explicitly set. This adds 10 when the overlord is at peace.
        if = {
            limit = { scope:actor = { at_war = no } }
            add = { value = 10 }
        }
    }
}
```

The `interaction_source_list` narrows candidates to subjects only, the `select_trigger.visible` further filters to different-religion subjects, and `enabled` applies the loyalty/liberty check at selection time.
