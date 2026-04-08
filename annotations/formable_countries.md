# Formable Countries
**Stage:** complete
**Keywords:** formable_countries, form_effect, potential, allow, level, required_locations_fraction, capital_required, rule, continents, regions, areas, locations, tag, flag, color

> **System type: Gameplay**

## Overview

Formable countries are scripted nation-forming decisions that let a country transform into a new polity (e.g. Scandinavia, Persia, Holy Roman Empire). Each definition specifies which territories must be held, a `rule` for plausibility filtering (historical/plausible/fantasy), and `form_effect` for what happens when the country forms. Players can only form countries of a higher `level` than their current rank; the same restriction applies to the AI.

## Vanilla File Locations

| Path | Purpose |
|---|---|
| `in_game/common/formable_countries/readme.txt` | Field reference |
| `in_game/common/formable_countries/00_formable_countries.txt` | All vanilla formable definitions |

Full file list in `_file_index.csv`.

## Block Structure

```
formable_key = {
    level = <int>                       # Minimum country rank level required to form.
    required_locations_fraction = <float>  # Fraction of listed territories that must be owned (default 1.0).
    capital_required = yes/no           # Must the capital be in the listed territories? Default no.
    rule = historical/plausible/fantasy # Plausibility filter tied to game rules. Default historical.

    potential = { <triggers> }          # Root = country. Whether the form button is visible.
    allow     = { <triggers> }          # Root = country. Whether the form button is enabled.

    form_effect = { <effects> }         # Root = country. Effects executed on formation.

    # --- Identity fields ---
    name      = <loc_key>               # New country name.
    flag      = <flag_tag>              # Flag tag.
    adjective = <loc_key>               # Adjective localisation.
    tag       = <country_tag>           # New country tag (3-letter).
    color     = <map_color_key>         # Map color after formation.

    # --- Territory requirements ---
    continents    = { <continent_keys> }
    sub_continents = { <sub_continent_keys> }
    regions       = { <region_keys> }
    areas         = { <area_keys> }
    locations     = { <location_keys> }
}
```

## Key Fields Reference

| Field | Purpose | Key constraint |
|---|---|---|
| `level` | Minimum rank level to form | Checked against current country rank level |
| `required_locations_fraction` | What fraction of listed territories must be owned | 0..1; default 1.0 (all) |
| `capital_required` | Whether the capital must be inside listed territories | Default no |
| `rule` | Plausibility gate matching game rule `historical/plausible/fantasy` | Controls UI visibility per game rule setting |
| `potential` | Visibility trigger | Root = country |
| `allow` | Enable trigger | Root = country; checked when player clicks form |
| `form_effect` | Formation effects | Root = country; fires once on formation |
| `tag` | New 3-letter country tag | Must be defined in country history files |
| `color` | Map color post-formation | References a color key |

## Modding Notes

- Territory lists (`continents`, `regions`, `areas`, `locations`) are OR-combined within each block but all listed blocks must contribute toward the `required_locations_fraction`.
- `rule = historical` shows the formable only in historical and plausible game rule modes; `plausible` shows it in plausible and fantasy; `fantasy` shows it only in fantasy mode.
- `potential` is purely a UI filter; `allow` is what the engine enforces. Keep `potential` broad and `allow` precise.
- Formation does not automatically grant the new tag — the `form_effect` block must do so with `change_tag = tag:X` or equivalent effects.
- New formable countries require matching entries in country history/data files for the tag, flag art, and localization.

## Example

Scandinavia (abbreviated):
```
SCA_f = {
    level = 3
    required_locations_fraction = 0.75
    rule = plausible

    potential = {
        culture = { has_culture_group = culture_group:scandinavian_group }
    }

    allow = { }

    name      = SCA
    flag      = SCA
    adjective = SCA_ADJ
    tag       = SCA
    color     = map_scandinavia

    regions = { region_scandinavia  region_sweden  region_norway }
}
```
Only `level = 3` (kingdom tier) or higher countries can form Scandinavia. The 75% threshold means three-quarters of Scandinavian region locations must be owned, not all of them. The `plausible` rule hides it in historical-only game rules.
