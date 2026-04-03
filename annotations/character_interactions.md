# Character Interactions
**Stage:** complete
**Keywords:** character_interaction, on_other_nation, on_own_nation, is_consort_action, potential, allow, effect, price, price_modifier, payer, payee, select_trigger, ai_will_do, ai_tick, ai_tick_frequency, cooldown, message

> **System type: Gameplay**

## Overview
Character interactions are the scripted button-driven actions players use to manage individual characters — executing, pardoning, appointing, marrying, promoting, banishing, or commissioning art. They share the same `select_trigger`, `price`, `allow`, `effect`, `ai_will_do`, and `cooldown` machinery as `country_interactions` and `generic_actions` (though character interactions do not support `automation_tick` or `player_automated_category`), and operate in character scope: `scope:actor` is the acting country; characters land in `scope:recipient`, `scope:target`, etc., as set by `select_trigger`. Three flags distinguish character interactions from other action systems: `on_other_nation`, `on_own_nation`, and `is_consort_action`.

## Vanilla File Locations
- `in_game/common/character_interactions/` — one `.txt` per interaction (~29 files + readme)
- `in_game/common/character_interactions/readme.txt` — field documentation
- Full file list in `_file_index.csv`.

## Block Structure
```pdx
<interaction_id> = {
    # --- Scope flags ---
    on_other_nation   = yes/no    # allow use on characters from other nations (default no)
    on_own_nation     = yes/no    # allow use on own-nation characters (default no)
    is_consort_action = yes/no    # restrict to consort-targeted actions

    # --- Sound & messaging ---
    sound   = <sound_key>
    message = yes/no              # show interaction message (default yes)
    show_message           = no   # suppress message to actor
    show_message_to_target = no   # suppress message to target character's controller
    show_in_gui_list       = no   # hide from auto-generated action lists

    # --- Cost ---
    price            = <price_key>         # e.g. price:abdicate_price
    price_modifier   = { <script value> }  # multiplier; scope:actor, scope:recipient, scope:target...
    payer            = { <script> }        # who pays (default: actor)
    payee            = { <script> }        # who receives payment (default: nobody)
    should_execute_price = no             # skip price payment (tooltips / testing)

    # --- Target selection (repeatable) ---
    select_trigger = {
        looking_for_a   = character/location/province/area/region/country/value/boolean
        target_flag     = <scope_name>     # default: target, target_1, target_2...
        source          = actor/recipient/target/target_1/.../world
        source_ai_override = actor/recipient/target/target_1/.../target_4   # AI-only source override
        source_flags    = <flags>          # include_dead, include_any_present, etc.
        source_flags_ai_override = <flags>   # AI-only source flag override
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
        allow_null_trigger = { <trigger> }
        allow_self     = yes/no
        move_to_next_section_on_click = yes/no
        top_widget     = <widget_key>
        bottom_widget  = <widget_key>
        column = { data = <col_id> width = <int> icon = <path> }
        default_sort   = <sort_key>
        none_available_msg_key = <loc_key>
        show_why_not_visible = yes/no
        show_why_not_enabled = yes/no
        show_if    = { <trigger> }
        visible    = { <trigger> }
        enabled    = { <trigger> }
        selected   = { <trigger> }
        min / max / step / default = { <script value> }
        map_mode   = <map_mode>
        map_color  = { <script color> }
        only_color_selectable = yes/no
        secondary_map_color = { <script color> }
    }

    # --- Eligibility ---
    potential = { <trigger> }    # scope:actor = acting country
    allow     = { <trigger> }    # scope:actor, scope:recipient, scope:target...

    # --- AI ---
    ai_tick           = never/daily/monthly
    ai_tick_frequency = <int or script value>   # every N ticks per country; can be bare integer (e.g. 120) or script block
    ai_will_do        = { <effect script> }  # scores the action for AI

    # --- Cooldown ---
    cooldown = {
        type   = <tag>
        days/weeks/months/years = <int>
    }

    # --- Effects ---
    effect = { <effect> }    # scope:actor = country, scope:recipient/target = character(s)
}
```

