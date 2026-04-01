# Traits System
**Stage:** annotated
**Game version:** 1.1.10
**Keywords:** traits, character, ruler, general, admiral, artist, explorer, modifier, flavor, chance

---

## Overview

Traits are modifiers attached to individual characters (rulers, generals, admirals, artists, explorers, children). Each trait belongs to a `category` that matches a character role; the modifier only applies when the character is acting in that role. Traits can have flavors for UI grouping, `allow` conditions, and a `chance` block for random acquisition probability.

---

## File Locations

| Path | Purpose |
|---|---|
| `in_game/common/traits/` | All trait definitions, split by character role |
| `in_game/common/traits/_traits.info` | Format reference |

| File | Role |
|---|---|
| `00_ruler.txt` | Ruler traits (personality, government approach, etc.) |
| `01_general.txt` | General traits (army tactics, cavalry, siege, etc.) |
| `02_admiral.txt` | Admiral traits (naval) |
| `03_artist.txt` | Artist traits |
| `04_explorer.txt` | Explorer traits |
| `05_child.txt` | Child traits (affect education speed) |
| `06_religious_figure.txt` | Religious figure traits |

---

## Syntax

```
trait_key = {
    category = ruler|general|admiral|artist|explorer|child   # Required.
    flavor   = <flavor_key>    # Optional. Groups trait in UI (e.g. personality, government_approach).

    allow = { <triggers> }     # Optional. Conditions for this trait to be valid for a character.
                               # Root = character, owner ?= country.

    modifier = { <modifiers> } # Applied when character's active role matches category.

    chance = {                 # MTTH-style block for random trait assignment probability.
        base = <int>
        # modifier blocks...
    }
}
```

---

## Categories

| Category | Character type | Notes |
|---|---|---|
| `ruler` | Ruler/monarch | Affects country-wide ruler modifiers |
| `general` | Army leader | Affects land combat |
| `admiral` | Naval commander | Affects naval combat |
| `artist` | Artist character | Affects cultural output |
| `explorer` | Explorer | Affects exploration |
| `child` | Child heir | Affects education speed (`character_child_education`) |
| `religious_figure` | Religious characters | Affects religion/conversion |

---

## Modifier Scope

Trait modifiers apply to the **character's active scope**, not directly to the country:
- `ruler` traits apply country-wide (owner country).
- `general` / `admiral` traits apply to the unit they command.
- `child` traits apply to the child's education progress.

---

## Flavor Values (ruler traits)

`flavor` is used for UI grouping and does not affect logic. Common values seen in vanilla:

| Flavor | Description |
|---|---|
| `personality` | Core character personality traits |
| `government_approach` | Governing style (tolerant, free_thinker, etc.) |

---

## Examples

### 1. Simple ruler trait
```
just = {
    allow = {
        adm > 33
        NOT = { has_trait = cruel }
    }
    category = ruler
    flavor = personality
    modifier = {
        global_estate_target_satisfaction = tiny_permanent_target_satisfaction
        monthly_towards_free_subjects = societal_value_minor_monthly_move
        peace_offer_fairness = 0.2
    }
}
```
`adm > 33` gates this to administratively capable rulers. Incompatible with `cruel`.

### 2. General trait with unit composition requirement
```
born_to_the_saddle = {
    allow = {
        unit ?= { sub_unit_fraction:army_cavalry > 0.2 }
    }
    category = general
    modifier = {
        army_cavalry_power = 0.1
    }
}
```
Only available to generals commanding armies with at least 20% cavalry.

### 3. Child trait with chance block
```
child_prodigy = {
    category = child
    modifier = {
        character_child_education = 1.5
    }
    chance = {
        base = 1
    }
}
```
The `chance` block controls MTTH probability. `base = 1` with no modifiers = equal weight among all child trait options.

---

## Modding Notes

- Add new traits to the appropriate role file, or a new file — all `.txt` files in the folder are loaded.
- `allow` conditions prevent the trait from being assigned if the character doesn't qualify.
- `flavor` is purely cosmetic/UI — can be any string; new flavor values show up automatically.
- Traits are referenced in scripts as `trait:<trait_key>` (e.g. `has_trait = trait:just`).
- The `?=` operator in `allow` handles the case where `owner` or `unit` may not exist.
