# Generic Actions
**Stage:** complete
**Keywords:** generic_action, type, owncountry, diplomacy, subject, character, location, internationalorganization, situation, internationalorganizationparliament, religious, religiousfaction, ai_tick, ai_tick_frequency, automation_tick, player_automated_category, select_trigger, potential, allow, effect, ai_will_do, price, cooldown, show_in_gui_list

> **System type: Gameplay**

## Overview
Generic actions are the fully-scripted, no-code action system for EU5. Any button-triggered player action — hiring advisors, creating colonial charters, managing markets, declaring IO wars, parliamentary bribes — is defined here if it doesn't need a bespoke code-side handler. Generic actions share the same `select_trigger` and `ai_will_do` machinery as country interactions but cover a wider range of contexts: own-country actions, diplomatic exchanges, subject management, character actions, location/province actions, IO actions, parliament actions, and situation actions. This is the primary system for adding new player-facing buttons without code changes.

## Vanilla File Locations
- `in_game/common/generic_actions/` — one or a few `.txt` files per theme (~120+ files)
- `in_game/common/generic_actions/readme.txt` — field documentation
- Full file list in `_file_index.csv`.

## Block Structure
```pdx
<action_id> = {
    # --- Type (determines scope context) ---
    type = owncountry/diplomacy/subject/character/location/internationalorganization/situation/internationalorganizationparliament/religious/religiousfaction

    # --- Sound & messaging ---
    sound   = <sound_key>
    message = <message_loc_key>
    show_message           = no    # suppress player notifications
    show_message_to_target = no    # suppress target notifications
    show_in_gui_list       = no    # hide from auto-generated GUI lists

    # --- Cost ---
    price            = <price_key>         # e.g. price:hire_advisor
    price_modifier   = { <script value> }  # multiplier; scope:actor, scope:recipient, scope:target...
    payer            = { <script> }        # who pays (default: actor)
    payee            = { <script> }        # who gets paid (default: nobody)
    should_execute_price = no              # skip price payment (tooltips / testing)

    # --- Target selection (repeatable) ---
    select_trigger = {
        looking_for_a   = character/location/province/province_definition/area/region/country/value/boolean
        target_flag     = <scope_name>     # default: target, target_1, target_2...
        source          = actor/recipient/target/target_1/.../world
        source_ai_override = <source>
        source_flags    = <flags>          # neighbor, possible_colonial_charters, adjacent_locations, etc.
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
        top_widget     = <widget_key>
        bottom_widget  = <widget_key>
        column = { data = <col_id> width = <int> icon = <path> }
        default_sort   = <sort_key>
        tooltip_msg_key = <loc_key>
        none_available_msg_key = <loc_key>
        show_why_not_visible = yes/no
        show_why_not_enabled = yes/no
        show_if    = { <trigger> }
        visible    = { <trigger> }
        enabled    = { <trigger> }
        selected   = { <trigger> }
        min        = { <script value> }
        max        = { <script value> }
        step       = { <script value> }
        default    = { <script value> }
        ai_override_value = { <script value> }
        map_mode   = <map_mode>
        map_color  = { <script color> }
        only_color_selectable = yes/no
        secondary_map_color = { <script color> }
    }

    # --- Eligibility ---
    potential    = { <trigger> }    # scope:actor = acting country
    allow        = { <trigger> }    # scope:actor, scope:recipient, scope:target...

    # --- AI ---
    ai_tick           = never/daily/monthly
    ai_tick_frequency = { <script value> }   # every N ticks per country
    ai_prerequisite   = { <trigger> }        # cheap early-out (scope:actor only)
    ai_will_do        = { <effect script> }  # scores the action for AI
    maximum_targets_in_one_tick = <int>      # -1 = unlimited; default 1
    disallowed_duplicates_of_targets_for_ai = { <target_flag list> }

    # --- Player automation ---
    automation_tick           = never/daily/monthly
    automation_tick_frequency = { <script value> }
    player_automated_category = finances/research/trade/productionmethods/laws/cabinet/parliament/estates/exploration/colonies/cultureacceptance/religiousdoctrines/buildings/rgo/armybuilder/navybuilder/governmentreforms

    # --- Cooldown ---
    cooldown = {
        type   = <tag>
        days/weeks/months/years = <int>
    }

    # --- Effects ---
    effect = { <effect> }    # scope:actor, scope:recipient, scope:target...; + scope:price, scope:price_modifier, scope:payer, scope:payee

    # --- UI ---
    force_click_and_confirm_or_hold = yes    # always show confirmation dialog
}
```

