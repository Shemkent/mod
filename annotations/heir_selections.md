# Heir Selections
**Stage:** complete
**Keywords:** heir_selection, succession_law, traverse_family_tree, include_ruler_siblings, through_female, allow_female, allow_male, allow_children, ignore_ruler, use_election, depth_first, sibling_score, potential, allowed_estates, heir_is_allowed, succession_effect, term_duration, is_a_praefecta

> **System type: Gameplay**

## Overview
Heir selection definitions scripted succession law rules — how a country determines which character becomes the next ruler. Each heir selection block defines candidate filtering (which family members are eligible), candidate scoring (`sibling_score`), an optional `potential` trigger (which laws can use this selection method), and an `heir_is_allowed` trigger (per-candidate validation). Succession laws in the `laws/` folder reference heir selection IDs via `succession_law = heir_selection:<id>`. Vanilla ships ~20+ heir selection types across five files organized by government type.

## Vanilla File Locations
- `in_game/common/heir_selections/` — 5 `.txt` files by government type (+ `00_heir_selections.info` template)
  - `monarchy.txt` — primogeniture variants, elective, etc.
  - `republic.txt` — elected presidents, term rulers
  - `theocracy.txt` — conclave, papal election
  - `tribal.txt` — tribal succession
  - `specialized.txt` — unique national succession (Japan, etc.)
- Full file list in `_file_index.csv`.

## Block Structure
```pdx
<heir_selection_id> = {
    # --- Family tree traversal ---
    traverse_family_tree    = yes/no   # search through the family tree for candidates
    depth_first             = yes/no   # prefer closer relatives (depth-first vs breadth-first)
    include_ruler_siblings  = yes/no   # include the ruler's siblings as candidates
    through_female          = yes/no   # allow succession through the female line
    allow_female            = yes/no   # female characters can be candidates
    allow_male              = yes/no   # male characters can be candidates
    allow_children          = yes/no   # children (under adult age) can be candidates
    ignore_ruler            = yes/no   # exclude the current ruler from calculation

    # --- External candidates ---
    include_other_countries = { <trigger> }   # conditions for importing candidates from other countries

    # --- Election ---
    use_election    = yes/no           # use an election mechanism instead of family tree
    show_candidates = yes/no           # show candidates in the UI during election
    cached          = yes/no           # cache candidate list for performance
    term_duration   = <int>            # months the elected ruler holds power

    # --- Availability ---
    potential = { <trigger> }          # when this heir selection is available as a succession law
    allowed   = { <trigger> }          # additional criteria for the law to be selectable
    locked    = { <trigger> }          # prevent changing away from this law if true

    # --- Candidate filtering ---
    allowed_estates = { <estate_id> ... }   # which estates can provide candidates
    heir_is_allowed = {
        # Trigger; root = candidate character, scope:target = country
    }

    # --- Candidate scoring ---
    sibling_score = {
        # Script value; root = candidate character
        # Higher score = preferred candidate
        # Typically: age bonus, gender bonus, legitimacy penalty for unfit rulers
        add = {
            desc = "LOC_KEY"
            if = { limit = { ... } value = <float> }
        }
    }

    # --- Post-succession ---
    succession_effect = { <effect> }   # fired when succession occurs via this method

    # --- Performance ---
    custom_tags = { <strings> }        # internal tags for script queries
}
```

## Key Fields Reference
| Field | Purpose | Key constraint |
|---|---|---|
| `traverse_family_tree` | Search the ruler's family for candidates | Set no for election-based systems |
| `through_female` | Allow female-line succession | Distinct from `allow_female` — controls inheritance path, not candidate gender |
| `allow_female` / `allow_male` | Candidate gender filter | Both can be yes (no gender restriction) |
| `depth_first` | Candidate search order | Yes = direct descendants fully checked before collateral relatives (siblings, cousins); No = breadth-first, all candidates at each generational distance before going deeper |
| `include_ruler_siblings` | Add ruler's siblings to candidate pool | Required for most agnatic/cognatic succession types |
| `sibling_score` | Rank candidate characters | Script value on the candidate; higher = more preferred; typically scores age, gender, fitness |
| `heir_is_allowed` | Per-candidate trigger | Root = candidate character; false = excluded from pool entirely |
| `allowed_estates` | Estate membership filter | Only characters belonging to listed estates are considered |
| `potential` | Law availability trigger | Controls which succession laws are unlockable; evaluated at law selection time |
| `locked` | Prevent succession law changes | Used for situations where changing succession would be illegal or blocked |
| `use_election` | Election-based succession | Candidate ranking still uses `sibling_score` for AI voter preference |
| `term_duration` | Elected ruler term length | Only meaningful when `use_election = yes`; in months |

