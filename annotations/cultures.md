# Cultures
**Stage:** complete
**Keywords:** culture, culture_group, language, color, tags, opinions, country_modifier, location_modifier, character_modifier, culture_groups, noun_keys, adjective_keys, has_culture_group, primary_culture

> **System type: Data/Reference**

## Overview
Cultures define the ethnic and linguistic identities of pops, characters, and locations in EU5. Each culture sets a language, map color, graphics tags, inter-culture opinion relationships, and optional modifiers applied when the culture is a country's primary culture, a location's dominant culture, or a character's culture. Culture groups are an organizational layer above individual cultures — they provide group-level modifiers and a parent-child relationship that some scripts (e.g., `has_culture_group`) test against. ~52 culture files and 2 culture group files ship in vanilla.

## Vanilla File Locations
- `in_game/common/cultures/` — ~52 `.txt` files by region (one or more cultures per file + `00_cultures.info` template)
- `in_game/common/culture_groups/00_culture_groups.txt` — all culture group definitions
- `in_game/common/culture_groups/00_culture_groups.info` — template
- Full file list in `_file_index.csv`.

## Block Structure

### Culture Definition
```pdx
<culture_id> = {
    # --- Identity ---
    language = <language_id>          # e.g. english_dialect, arabic_language
    color    = <map_color_key>        # e.g. map_ENG — Jomini palette map color

    # --- Graphics ---
    tags = {
        <gfx_tag>    # portrait/flag graphics tags; e.g. english_gfx, european_gfx
        <gfx_tag>
    }

    # --- Modifiers (all optional; empty blocks are valid) ---
    country_modifier   = { <modifier block> }   # when this is the country's primary culture
    location_modifier  = { <modifier block> }   # when this is the location's dominant culture
    character_modifier = { <modifier block> }   # when a character has this culture

    # --- Opinions ---
    opinions = {
        <culture_id> = enemy/negative/neutral/positive/kindred
        # repeatable; omit entries for neutral (default)
    }

    # --- Group membership (repeatable) ---
    culture_groups = {
        <culture_group_id>    # e.g. british_group, slavic_group
    }

    # --- Country name generation ---
    noun_keys      = { <string_keys> }   # nouns for procedural country names
    adjective_keys = { <string_keys> }   # adjectives for procedural country names
}
```

### Culture Group Definition
```pdx
<culture_group_id> = {
    # All modifier blocks are optional; many vanilla groups are entirely empty (= {})
    country_modifier   = { <modifier block> }
    location_modifier  = { <modifier block> }
    character_modifier = { <modifier block> }
}
```

## Key Fields Reference
| Field | Purpose | Key constraint |
|---|---|---|
| `language` | Associates the culture with a language ID | Must match an ID in `languages/`; affects name pools and court/market language calculations |
| `color` | Map display color | Jomini palette key (e.g. `map_ENG`); palette is defined outside `common/` |
| `tags` | Graphics/portrait tags | Controls which portrait assets and flag overlays are used; tags are tested by the portrait system |
| `opinions` | Culture-to-culture relationship | `enemy`/`negative`/`neutral`/`positive`/`kindred`; affects base opinion between cultures' countries; omitted entries default to neutral |
| `culture_groups` | Parent group membership (repeatable) | A culture can belong to multiple groups; tested via `has_culture_group = culture_group:<id>` |
| `country_modifier` | Modifiers when primary culture | Stacks with other country modifiers; empty block `= {}` is valid |
| `location_modifier` | Modifiers when dominant culture | Applied per location where this is the dominant culture |
| `character_modifier` | Modifiers when character's culture | Applied to characters with this culture |
| `noun_keys` / `adjective_keys` | Procedural country name pools | Optional; strings referencing localization keys for name generation; many cultures omit these |

## Modding Notes
- **Adding a new culture** requires an entry in `cultures/`, a `culture_groups` membership (at minimum one group), a `language` reference that exists in `languages/`, and localization. The engine loads all `.txt` files in the folder.
- **Culture groups** can provide additional modifiers that stack with culture-level modifiers. Modifier blocks are optional — many vanilla groups are empty (`= {}`). A culture belonging to multiple groups receives modifiers from all non-empty groups.
- **`opinions`** entries are one-directional. An `enemy` entry for `irish` inside the `english` block does not automatically create a reciprocal `enemy` entry in the `irish` block — that requires an explicit entry there.
- **`tags`** control portrait assets. Adding a new culture with a new ethnic appearance requires new portrait assets in `gfx/` with matching tag names. Without tags, the engine falls back to a default portrait set.
- **`has_culture_group`** trigger syntax: `has_culture_group = culture_group:<group_id>`. Individual culture tests use `culture = culture:<culture_id>`.
- **`language`** affects court language and market language mechanics. A culture whose language is the court language receives bonuses defined in the societal values system (`court_language_is_market_language_importance_modifier`, etc.).
- **Cross-system:** cultures interact with the language system (name pools, court/market language), the population system (dominant culture affects location modifiers), the estate system (dhimmi and tribes estates gate on culture/religion), and the societal values system (cultural influence modifiers).

## Example
`english` — with opinions, graphics tags, and group membership:

```pdx
english = {
    language = english_dialect
    color    = map_ENG
    tags = { english_gfx british_gfx western_european_gfx european_gfx }

    country_modifier   = { }
    character_modifier = { }
    location_modifier  = { }

    opinions = {
        irish         = enemy
        highland      = negative
        norse_gael    = negative
        welsh         = negative
        cornish       = negative
        anglo_irish   = kindred
        norman        = positive
        northumbrian  = kindred
    }

    culture_groups = {
        british_group
    }

    noun_keys      = { adder falcon baron }
    adjective_keys = { black red yellow }
}
```

English belongs to `british_group` and inherits that group's modifiers in addition to any set in its own `country_modifier` block. It has `kindred` relations with `anglo_irish` and `northumbrian`, and `enemy` relations with `irish`. The `tags` control which portrait assets are used for characters with English culture.
