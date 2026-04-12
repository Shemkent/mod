# Death Reasons, Heir Designation Reasons & Country Description Categories
**Stage:** complete
**Keywords:** `death_reason`, `designated_heir_reason`, `country_description_categories`, `possible_parameter`

> **System type: UI/Presentation**
> **Cluster:** UI / Presentation

## Overview
Three small enumeration files that register named IDs used elsewhere in the game for display and localisation purposes. **Death reasons** label how a character died (battle, disease, execution, etc.) and can carry optional contextual parameters. **Designated heir reasons** label why a character was chosen as heir (dynasty marriage, ruler preference, etc.). **Country description categories** label the thematic sections of a country's lobby profile (diplomatic, administrative, military).

---

## Death Reasons

### Vanilla File Locations
`in_game/common/death_reason/` — 3 files:
- `00_hardcoded.txt` — engine-driven causes (battle, siege, disease, marching, etc.)
- `01_content.txt` — script-driven causes (execution, assassination, seppuku, etc.)
- `02_life_expectancy.txt` — natural death causes

### Block Structure

```pdx
assassination = {
    possible_parameter = location  # optional context passed to the death message
    possible_parameter = killer    # can declare multiple parameters
}

battle = {
    possible_parameter = location
    # hardcoded — the engine fires this; do not use in script effects
}

# 02_life_expectancy.txt entries use a weighted-random structure:
old_age = {
    random = yes                   # eligible for random natural-death selection
    trigger = {                    # [optional] character must meet this to be eligible
        age_in_years >= 60
    }
    weight = {                     # relative selection weight (script value or flat int)
        value = age_in_years
        subtract = 60
        multiply = 4
    }
}

choking_accident = {
    random = yes
    weight = 1                     # flat weight; no trigger = always eligible
}
```

### Key Fields

| Field | Purpose | Notes |
|---|---|---|
| `possible_parameter` | Optional contextual data | `location`, `killer`, `disease_outbreak`; determines what appears in the death tooltip |
| `random` | Marks entry as eligible for random natural-death selection | Only in `02_life_expectancy.txt` entries |
| `trigger` | Eligibility condition for random selection | Root = dying character; omit = always eligible |
| `weight` | Relative selection weight | Script value or flat integer; higher = more likely |

### Modding Notes
- `00_hardcoded.txt` entries are fired by the engine; they cannot be triggered from script. Modifying them has no effect on when they occur.
- New death reasons in `01_content.txt` can be passed to `kill_character = { death_reason = my_reason killer = ... }` effects.
- Localisation keys follow the pattern `death_reason_<id>` and `death_reason_<id>_desc`.
- `possible_parameter` is a declaration — it does not require the parameter to be present, but enables it to display in tooltips when provided.

---

## Designated Heir Reasons

### Vanilla File Locations
`in_game/common/designated_heir_reason/` — 1 file:
- `00_standard.txt`

### Block Structure

```pdx
married_into_ruling_dynasty = { }
preferred_heir_of_ruler = { }
designated_as_appanage = { }
prince_in_exile = { }
corruler = { }
sibling_of_ruler = { }
infant_emperor = { }
```

Empty blocks — the key is the ID; no fields are defined. Localisation key: `designated_heir_reason_<id>`.

### Modding Notes
- New reasons can be created freely; they are referenced by heir-designation effects and displayed in character tooltips.
- No triggers or conditions are tied to these IDs in script — they are display-only.

---

## Country Description Categories

### Vanilla File Locations
`in_game/common/country_description_categories/` — 2 files:
- `readme.txt` — explains the system
- `categories.txt` — defines the three categories

### Block Structure

```pdx
diplomatic = {}
administrative = {}
military = {}
```

Empty blocks; the key is the category ID. Referenced from country definition files (`in_game/setup/countries/`) to assign lobby profile text to a category section.

### Localisation Keys

| Key | Purpose |
|---|---|
| `country_description_category_name_<id>` | Category header label |
| `country_description_category_desc_<id>` | Category description body |

### Modding Notes
- Categories are used to organise the country selection lobby. Adding a new category requires corresponding loc keys and country definition entries.
- Safe to add; the vanilla three (diplomatic, administrative, military) cover all base-game countries.
