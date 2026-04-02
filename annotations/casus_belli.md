# Casus Belli
**Stage:** complete
**Keywords:** casus_belli, cb_, war_goal_type, allow_creation, allow_declaration, visible, province, ai_will_do

> **System type: Gameplay**

## Overview
Casus belli (CB) blocks define the legal justifications a country may use to declare war. Each CB controls who can see it, who can create it, which provinces are valid targets, how fast it accumulates, and which war goal type governs the resulting peace deal. Modders add CBs to introduce new war types or restrict existing ones by government type, advance, religion, or other triggers. CBs are the entry point into EU5's war system: they bind declaration logic to a war goal, and the war goal then governs what can be demanded at the peace table.

## Vanilla File Locations
- `in_game/common/casus_belli/` — one `.txt` file per CB or logical group (~60 files)
  - `00_hardcoded.txt` — engine-generated CBs (no_cb, event wars, insults, rebel support, etc.)
  - `01_event_triggered.txt` — placeholder; event-triggered CBs are spawned via script
  - Named files for each substantive CB (`conquest.txt`, `imperialism.txt`, `humiliate_rival.txt`, etc.)
- Full file list in `_file_index.csv`.

## Block Structure
```pdx
<cb_id> = {
    # Visibility & access (all triggers; root = country, scope:target = target)
    visible          = { <trigger> }       # show CB in UI at all
    allow_creation   = { <trigger> }       # can accumulate / fabricate this CB
    allow_declaration = { <trigger> }      # can use to declare war right now

    # Province-scoped targeting (root = province, scope:actor / scope:recipient)
    province = { <trigger> }               # is this province a valid CB target?

    # Mechanics
    war_goal_type    = <wargoal_id>        # which war goal applies when this CB is used
    speed            = <float>             # percentage of CB accumulated per month; 100/speed = months to complete
    days/weeks/months/years = <int>        # sets the maximum CB lifespan before it expires; overrides CASUS_BELLI_MONTHS define

    # War enthusiasm (script values; root = country, scope:war = war)
    additional_war_enthusiasm          = { <script value> }
    additional_war_enthusiasm_attacker = { <script value> }
    additional_war_enthusiasm_defender = { <script value> }

    # AI controls
    ai_will_do               = { <script value> }   # overrides default AI eagerness
    ai_subjugation_desire    = { <script value> }   # root = country; scope:recipient, scope:subject_type, scope:war
    ai_cede_location_desire  = { <script value> }   # root = country; scope:location, scope:war
    antagonism_reduction_per_warworth_defender = { <script value> }   # reduces antagonism on the defender side proportional to war worth; root = country, scope:recipient, scope:war

    # Flags
    no_cb                   = yes          # marks the "no justification" CB
    trade                   = yes          # trade-related CB (for UI categorisation)
    allow_separate_peace    = no           # defaults yes
    cut_down_in_size_cb     = yes          # AI hint: prefer release treaties
    can_expire              = yes/no
    allow_wars_on_own_subjects = yes       # needed for overlord-vs-subject CBs
    allow_ports_for_reach_ai   = yes       # AI ignores control requirements (crusades)
    max_warscore_from_battles  = <int>     # override NDefines::NDiplomacy::WARSCORE_MAX_FROM_BATTLES

    # Tagging (UI classification)
    custom_tags    = { tag_a tag_b }   # arbitrary string identifiers; referenced in triggers via cb_has_custom_tag
    show_tags_in_ui = yes              # exposes these tags in CB tooltips
}
```

## Key Fields Reference
| Field | Purpose | Key constraint |
|---|---|---|
| `visible` | Gate for UI display | Runs every frame on diplomacy screen — keep cheap |
| `allow_creation` | Gate for fabricating / accumulating the CB | Separate from declaration; a CB can be visible but not creatable |
| `allow_declaration` | Gate for the actual war declaration | Checked at declaration time only |
| `province` | Province-level targeting validity | Root = province, not country; use `scope:actor`/`scope:recipient` for countries |
| `war_goal_type` | Links to a war goal definition | Must match an ID in `wargoals/00_default.txt` |
| `speed` | Accumulation rate: `100/speed` = months to complete | Omitting uses the global define `CASUS_BELLI_MONTHS`; distinct from the duration set by `days/weeks/months/years` |
| `days/weeks/months/years` | Maximum CB lifespan before it expires | Sets expiry time, not accumulation speed; omitting uses `CASUS_BELLI_MONTHS` define |
| `ai_will_do` | Script value; overrides default CB selection weight | Return ≤ 0 to disable AI use; magnitude beyond zero also matters |
| `no_cb` | Marks the engine CB for unjustified war | Only one CB should have this |
| `allow_wars_on_own_subjects` | Needed for CBs targeting own subjects | Defaults no |

## Modding Notes
- **One file per CB** is the vanilla convention for named CBs; `00_hardcoded.txt` groups engine-internal ones.
- **Load order matters for overrides:** to replace a vanilla CB, match its filename prefix or load after it; CB IDs must be globally unique.
- **`visible` is performance-sensitive** — it runs on the diplomacy screen for every potential target. Avoid heavy iterations inside it.
- **`province` is root-scoped to the province**, not the country. This is a common scope mistake — use `scope:actor`/`scope:recipient` to reference countries.
- **`war_goal_type` is the CB's only hard link to the war system.** Changing it changes what can be demanded in the resulting peace deal without touching the CB declaration logic.
- **`allow_wars_on_own_subjects = yes`** must be set for any CB intended to be declared against a subject (e.g. a rebellion or subjugation CB).
- **Event-triggered CBs (`cb_war_from_event`, etc.)** have `visible = { always = no }` and are granted by script — they never appear in the normal CB list.
- **AI tuning:** `ai_will_do` suppresses or promotes AI use regardless of the CB's underlying desirability. Returning 0 or a negative value is sufficient to prevent AI use; returning a large positive value makes the AI strongly prefer this CB.
- **`custom_tags`** are arbitrary strings defined inline. They can be tested in script via the `cb_has_custom_tag` trigger. The `show_tags_in_ui = yes` flag exposes them in the CB tooltip for modder/player inspection.
- **`antagonism_reduction_per_warworth_defender`** reduces the antagonism generated on the defender side in proportion to the war's "war worth" value. It is used for CBs where a strong defender should receive less reputational penalty for fighting back.
- **Missing `visible` block:** if `visible` is omitted, the CB is always visible (not hidden). This is different from `allow_creation` — a CB can be visible but not yet accumulable.

## Example
`conquest.txt` — the core territorial CB used to claim provinces where the attacker has cores:

```pdx
cb_conquer_province = {
    allow_creation = {
        scope:target = {
            any_owned_location = {
                is_core_of = root    # target must own at least one of root's cores
            }
        }
    }

    # Province-level filter: only core provinces are valid targets
    # Province-level filter: only provinces where at least one location is a core are valid targets.
    # In EU5, a "province" is a geographic grouping; a "location" is the individual ownable tile within it.
    # `any_location_in_province` iterates the locations inside the scoped province.
    province = {
        any_location_in_province = {
            is_core_of = scope:actor
        }
    }

    war_goal_type = conquer_province    # → wargoals/00_default.txt
}
```

No `visible` block means the CB is always visible (not hidden). No `speed` means accumulation uses the `CASUS_BELLI_MONTHS` global define. The peace deal options are entirely controlled by the `conquer_province` war goal.
