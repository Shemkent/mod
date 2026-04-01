# Laws System
**Stage:** annotated
**Game version:** 1.1.10
**Keywords:** laws, policies, law_category, country_modifier, estate_preferences, government

---

## Overview

A **law** is a named slot that holds one active **policy** at a time. Laws represent broad policy areas (succession, military levies, trade, religion, etc.); policies are the concrete choices within them. Only one policy per law can be active; switching costs a price and takes an implementation time.

Laws apply to countries (most laws) or international organizations (IO laws — see readme.txt for IO-specific fields).

---

## File Locations

| Path | Purpose |
|---|---|
| `in_game/common/laws/` | All law and policy definitions |
| `in_game/common/laws/readme.txt` | Full attribute reference including IO fields |

### File organisation (by theme)

| File(s) | Contents |
|---|---|
| `00_monarchy.txt` | Monarchy-specific laws (succession, levies) |
| `00_republic.txt` | Republic-specific laws (elections, foundations) |
| `00_theocracies.txt` | Theocracy laws |
| `00_tribes.txt` | Tribal government laws |
| `01_common.txt` | Universal laws (education, trade, religion, socioeconomic) |
| `01_military_laws.txt` | Military policy laws |
| `01_naval_laws.txt` | Naval policy laws |
| `01_legal_system.txt` | Legal system laws |
| `02_country_specific.txt` | Country-tag-gated laws |
| `02_distribution_of_power.txt` | Power distribution laws |
| `03_estate_laws.txt` | Estate-specific laws (nobles, burghers, clergy, peasants) |
| `04_order_of_chivalry.txt` | Military order laws |
| `10_reforms_from_events.txt` | Laws unlocked by events |
| `20_hre.txt`, `20_shogunate.txt`, etc. | IO / confederation laws |
| `40_personal_unions.txt` | Personal union laws |
| `colonial_laws.txt` | Colonial laws |
| `christian_tenets.txt`, `sects.txt`, etc. | Religion-specific laws |

---

## Syntax

### Law block
```
law_key = {
    law_category   = <category>       # Required. Groups law in UI tabs.
    law_gov_group  = monarchy|republic|theocracy|tribe  # Optional. Restricts to a gov type.
    law_religion_group = <religion>   # Optional. Restricts to a religion/group.
    law_country_group  = <tag>        # Optional. Restricts to a country tag.
    unique         = yes/no           # Default: no.
    requires_vote  = yes/no           # IO only. Passing a policy requires a vote.
    custom_tags    = { <strings> }
    show_tags_in_ui = yes/no

    potential = { <triggers> }        # Root = country. Whether the law is displayed at all.
    allow     = { <triggers> }        # Root = country. Whether the law can be changed.
    locked    = { <triggers> }        # Root = country. Whether the law is locked (uninteractable).

    # Everything else is a policy key:
    policy_key = { ... }
    policy_key = { ... }
}
```

### Policy block
```
policy_key = {
    # --- Availability ---
    potential = { <triggers> }      # Root = country. Whether this policy option is visible.
    allow     = { <triggers> }      # Root = country. Whether this policy can be activated.
    unique    = yes                 # Marks a culture/tag-specific policy variant.

    # --- Cost & timing ---
    price     = <price>             # Gold or other resource cost.
    years     = <int>               # Implementation time in years.
    months    = <int>               # (alternative to years)
    weeks     = <int>               # (alternative)
    days      = <int>               # (alternative)

    # --- Effects ---
    on_pay_price       = { <effects> }  # Root = country. Fired when price is paid.
    on_activate        = { <effects> }  # Root = country. Fired when policy is chosen.
    on_fully_activated = { <effects> }  # Root = country. Fired at 100% implementation.
    on_deactivate      = { <effects> }  # Root = country. Fired when policy is removed/replaced.

    # --- Modifiers (scaled+triggered modifier syntax) ---
    country_modifier  = { <modifiers> }   # Applied to the country.
    province_modifier = { <modifiers> }   # Applied to provinces.
    location_modifier = { <modifiers> }   # Applied to locations.

    # --- Estate interactions ---
    estate_preferences = {
        <estate_type>   # Estates that prefer this policy.
    }

    # --- AI weights ---
    wants_this_policy_bias = <scripted_maths>
}
```

