# Artist Types
**Stage:** complete
**Keywords:** `artist_type_id`, `potential`

> **System type: Data/Reference**
> Defines which artist specializations exist and when they are available to a country. Used as a filter by `artist_work` entries and character interactions.

## Overview
Artist types are named specializations (e.g. `painter`, `composer`, `calligrapher`) that determine what works of art a country's artists can produce. Each type optionally has a `potential` trigger restricting it to certain religions, court languages, or advance requirements. The type is assigned to characters via game logic and then cross-referenced by `artist_work` to gate which work types they can create.

## Vanilla File Locations

| File | Content |
|---|---|
| `in_game/common/artist_types/00_default.txt` | All 13 artist type definitions |
| `in_game/common/artist_types/readme.txt` | Format documentation |

## Block Structure
```
artist_type_id = {
    potential = {          # Optional country-scope trigger; omit = always available
        OR = {
            religion = religion:orthodox
            court_language = language:arabic_language
        }
    }
}
```

## Key Fields Reference

| Field | Purpose | Notes |
|---|---|---|
| `potential` | Country-scope availability condition | Omit for universally available types |

## Modding Notes
- The block body is empty for universal types (e.g. `painter = {}`).
- `potential` uses country scope — the country must meet it for this artist type to be hirable/assignable.
- New types need matching localization and should be referenced in `artist_work` entries to be useful.
- Cross-system: `artist_work` `allow` blocks filter by `artist_type`; character trait or interaction systems assign the type to a character.

## Example
```
calligrapher = {
    potential = {
        OR = {
            court_language = language:arabic_language
            court_language.language_family ?= language_family:chinese_language_family
            court_language = language:japanese_language
            # ... other East/South Asian language families
        }
    }
}

painter = {    # Universal — no potential block
}
```
*`calligrapher` is restricted to specific court languages; `painter` is available to all countries.*
