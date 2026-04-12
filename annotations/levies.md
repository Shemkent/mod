# Levies
**Stage:** complete
**Keywords:** levies, levy, unit, size, allowed_pop_type, allowed_culture, country_allow, recruitment_method, training

> **System type: Gameplay**

## Overview

Levies are pop-based military units that countries can conscript without full recruitment costs. Each levy definition binds a unit type to a pop eligibility filter and a size formula. The engine selects levies in file order — highly specific levies (culture- or pop-restricted) must appear before generic ones, or they will be crowded out. Companion folder `recruitment_method/` defines how levied (and standard) units are trained: normal, elite, or rushed. The engine fills levy slots in file order — highly specific levies appear at the top of each file so they are matched before generic fallbacks.

## Vanilla File Locations

| Path | Purpose |
|---|---|
| `in_game/common/levies/readme.txt` | Field reference |
| `in_game/common/levies/00_revolutions_levies.txt` | Conscripts and generic cavalry |
| `in_game/common/levies/01_absolutism_levies.txt` | Absolutism-era specialized levies |
| `in_game/common/levies/02_reformation_levies.txt` | Reformation-era levies |
| `in_game/common/levies/03_discovery_levies.txt` | Discovery-era levies |
| `in_game/common/levies/04_renaissance_levies.txt` | Renaissance-era levies |
| `in_game/common/levies/05_traditions_levies.txt` | Traditions-era levies |
| `in_game/common/levies/06_tribal_levies.txt` | Tribal government levies |
| `in_game/common/levies/10_traditions_levies_navy.txt` | Naval levies |
| `in_game/common/recruitment_method/readme.txt` | Recruitment method field reference |
| `in_game/common/recruitment_method/army.txt` | Army training methods |
| `in_game/common/recruitment_method/navy.txt` | Naval training methods |

Full file list in `_file_index.csv`.

## Block Structure

### Levy definition
```
levy_key = {
    unit        = <unit_type_key>          # Required. The unit type conscripted.
    size        = <scripted_value|float>   # Required. Number of units granted (Root = country).

    country_allow = { <triggers> }         # Optional. Country-level gate (Root = country).
    allow         = { <triggers> }         # Optional. Pop-level gate (Root = pop).
    allow_as_crew = { <triggers> }         # Optional. Crew eligibility (Root = pop).

    # Pop type and culture filters (multiple allowed; none = all eligible)
    allowed_pop_type = <pop_type_key>      # Restrict to this pop type.
    allowed_culture  = <culture_key>       # Restrict to this culture.
}
```

### Recruitment method definition
```
method_key = {
    army      = yes/no     # Applies to army (yes) or navy (no). Required.
    default   = yes/no     # Whether this is the default method if none specified.
    strength  = <float>    # Multiplier on unit strength (1.0 = normal).
    experience = <int>     # Starting experience value (0–100).
    build_time = <float>   # Multiplier on training duration (1.0 = normal, -0.75 = 75% shorter).
}
```

## Key Fields Reference

| Field | Purpose | Key constraint |
|---|---|---|
| `unit` | Unit type key from `unit_types/` | Must exist in unit_types |
| `size` | Script value for how many units the levy grants | Evaluated per-country |
| `country_allow` | Country-level prerequisite triggers | Root = country |
| `allow` | Pop-level eligibility filter | Root = pop; restricts which pops count |
| `allow_as_crew` | Additional crew eligibility (naval levies) | Root = pop |
| `allowed_pop_type` | Shorthand pop type filter; multiple entries OR-combined | None = all pop types |
| `allowed_culture` | Culture filter; multiple entries OR-combined | None = all cultures |
| `strength` (method) | Unit strength modifier (default 1.0) | Multiplicative |
| `experience` (method) | Starting experience 0–100 | `elite_training` uses 20 |
| `build_time` (method) | Training time multiplier | -0.75 = 75% faster; 0.5 = 50% of normal duration; 1.0 = unchanged |

## Modding Notes

- **File order is critical**: the engine fills levy slots top-to-bottom. Place country- or culture-specific levies above generic ones, or they will never be selected.
- The `size` script value is scoped to the country — use `num_pop_type:soldiers`, `total_population`, or similar.
- `allowed_pop_type` entries are OR-combined (a pop matching any listed type qualifies).
- `allowed_culture` entries are OR-combined similarly.
- `allow` (pop-level) is an additional pop filter on top of `allowed_pop_type` — useful for religion or tech checks.
- `country_allow` is checked once per country, not per pop; gate expensive levies here.
- New recruitment methods need only `army = yes/no` and `default = yes/no`. Custom methods are not surfaced in UI beyond basic selection — no additional event or trigger integration in v1.1.10.
- Cross-system dependency: `unit` references a unit type key from `unit_types/` — the unit must be defined there before the levy can be used.

## Example

### Standard peasant/laborer conscripts
```
levy_conscripts = {
    size = levy_generic_infantry_size
    unit = a_conscripts

    allowed_pop_type = peasants
    allowed_pop_type = laborers
    allowed_pop_type = soldiers
    allowed_pop_type = burghers
}
```
All four pop types qualify; no culture or country restriction — this is the baseline levy available to every country.

### Noble cavalry levy
```
levy_gendarmerie = {
    size = levy_generic_noble_cavalry_size
    unit = a_gendarmerie

    allowed_pop_type = nobles
}
```
Restricted to nobles only. Because it appears after `levy_conscripts` in the same file, conscript slots fill first; gendarmerie fills remaining levy space from nobles.

### Elite recruitment method
```
elite_training = {
    army      = yes
    experience = 20
    build_time = 0.5     # Takes 50% of normal build time (50% faster)
}
```
Units arrive with 20 experience points at the cost of 50% longer build time.