## Action Types
| Type | Scope context | Typical use |
|---|---|---|
| `owncountry` | `scope:actor` = acting country; no recipient | Self-management: hiring, building, exploring |
| `diplomacy` | `scope:actor` + `scope:recipient` = two countries | Offers, demands, requests between countries |
| `subject` | `scope:actor` = overlord; `scope:recipient` = subject | Subject management |
| `character` | Scopes into a character | Character interactions scripted as generic actions |
| `location` | Scopes into a location / province | Location-targeted actions |
| `internationalorganization` | Scopes into an IO | Joining, leaving, IO-specific actions |
| `situation` | Scopes into a situation | Actions tied to active situations |
| `internationalorganizationparliament` | Scopes into an IO parliament | Parliamentary vote actions |
| `religious` | Scopes into a religion | Actions tied to a specific religion |
| `religiousfaction` | Scopes into a religious faction | Actions tied to religious factions within a religion |

## Key Fields Reference
| Field | Purpose | Key constraint |
|---|---|---|
| `type` | Sets the scope context for `potential`, `allow`, and `effect` | Determines what scopes (`scope:actor`, `scope:recipient`, etc.) are available |
| `ai_tick` | AI evaluation cadence | `never` disables AI use entirely; `daily` is expensive for multi-country actions |
| `ai_tick_frequency` | How often (in ticks) the AI re-evaluates per country | Script value; root = country |
| `ai_prerequisite` | Cheap per-country early-out before full AI evaluation | Only `scope:actor` is available here; no targets |
| `maximum_targets_in_one_tick` | How many targets the AI may act on per check | Default 1; set -1 for bulk actions |
| `player_automated_category` | Enables player automation for this action | Must match one of the fixed automation category strings |
| `automation_tick` / `_frequency` | Separate tick rate for player automation (distinct from AI tick) | Can differ from `ai_tick` |
| `force_click_and_confirm_or_hold` | Forces a confirmation dialog before executing | Use for dangerous IO-level actions (declaring wars) |
| `show_in_gui_list` | Controls automatic GUI list inclusion | Set `no` for actions that appear via custom widgets only |
| `source_flags` | Narrows candidate pool before `visible`/`enabled` run | Always provide; `possible_colonial_charters`, `neighbor`, `border`, etc. save significant CPU |
| `cache_targets` | Caches the target list when it never changes per actor | Set `yes` when the list is fully static (e.g. fixed province definitions) |
| `ai_override_value` | Single script value for AI target evaluation (skips full evaluation) | Use for performance when AI only needs a simple ranking |

## Modding Notes
- **Generic actions vs country interactions:** functionally identical — same block fields, same `select_trigger` machinery. The distinction is only conventional: `country_interactions/` holds specifically bilateral-diplomacy and subject-management actions; `generic_actions/` holds everything else. Both folders are loaded identically by the engine.
- **`type` determines available scopes**, not the UI category. A `type = owncountry` action can still target a location via `select_trigger`; the type just sets what is in scope before selection.
- **`ai_tick = never`** is the correct way to disable AI use of an action that is already handled by code-side AI or is player-only. Do not rely on `ai_will_do` returning 0, as the action may still be evaluated and just score nothing.
- **`player_automated_category`** ties the action into EU5's player automation system. If the player enables automation for that category, the action runs on `automation_tick` following `ai_will_do` scoring. Only the fixed category strings are valid; unknown strings are silently ignored.
- **`maximum_targets_in_one_tick`** with `disallowed_duplicates_of_targets_for_ai` lets the AI act on multiple non-overlapping targets in a single evaluation pass — useful for bulk-construction or mass-hiring actions.
- **`force_click_and_confirm_or_hold`** forces a confirmation dialog before the action fires. Used in vanilla for IO actions (e.g. `tatar_yoke.txt`) where an immediate click would bypass the normal confirmation flow. Use for any action whose consequence is difficult to reverse.
- **Cooldown sharing:** as with country interactions, `cooldown.type` is a shared tag. Multiple actions with the same type share one cooldown counter. This can be intentional (e.g. all "ask for money" variants share a cooldown) or an accidental collision if the same tag is reused in a mod.
- **`select_trigger` is the full shared selection schema** (identical to country interactions and peace treaties). Refer to `readme.txt` for the complete field list. Performance-critical fields: `source_flags`, `cache_targets`, `ai_override_value`, `ai_interaction_source_list`.

## Example
`hire_advisor` — own-country action that spawns a new cabinet character, AI-driven by how short-staffed the country is:

```pdx
hire_advisor = {
    type  = owncountry
    price = price:hire_advisor

    ai_tick           = daily
    ai_tick_frequency = 180           # AI checks every 180 days (~6 months)
    automation_tick   = monthly
    automation_tick_frequency = 1
    player_automated_category = governmentreforms

    potential = { }    # visible to all countries
    allow     = { }    # no extra requirement beyond price

    ai_will_do = {
        scope:actor = {
            value = {
                add = modifier:government_size   # target headcount (how many you should have)
                subtract = num_cabinet_capable_characters
                multiply = 14                    # scaling weight
            }
        }
    }

    effect = {
        scope:actor = {
            # ... creates a character assigned to the appropriate estate type
            create_character = { ... }
        }
    }
}
```

The action has no `select_trigger` — the effect script chooses the estate internally. The AI checks every 6 months and hires when the cabinet is understaffed. The same action also participates in player automation under the `governmentreforms` category.
