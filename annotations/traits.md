# Traits
**Stage:** complete
**Keywords:** trait, category, flavor, allow, modifier, chance, personality, government_approach, interests, education, ruler, general, admiral, artist, explorer, has_trait, adm, dip, mil, trait_flavor, monthly_towards

> **System type: Gameplay**

## Overview
Traits are character-level modifiers that shape a ruler's, general's, admiral's, artist's, or explorer's effectiveness and gameplay options. Each trait has a `category` that restricts it to a specific character role, a `flavor` that groups it visually in the UI, an `allow` trigger controlling eligibility, a `modifier` block applied when the character is in the matching role, and an optional `chance` block (MTTH-style) controlling how likely the trait is to appear. Trait modifiers interact with estates, societal values, stability, institutions, culture, and the diplomatic/military systems. Vanilla ships 100+ traits across 7 files.

## Vanilla File Locations
- `in_game/common/traits/` — 7 role files + `_traits.info` template
  - `00_ruler.txt` — personality and governance traits for rulers
  - `01_general.txt` — general-specific combat traits
  - `02_admiral.txt` — admiral-specific naval traits
  - `03_artist.txt` — traits for court artists
  - `04_explorer.txt` — traits for explorers
  - `05_child.txt` — traits present during childhood
  - `06_religious_figure.txt` — traits for religious characters
- `in_game/common/trait_flavor/00_default.txt` — defines flavor categories and their UI colors
- Full file list in `_file_index.csv`.

## Block Structure
```pdx
<trait_id> = {
    # --- Role ---
    category = ruler/general/admiral/artist/explorer/child/religious_figure
    # Default: ruler (if omitted). The info template lists only five roles;
    # child and religious_figure are additional vanilla categories not in the template.
    flavor   = personality/government_approach/interests/education   # UI color grouping

    # --- Eligibility ---
    allow = {
        # Trigger on the character; root = character being assessed
        # Common: adm > X, dip > X, mil > X, has_trait = X, NOT = { has_trait = Y }
        # Owner context: owner ?= { government_type = ... }
    }

    # --- Effect ---
    modifier = {
        # Applied to the country while this character is in-role (ruler traits are country-scope)
        global_estate_target_satisfaction = <scripted_constant>
        monthly_legitimacy = <float>
        monthly_republican_tradition = <float>
        peace_offer_fairness = <float>
        stability_importance_modifier = <float>
        cultural_tradition_modifier = <float>
        embrace_institution_cost_modifier = <float>
        bias_for_militarist_policies = <int>
        monthly_towards_<societal_value_side> = societal_value_minor_monthly_move
        can_execute_characters = yes   # flag modifiers also valid
    }

    # --- Appearance probability ---
    chance = {
        # MTTH (Mean Time To Happen) style script value.
        # The value is the expected number of months before this trait appears randomly.
        # Lower base months = more frequent random assignment.
        # root = character
        value = <base_months>
        # modify = { add = X limit = { ... } }
    }
}
```

## Trait Flavor Categories
Defined in `trait_flavor/00_default.txt` as named color groups (rgb). All vanilla flavors:

| Flavor | Typical content |
|---|---|
| `personality` | Core character traits (just, cruel, tolerant, zealot, etc.) |
| `government_approach` | Governance and diplomatic disposition (tolerant, warmonger, etc.) |
| `interests` | Passions (scholar, hunter, etc.) |
| `education` | Education-outcome traits (military education, administrative education, etc.) |

## Key Fields Reference
| Field | Purpose | Key constraint |
|---|---|---|
| `category` | Which character role this trait applies to | Defaults to `ruler` if omitted; modifiers only activate when the character is serving in that role |
| `flavor` | Visual grouping | Must match an ID in `trait_flavor/`; unknown flavors render without color |
| `allow` | Character eligibility trigger | Root = character; tested at assignment time, not re-enforced after |
| `modifier` | Modifiers applied while character is in role | Uses country modifier namespace; `monthly_towards_*` pushes societal values |
| `chance` | MTTH probability for random assignment | Omit if the trait is assigned only via events or decisions |

## Vanilla Trait Categories
| Category | Key modifier themes |
|---|---|
| `ruler` | Estate satisfaction, legitimacy, stability costs, societal value pushes, war tolerance, diplomacy |
| `general` | Combat modifiers, discipline, siege ability, morale, cavalry/infantry bonuses |
| `admiral` | Naval combat, ship maintenance, trade protection |
| `artist` | Prestige, cultural tradition, art commission output |
| `explorer` | Exploration speed, colonial range, encounter modifiers |
| `child` | Education stat bonuses; transitions to adult traits on coming-of-age |
| `religious_figure` | Conversion speed, religious unity, clergy estate satisfaction |

## Modding Notes
- **Trait modifiers apply only when the character is in the matching role.** A `general` trait on a character who is currently ruling does not grant its modifiers. When assigned to lead an army, the modifiers activate.
- **`allow` is tested at assignment time, not re-enforced after.** If a character later loses the prerequisite stat or trait, the existing trait remains.
- **Trait exclusivity** is enforced manually via `NOT = { has_trait = X }` in `allow`. The engine has no built-in mutual exclusion system.
- **`monthly_towards_<side>`** in `modifier` pushes the country's societal value slider monthly while the character is in role. Choose `societal_value_minor_monthly_move` for a subtle nudge or `societal_value_monthly_move` for a stronger push — both are named constants defined in static modifiers.
- **Adding a new trait** is purely additive — define the block in any `.txt` file in `traits/`, add localization, and the trait is available for event assignment. All `.txt` files in the folder are loaded.
- **`trait_flavor/`** defines only UI display (name + color). Adding a new flavor requires an entry there plus localization. Unknown flavor IDs render without color but do not error.
- **`has_trait`** is the trigger to check for a trait in other scripts. Syntax: `has_trait = <trait_id>` (not `trait:<id>`).
- **Cross-system:** trait modifiers interact with estates, societal values, institutions, stability, and AI biases. Some traits enable otherwise-blocked actions (`can_execute_characters = yes`). Traits also interact with the character stats (`adm`, `dip`, `mil`) used in `allow` blocks.

## Example
`just` — a ruler personality trait requiring administrative aptitude and excluding cruelty:

```pdx
just = {
    allow = {
        adm > 33                        # requires administrative stat above 33
        NOT = { has_trait = cruel }     # mutually exclusive with cruel
    }
    category = ruler
    flavor   = personality

    modifier = {
        global_estate_target_satisfaction = tiny_permanent_target_satisfaction
        monthly_towards_free_subjects = societal_value_minor_monthly_move
        peace_offer_fairness = 0.2
        stability_importance_modifier = 0.1
    }
}
```

Available to any ruler with `adm > 33` who does not have `cruel`. While the character is ruling, the country gains a small estate satisfaction bonus and a monthly push toward free subjects on the serfdom/free_subjects axis. There is no `chance` block — assignment is event-driven only.
