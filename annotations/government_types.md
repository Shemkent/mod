# Government Types System
**Stage:** complete
**Game version:** 1.1.10
**Keywords:** government_types, government_power, heir_selection, map_color, modifier

> **System type: Gameplay**

## Overview

Government types are the top-level classification of a country's ruling system. Each country has exactly one government type at a time. The type determines:
- Which **government power** resource the country uses (legitimacy, republican_tradition, etc.)
- Which **heir selection** methods are available
- The **map color** key for the political map mode
- A baseline **modifier** applied to all countries of that type
- Which **laws** and **reforms** are available (via `law_gov_group` and `government = <type>` in those files)

---

## Vanilla File Locations

| Path | Purpose |
|---|---|
| `in_game/common/government_types/00_default.txt` | All government type definitions |

---

## Block Structure

```
government_type_key = {
    # --- Power resource ---
    government_power = <resource_key>   # e.g. legitimacy, republican_tradition, devotion, horde_unity, tribal_cohesion

    # --- Heir selection methods (list all supported types) ---
    heir_selection = <method_key>
    heir_selection = <method_key>
    # ...

    # --- Presentation ---
    map_color = <color_key>             # Political map mode color reference.
    use_regnal_number = yes/no          # Ruler displayed with ordinal (Henry VIII vs Henry).
    generate_consorts = yes/no          # Whether consorts are generated for rulers.

    # --- Estate default ---
    default_character_estate = <estate_type>  # Estate characters default to if unassigned.

    # --- Baseline modifier ---
    modifier = { <modifiers> }          # Applied to all countries of this type.

    # --- Other ---
    revolutionary_country_antagonism = <int>  # Antagonism generated toward revolutionary countries.
}
```

---

## Key Fields Reference

| Field | Purpose | Key constraint |
|---|---|---|
| `government_power` | The power resource this government type uses (e.g. `legitimacy`, `republican_tradition`, `devotion`, `horde_unity`, `tribal_cohesion`). | One per government type; must match a defined resource. |
| `heir_selection` | Heir selection method valid for this government type. Repeated to list all supported methods. | Active method for a specific country is determined by that country's laws, not this field directly. |
| `map_color` | Color key used for this government type in the political map mode. | References a color defined in the map color tables. |
| `use_regnal_number` | When `yes`, rulers are displayed with an ordinal (e.g. Henry VIII rather than Henry). | Boolean; optional. |
| `generate_consorts` | Whether consorts are generated for rulers of this government type. | Boolean; optional. |
| `default_character_estate` | Estate characters default to if no explicit estate is assigned; affects AI estate interaction. | Optional; estate type key. |
| `modifier` | Permanent baseline modifier applied to all countries of this government type. | Use sparingly — prefer reforms for granular control. |
| `revolutionary_country_antagonism` | Antagonism score generated toward revolutionary countries each month; higher values make hostile interactions more likely. | Integer; optional. |

---

## Vanilla Government Types

| Key | Power resource | Notes |
|---|---|---|
| `monarchy` | `legitimacy` | Heir succession; `use_regnal_number = yes` |
| `republic` | `republican_tradition` | Term-based succession |
| `theocracy` | `devotion` | Clergy-led; forces subject conversion |
| `steppe_horde` | `horde_unity` | War-focused; cheap no-CB, looting bonus |
| `tribe` | `tribal_cohesion` | Cheap diplomacy; reduced food consumption |

---

## Heir Selection Methods

Each government type lists the `heir_selection` methods that are valid for countries of that type. The active method for a country is set by their current **laws** and **reforms** (not directly in government_types). Examples:

| Method | Used by |
|---|---|
| `cognatic_primogeniture` | monarchy |
| `salic_law` | monarchy |
| `elective_succession` | monarchy (unlocked by law) |
| `republic_2_year_terms` | republic |
| `oligarchic_elective` | republic |
| `bishopric_elective` | theocracy |
| `tribal_oldest_male` | tribe, steppe_horde |

---

## Example

The vanilla `monarchy` definition (abbreviated):
```
monarchy = {
    government_power     = legitimacy
    use_regnal_number    = yes
    generate_consorts    = yes
    map_color            = government_map_color_monarchy
    default_character_estate = nobles_estate
    revolutionary_country_antagonism = 10

    heir_selection = cognatic_primogeniture
    heir_selection = salic_law
    heir_selection = absolute_cognatic_primogeniture
    heir_selection = elective_succession
    heir_selection = agnatic_primogeniture
    heir_selection = tanistry
    # ... additional methods

    modifier = {
        legitimacy_monthly    = 0.005
        global_legitimacy_cap = 100
    }
}
```
The `heir_selection` list declares every method the engine allows for monarchies; which one a specific country uses is determined by their active laws.

---

## Modding Notes

- Government types are rarely modded directly; most customization is done via **reforms** and **laws**.
- To add a new government type: add a new block to `00_default.txt`, define a `government_power` resource (or reuse an existing one), and list valid `heir_selection` methods.
- `modifier` on a government type is a permanent global baseline — use sparingly; prefer reforms for granular control.
- Adding a new `government_power` requires parallel scripting of the resource UI and script values that cap and increment it — government_types.txt alone is not sufficient.
- New `heir_selection` method keys must be registered both here (in the list) and in heir_selections/ definitions.
