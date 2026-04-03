# Religions
**Stage:** complete
**Keywords:** religion, religion_group, group, color, language, definition_modifier, opinions, tags, has_karma, has_purity, has_honor, max_sects, factions, religious_focuses, allow_slaves_of_same_group, convert_slaves_at_start

> **System type: Data/Reference**

## Overview
Religion definitions assign each faith to a religion group, give it a map color and graphics tags, declare which mechanics it uses (`has_karma`, `has_purity`, `has_honor`), set its `definition_modifier` (passive country bonuses while the religion is held), and define opinion relationships with other religions. Religion groups are the organizational layer above individual religions — they control slavery mechanics and provide optional group-level modifiers. Religions also reference factions (`factions {}`) and religious focuses (`religious_focuses {}`). Vanilla ships ~29 religion files and 1 religion group file.

## Vanilla File Locations
- `in_game/common/religions/` — ~29 `.txt` files (one or more religions per file, organized by tradition)
- `in_game/common/religion_groups/00_default.txt` — all religion group definitions
- Full file list in `_file_index.csv`.

## Block Structure

### Religion Definition
```pdx
<religion_id> = {
    # --- Identity ---
    color    = <color_key>             # map/UI display color
    group    = <religion_group_id>     # parent group (e.g. christian, buddhist, muslim)
    language = <language_id>           # liturgical language (optional)

    # --- Mechanics flags ---
    has_karma  = yes/no    # enables karma mechanic for this religion
    has_purity = yes/no    # enables purity mechanic
    has_honor  = yes/no    # enables honor mechanic
    max_sects  = <int>     # maximum number of active sects (optional)

    # --- Country modifiers (while a country holds this religion) ---
    definition_modifier = {
        <modifier_key> = <value>
        can_have_monasteries = yes   # flag modifier example
    }

    # --- Opinion relationships ---
    opinions = {
        <religion_id> = kindred/positive/neutral/negative/enemy   # OR a flat integer (e.g. -20)
    }

    # --- Graphics ---
    tags = { <gfx_tag> ... }   # portrait/UI tags; e.g. buddhist_gfx, shinto_gfx

    # --- Sub-systems (optional; listed by ID, defined in their own folders) ---
    factions = {
        <faction_id>    # religious factions available to countries of this religion
    }
    religious_focuses = {
        <focus_id>      # research focuses available to countries of this religion
    }
}
```

### Religion Group Definition
```pdx
<religion_group_id> = {
    color = <color_key>

    allow_slaves_of_same_group  = yes/no   # can countries of this group enslave same-group pops
    convert_slaves_at_start     = yes/no   # slaves convert to owner religion at game start

    modifier = {
        # Optional group-level modifier applied to all countries in this group
        allow_rgo_slave_demand = yes   # example: Muslim group enables slave RGO demand
    }
}
```

## Key Fields Reference
| Field | Purpose | Key constraint |
|---|---|---|
| `group` | Assigns the religion to a parent group | Must match an ID in `religion_groups/`; groups affect slavery and opinion mechanics |
| `definition_modifier` | Country modifiers active while holding this religion | Same modifier namespace as country modifiers; applied permanently while the religion is state religion |
| `opinions` | Relationship to other religions | Values can be named (`kindred`) or flat integers; used by AI for tolerance and antagonism calculations |
| `has_karma` / `has_purity` / `has_honor` | Religion-specific resource mechanics | Enables the corresponding resource bar in the UI and related triggers/effects; only meaningful if supporting scripts exist |
| `max_sects` | Cap on active sects | Limits how many sect variants can be simultaneously active for countries of this religion |
| `factions` | Religious factions available | References IDs in `religious_factions/`; faction is only shown if its `visible` trigger passes |
| `religious_focuses` | Research focus list | References IDs in `religious_focuses/`; each focus is an independent researchable doctrine |
| `allow_slaves_of_same_group` (group) | Slavery restriction | `no` prevents enslaving pops of the same group (e.g. Christians cannot enslave Christians) |
| `convert_slaves_at_start` (group) | Starting conversion | `yes` converts all owned slaves to the owner's religion at scenario start |
| `modifier` (group) | Group-level country modifier | Applied to all countries in the group; stacks with `definition_modifier` on individual religions |

## Modding Notes
- **Adding a new religion** requires: an entry in `religions/`, a valid `group` reference, localization, and at minimum a `color`. The engine loads all `.txt` files in the folder.
- **`definition_modifier`** provides the baseline country bonuses for holding a faith. These stack with modifiers from `religious_aspects`, `holy_sites`, and `religious_schools`.
- **`opinions`** entries are one-directional. An `enemy` entry from A to B does not create reciprocal enmity — add entries in both religion files for symmetric relationships.
- **Mechanics flags** (`has_karma`, `has_purity`, `has_honor`) only enable the UI resource bar and related script functions. The actual karma/purity/honor gain/spend logic must be scripted in events, laws, and effects.
- **`factions` and `religious_focuses`** are lists of IDs; the definitions live in `religious_factions/` and `religious_focuses/` respectively. A faction or focus not listed here is invisible to countries of this religion even if its own `potential` trigger passes.
- **Religion groups are minimal** — most vanilla groups have only `color` and slavery flags. The `modifier` block is optional and used only where group-wide mechanics are needed (e.g. the Muslim group's `allow_rgo_slave_demand`).
- **Cross-system:** religion IDs are referenced throughout EU5 — in estate triggers, culture opinions, holy sites, religious aspects, societal value modifiers, and country history files. Renaming a religion breaks all references silently.

## Example
`bon` — a Tibetan Buddhist religion with karma, literacy bonus, and kindred opinion toward related faiths:

```pdx
bon = {
    color = color_bon
    group = buddhist
    language = tibetan_language

    has_karma = yes

    definition_modifier = {
        global_max_literacy = 5
        global_estate_target_satisfaction = small_permanent_target_satisfaction
        monthly_karma = 0.1
        can_have_monasteries = yes
    }

    opinions = {
        dongba_religion  = kindred
        tibetan_buddhism = kindred
    }

    tags = { buddhist_gfx }
}
```

`bon` belongs to the `buddhist` group, inheriting its slavery rules and group modifier. The `definition_modifier` grants literacy and estate satisfaction to any country that holds Bon as its state religion. The `kindred` opinions with related Tibetan faiths create positive AI disposition between their adherents.
