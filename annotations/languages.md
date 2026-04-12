# Languages
**Stage:** complete
**Keywords:** language, language_family, color, family, male_names, female_names, dynasty_names, lowborn, court_language, market_language, common_language

> **System type: Data/Reference**

## Overview
Languages define the name pools used for characters and dynasties, associate with a language family for map display, and are referenced by cultures (`language` field in `cultures/`). Language families are an organizational layer grouping related languages for map coloring. Languages interact with EU5's court language, market language, and common language mechanics — the language spoken at court, in trade, and by the common people influences societal value modifiers and cultural assimilation. Vanilla ships ~53 language files and 1 language families file.

## Vanilla File Locations
- `in_game/common/languages/` — ~53 `.txt` files by region (one or more languages per file)
- `in_game/common/language_families/00_language_families.txt` — all language family definitions
- Full file list in `_file_index.csv`.

## Block Structure

### Language Definition
```pdx
<language_id> = {
    color  = <map_color_key>           # map display color (e.g. map_haida_l)
    family = <language_family_id>      # optional; parent language family

    # --- Name pools ---
    male_names    = { <string> <string> ... }   # name pool for male characters
    female_names  = { <string> <string> ... }   # name pool for female characters
    dynasty_names = { <string> <string> ... }   # name pool for dynasties

    # --- Lowborn names (optional) ---
    lowborn = {
        male_names    = { <string> ... }
        female_names  = { <string> ... }
    }
}
```

### Language Family Definition
```pdx
<language_family_id> = {
    color = <map_color_key>   # color for the language family map mode
}
```

## Key Fields Reference
| Field | Purpose | Key constraint |
|---|---|---|
| `family` | Parent language family | Optional; used for map display grouping only — no gameplay effect from the family relationship |
| `male_names` / `female_names` | Character name pools | Names drawn from this pool when generating characters of this culture's language |
| `dynasty_names` | Dynasty name pool | Drawn when generating new dynasties for cultures using this language |
| `lowborn` | Separate lowborn name pools | Optional; if absent, regular name pools are used for lowborn characters |
| `color` | Map color | Jomini palette key for the language map mode display |

## Modding Notes
- **Languages are referenced by cultures**, not directly by countries or pops. A culture's `language = <language_id>` determines which name pools are used for characters of that culture and which language contributes to court/market/common language calculations.
- **Court, market, and common language** are emergent from the cultures present in a country's court, its dominant trading partner, and its most widespread pop culture. The language system provides the name strings; the mechanics that promote a language to "court language" or "market language" status live in the societal values and population systems.
- **`lowborn` name pools** allow cultural differentiation between noble and common characters. Without `lowborn`, the engine uses `male_names`/`female_names` for all characters regardless of estate.
- **Adding a new language** requires an entry in `languages/`, a reference from the associated culture in `cultures/`, and localization. The name pools are plain strings — no localization keys, just names.
- **Language families** are display-only. The `family` field does not affect any gameplay mechanic; it only controls which color a language uses in the language family map mode.
- **Cross-system:** language IDs are referenced in societal value modifier keys (`court_language_is_market_language_importance_modifier`, `court_language_is_liturgical_language_importance_modifier`, `court_language_is_common_language_importance_modifier`). The language that matches the court language provides its associated modifier bonuses/penalties.

## Example
`english_dialect` (abbreviated):

```pdx
english_dialect = {
    color  = map_english_l
    family = germanic_language_family

    male_names    = { William Henry Richard John Robert Edward Thomas ... }
    female_names  = { Margaret Elizabeth Catherine Anne Joan Mary ... }
    dynasty_names = { York Lancaster Plantagenet Tudor Stuart ... }

    lowborn = {
        male_names   = { Tom Jack Will Rob ... }
        female_names = { Bess Nell Kate Meg ... }
    }
}
```

The `family = germanic_language_family` entry groups English with other Germanic languages on the language family map. Noble characters generated for English-culture countries draw from `male_names`/`female_names`; lowborn characters use the `lowborn` sub-pools.
