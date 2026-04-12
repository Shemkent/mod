# Religious Aspects
**Stage:** complete
**Keywords:** religious_aspect, religion, enabled, modifier, opinions, has_religious_aspect, monthly_religious_influence, mutual_exclusivity

> **System type: Gameplay**

## Overview
Religious aspects are optional doctrines or practices that countries can adopt for their religion. Each aspect applies a modifier block (country-level bonuses/penalties) and opinion effects with other aspects. Aspects are religion-scoped — each definition lists which religions can adopt it via one or more `religion` fields. Mutual incompatibility is enforced via `enabled` triggers that check whether conflicting aspects are already active. Vanilla ships ~16 files of aspects organized by tradition (common, catholic, protestant, etc.).

## Vanilla File Locations
- `in_game/common/religious_aspects/` — ~16 `.txt` files by tradition (+ `common.txt` for cross-tradition aspects)
- Full file list in `_file_index.csv`.

## Block Structure
```pdx
<aspect_id> = {
    # --- Religion eligibility (repeatable) ---
    religion = <religion_id>    # this aspect is available to this religion
    religion = <religion_id>    # multiple entries for aspects shared across religions

    # --- Availability ---
    enabled = {
        # Trigger evaluated on the country; root = country
        # Used to enforce mutual exclusivity:
        NOT = { has_religious_aspect = religious_aspect:<incompatible_id> }
    }

    # --- Effect ---
    modifier = {
        <modifier_key> = <value>
        monthly_religious_influence = <float>
        monthly_towards_communalism = societal_value_monthly_move
        # etc.
    }

    # --- Opinion relationships ---
    opinions = {
        <aspect_id> = <int>    # positive = bonus opinion, negative = penalty
        # typically: same aspect = +10 (solidarity), rival aspects = -20
    }
}
```

## Key Fields Reference
| Field | Purpose | Key constraint |
|---|---|---|
| `religion` (repeatable) | Which religions can adopt this aspect | At least one required; multiple entries allow cross-faith aspects |
| `enabled` | Availability trigger | Root = country; used primarily for mutual exclusivity checks via `has_religious_aspect` |
| `modifier` | Country modifiers while this aspect is active | Same namespace as country modifiers; `monthly_towards_*` can push societal values |
| `opinions` | Aspect-to-aspect opinion modifiers | Positive = countries sharing the aspect have better opinions; negative = tension |
| `has_religious_aspect` | Trigger to check if an aspect is active | Syntax: `has_religious_aspect = religious_aspect:<aspect_id>` |

## Modding Notes
- **Adding a new aspect** requires: a definition block with at least one `religion` entry, localization, and optionally wiring it into the religion's adoption UI (via laws or generic actions). The engine loads all `.txt` files in the folder.
- **`enabled` is the mutual exclusivity mechanism.** There is no built-in exclusion — all conflicts must be scripted as `NOT = { has_religious_aspect = ... }` in the `enabled` trigger. A country will see but cannot adopt an aspect if `enabled` fails.
- **`opinions` keys are aspect IDs**, not religion IDs. `adoptionism = 10` means countries sharing the `adoptionism` aspect have +10 opinion toward each other, regardless of religion.
- **`modifier` stacks** with the religion's own `definition_modifier`, holy site modifiers, and religious school modifiers. Design with the full modifier stack in mind to avoid overpowering combinations.
- **Aspects are not tied to laws by default.** The adoption mechanism (how a country gains or loses an aspect) must be scripted separately via events, generic actions, or laws that call `add_religious_aspect`/`remove_religious_aspect` effects.
- **Cross-system:** aspects interact with societal values (`monthly_towards_*`), estate satisfaction (some aspects boost clergy), and the religious influence resource. The `has_religious_aspect` trigger is usable in all standard trigger contexts.

## Example
`adoptionism` — a Christological doctrine available to four heterodox Christian faiths, incompatible with modalism:

```pdx
adoptionism = {
    religion = bogomilism
    religion = catharism
    religion = lollardy
    religion = paulicianism

    enabled = {
        NOT = { has_religious_aspect = religious_aspect:modalism }
    }

    modifier = {
        global_clergy_desired_pop = 0.01
        stability_cost = -0.1
        monthly_religious_influence = 0.1
        monthly_towards_communalism = societal_value_monthly_move
    }

    opinions = {
        adoptionism = 10    # solidarity with co-adopters
        modalism    = -20   # theological rivalry
    }
}
```

Any country holding one of the four listed religions can adopt `adoptionism` as long as `modalism` is not already active. The `enabled` trigger is the only enforcement — it is scripted, not automatic. Countries sharing this aspect gain +10 mutual opinion, while those with `modalism` face −20.
