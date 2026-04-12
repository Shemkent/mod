# Child Educations
**Stage:** complete
**Keywords:** child_education, modifier, country_modifier, price_to_select, price_to_deselect, allow, character_child_education, court_spending_cost

> **System type: Gameplay**

## Overview
Child educations define the educational programs available for child characters at court. Each education applies a character modifier (boosting education speed or stat focus) and an optional country modifier (e.g., court spending cost). Players select and deselect educations for eligible children at a scripted price. At coming-of-age, the accumulated education determines which adult traits the character receives. Two files exist in vanilla: `00_default.txt` with all education definitions and `readme.txt`.

## Vanilla File Locations
- `in_game/common/child_educations/00_default.txt` — all education definitions
- `in_game/common/child_educations/readme.txt` — field documentation
- Full file list in `_file_index.csv`.

## Block Structure
```pdx
<education_id> = {
    # --- Modifiers ---
    modifier = {
        # Applied to the child character while this education is active
        character_child_education    = <float>   # overall education speed multiplier
        character_adm_child_education = <float>  # administrative focus
        character_dip_child_education = <float>  # diplomatic focus
        character_mil_child_education = <float>  # military focus
    }

    country_modifier = {
        # Applied to the country while this education is active
        court_spending_cost = <float>   # modifier to court upkeep cost
    }

    # --- Cost ---
    price_to_select   = <price_key>   # cost to activate this education
    price_to_deselect = <price_key>   # cost to cancel it

    # --- Eligibility ---
    allow = {
        # Trigger; root = child character
    }
}
```

## Key Fields Reference
| Field | Purpose | Key constraint |
|---|---|---|
| `modifier` | Character modifiers on the child | `character_child_education` = overall speed; stat-specific fields focus on adm/dip/mil |
| `country_modifier` | Country-level modifier while education is active | Used for cost effects (e.g. expensive tutors increase court spending) |
| `price_to_select` | Cost to start the education | Paid by the acting country; references `prices/` by ID |
| `price_to_deselect` | Cost to cancel the education | May differ from selection cost |
| `allow` | Eligibility trigger | Root = child character; can test character traits, stats, or owner conditions |

## Modding Notes
- **Adding a new education** requires an entry in `00_default.txt`, price definitions in `prices/` if using new price IDs, and localization.
- **`character_adm/dip/mil_child_education`** determines which adult traits the child is likely to receive at coming-of-age. High `adm` focus makes administrative education traits more probable.
- **`country_modifier`** stacks with other country modifiers. Used to model the economic cost of expensive education programs (private tutors, prestigious academies).
- **Cross-system:** child educations interact with the traits system (which adult traits emerge at coming-of-age), the pricing system (`prices/`), and the character generation system. The education modifiers influence the education score that determines adult trait outcomes.