---

## law_category Values

| Category | Used for |
|---|---|
| `administrative` | Governance, succession, foundation |
| `military` | Army composition, levy rules |
| `socioeconomic` | Trade, education, social policy |
| `religious` | Religious tolerance, church relations |
| `estates` | Estate rights and powers |
| `centralization` | Power distribution, autonomy |
| `election` | Republic electoral systems |
| `foreign_policy` | Diplomatic posture |
| `foundation` | Constitutional foundation laws |
| `federal` | Confederate/IO structure |
| `free_city` | HRE free city laws |
| `leadership` | IO leadership rules |
| `elector` | HRE elector laws |

---

## Modifier Scaling

Policy modifiers support a scaled+triggered form:
```
country_modifier = {
    scale = <scripted_maths>
    potential_trigger = { <triggers> }
    <modifier_key> = <value>
}
```
When no `scale` or `potential_trigger` is present, the modifier applies fully upon 100% implementation. During implementation, modifiers scale from 0 to full.

---

## Examples

### 1. Simple monarchy law with two policies
```
feudal_de_jure_law = {
    law_category = administrative
    law_gov_group = monarchy
    potential = { government_type = government_type:monarchy }

    by_tradition = {
        country_modifier = {
            cultural_tradition_modifier = 0.05
            monthly_towards_inward = societal_value_monthly_move
        }
        years = 2
        estate_preferences = { nobles_estate  clergy_estate  peasants_estate }
    }

    by_blood = {
        country_modifier = {
            global_integration_speed_modifier = 0.10
            monthly_towards_outward = societal_value_monthly_move
        }
        years = 2
        estate_preferences = { burghers_estate }
    }
}
```

### 2. Common law gated by building and with per-policy potential
```
education_elites_law = {
    law_category = socioeconomic
    potential = {}
    allow = { total_effective_building_levels:university > 0 }

    forums_of_thought = {
        potential = {
            NOT = { has_estate_privilege = estate_privilege:clergy_strengthen_faith }
            OR = {
                "estate_power(estate_type:crown_estate)" >= 0.6
                "estate_power(estate_type:clergy_estate)" <= 0.25
            }
        }
        country_modifier = {
            clergy_estate_target_satisfaction = medium_permanent_target_satisfaction_penalty
            global_institution_growth_modifier = 0.20
        }
        estate_preferences = { peasants_estate  burghers_estate }
        years = 2
    }

    clerical_institutions = {
        potential = { NOT = { government_type = government_type:theocracy } }
        country_modifier = {
            clergy_estate_target_satisfaction = medium_permanent_target_satisfaction
            global_clergy_max_literacy = 10
        }
        estate_preferences = { clergy_estate }
        years = 2
    }
}
```

### 3. Policy with on_activate effects
```
nobles_electorate_policy = {
    potential = { government_type = government_type:monarchy }
    country_modifier = {
        nobles_estate_target_satisfaction = medium_permanent_target_satisfaction
        global_nobles_estate_power = 0.1
        monthly_legitimacy = 0.05
    }
    estate_preferences = { nobles_estate }
    years = 2
    on_activate = {
        unlock_heir_selection_effect = { type = elective_succession }
    }
    on_deactivate = {
        lock_heir_selection_effect = { type = elective_succession }
    }
}
```
`on_activate` / `on_deactivate` unlock non-modifier gameplay features that can't be expressed as modifiers.

---

## Modding Notes

- New laws: add a top-level block to any laws file (or a new file).
- New policies in an existing law: add a new key inside the law block — any key that isn't a reserved attribute is treated as a policy.
- `law_gov_group`, `law_religion_group`, `law_country_group` are efficiency shortcuts — equivalent to a `potential` trigger but handled by the engine.
- Always fill `estate_preferences` for policies that meaningfully favour or disfavour an estate — it drives AI and estate opinion signals.
- For culture-unique or tag-unique policies, set `unique = yes` on the policy.