## Modding Notes
- **Heir selections are referenced by succession laws.** The actual gameplay activation comes from a law in `laws/` setting `succession_law = heir_selection:<id>`. The heir selection file defines the mechanics; the law file defines when it is available and what it costs to enact.
- **`sibling_score` controls candidate preference**, not eligibility. A character scoring −1000 (unsuited to rule) is still in the pool but will almost never be selected. Use `heir_is_allowed` to hard-exclude candidates.
- **`through_female` vs `allow_female`:** `through_female = yes` means the succession can trace through a female character to her children; `allow_female = yes` means female characters can themselves inherit. These are independent flags.
- **`include_ruler_siblings`** must be yes for any system that should consider the ruler's brothers and sisters. Without it, only direct descendants are candidates.
- **`allowed_estates`** is an estate whitelist. If omitted, all characters are eligible regardless of estate. This is used to restrict succession to noble families (`nobles_estate`) in hereditary monarchies.
- **Election systems** (`use_election = yes`) still use `sibling_score` for AI voting weights. A high-scoring candidate will receive more AI votes.
- **`succession_effect`** fires on every succession using this method. Use for one-time consequences like legitimacy changes or event triggers.
- **`sibling_score` commonly uses an `if`/`else` pattern.** The `if = { limit = { exists = root } ... } else = { ... }` structure provides real scoring when `root` is a character context and fallback display values when `root` does not exist (used by the UI to show expected scores). Always include the `else` branch when using `exists = root` guards.
- **`is_a_praefecta`** is a character trigger for legally-male characters (relevant to some cultures and government types). Include a bonus for `is_a_praefecta = yes` alongside the male-candidate bonus in any gender-biased succession system.
- **Cross-system:** heir selections reference the estate system (`allowed_estates`), character stats and modifiers (`modifier:blocked_from_being_ruler`), and government types. The `potential` trigger often tests `government_type` or active laws to restrict selection methods to appropriate regimes.

## Example
`cognatic_primogeniture` — all children and siblings eligible regardless of gender, older preferred:

```pdx
cognatic_primogeniture = {
    include_ruler_siblings = yes
    through_female         = yes
    allow_female           = yes
    allow_children         = yes
    ignore_ruler           = yes
    potential = {
        NOR = {
            succession_law = heir_selection:fratricide_succesion
            succession_law = heir_selection:unigeniture
        }
    }
    allowed_estates = { nobles_estate }
    heir_is_allowed = { }             # no additional restrictions
    traverse_family_tree = yes
    depth_first          = yes
    sibling_score = {
        if = {
            limit = { exists = root }
            add = {
                desc = "HEIR_SELECTION_MALE_DESC"
                # Fires when root doesn't exist OR candidate is male
                if = { limit = { OR = { not = { exists = root } is_female = no } } value = 100 }
            }
            add = {
                desc = "HEIR_SELECTION_LEGALLY_MALE_DESC"
                # is_a_praefecta = legally-male characters also score the male bonus
                if = { limit = { is_a_praefecta = yes } value = 100 }
            }
            add = {
                desc = "HEIR_SELECTION_OLDER_HEIR_DESC"
                value = character_age     # older candidates preferred
            }
            add = {
                desc = "HEIR_SELECTION_IS_UNSUITED_TO_RULE_DESC"
                if = { limit = { modifier:blocked_from_being_ruler = yes } value = -1000 }
            }
        }
        else = {
            # Fallback display values for UI rendering when root does not exist
            add = { desc = "HEIR_SELECTION_MALE_DESC"             value = 100 }
            add = { desc = "HEIR_SELECTION_LEGALLY_MALE_DESC"     value = 100 }
            add = { desc = "HEIR_SELECTION_OLDER_HEIR_DESC"       value = 1 }
            add = { desc = "HEIR_SELECTION_IS_UNSUITED_TO_RULE_DESC" value = -1000 }
        }
    }
}
```

Male characters (and legally-male characters via `is_a_praefecta`) score +100, and older candidates score higher via `character_age`. Characters with `blocked_from_being_ruler` score −1000 and will not be selected unless no alternatives exist. The `else` branch provides display-only fallback values for UI rendering when no `root` context exists. `allowed_estates = { nobles_estate }` restricts the pool to noble-estate characters only.