## Key Fields Reference
| Field | Purpose | Key constraint |
|---|---|---|
| `on_other_nation` | Allow targeting foreign characters | Default no; set yes for diplomacy (marriages, ransom, etc.) |
| `on_own_nation` | Allow targeting own-country characters | Default no; set yes for domestic management |
| `is_consort_action` | Restricts to consort-targeted UI context | Separate button group in the character UI |
| `potential` | Visibility trigger | Root = `scope:actor` (country); character is not yet in scope |
| `allow` | Enablement trigger | Full character scopes available; `scope:recipient` / `scope:target` = characters |
| `price` | Named price definition | Via `price:<price_id>` from `prices/` folder |
| `ai_will_do` | AI scoring script (uses effect-script syntax; produces a numeric score) | Higher score = AI more likely to use; same mechanics as country_interactions |
| `should_execute_price` | Skip price payment | Set `no` for tooltip/testing actions that show costs without charging them |
| `show_in_gui_list` | Auto-include in GUI action lists | Set `no` for interactions that appear only via custom widgets |
| `cooldown.type` | Shared cooldown tag | Multiple interactions with the same type share one cooldown counter |

## Modding Notes
- **Structurally identical to `generic_actions` and `country_interactions`** — the same `select_trigger`, `price`, `ai_will_do`, and `cooldown` schema applies. The only character-interaction-specific fields are `on_other_nation`, `on_own_nation`, and `is_consort_action`. Refer to `generic_actions.md` for the full `select_trigger` field list.
- **`scope:actor` is always the acting country**, not the character performing the action. To scope into the character, use `scope:recipient`, `scope:target`, etc., as set by `select_trigger`.
- **`on_other_nation` and `on_own_nation` can both be `yes`**, making the interaction available for characters from any country.
- **No `automation_tick` / `player_automated_category`** — character interactions cannot be automated by players the way generic actions can.
- **`cooldown.type` is shared** across all three action systems. Accidental tag collisions between character and country interactions will merge their cooldown counters.
- **Cross-system:** character interactions reference `prices/`, character scope triggers, estate types, and cabinet positions. An interaction that appoints a character to a role requires that role to be defined in the government type or cabinet system.

## Example
`abdicate` — ruler voluntarily steps down in favour of the heir:

```pdx
abdicate = {
    message           = yes
    is_consort_action = no
    on_own_nation     = yes

    price = price:abdicate_price

    price_modifier = {           # price reduced for older rulers (less resistance)
        add = 1
        if = {
            limit = { scope:recipient ?= { age_in_years > 40 } }
            subtract = {
                desc = "CHAR_AGE_LABEL"
                scope:recipient = {
                    value    = age_in_years
                    subtract = 40
                    multiply = 0.02
                }
            }
        }
    }

    potential = { }              # always visible

    allow = {
        scope:actor = { has_heir = yes }
    }

    ai_tick           = daily
    ai_tick_frequency = 120      # bare integer: check every 120 days

    select_trigger = {
        looking_for_a = character
        source        = actor
        target_flag   = recipient    # result goes into scope:recipient (not scope:target)
        name          = "choose_character"
        column        = { data = name }
        visible  = { scope:actor = { ruler ?= root } }
        enabled  = {
            or = {
                and = { is_female = yes; scope:actor = { heir ?= { is_adult = yes; is_female = no } } }
                and = { total_abilities < 100; character_age > 60 }
            }
        }
    }

    effect = {
        scope:actor = { set_new_ruler = heir }
    }

    ai_will_do = {
        add = {
            desc  = "RULER_ABILITY"
            scope:recipient = { subtract = total_abilities; multiply = 0.02 }
        }
        add = {
            desc  = "HEIR_ABILITY"
            scope:recipient = { add = total_abilities; multiply = 0.02 }
        }
    }
}
```

Key patterns to note: `target_flag = recipient` stores the selected character in `scope:recipient` rather than the default `scope:target`. `ai_tick_frequency = 120` is a bare integer (not a block). The `price_modifier` block reduces cost for older rulers. The AI abdicates when the heir's abilities outweigh the current ruler's — a positive score favours abdication.
