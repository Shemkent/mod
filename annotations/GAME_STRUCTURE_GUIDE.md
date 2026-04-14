# EU5 Game File Structure Guide

Covers `in_game/common/` (115 folders) and `main_menu/common/` (14 entries).
Annotation status is kept in `_file_index.csv`.

---

## Annotated Systems

Cluster membership and metadata live in [`_system_index.json`](_system_index.json).

### Government & Politics
[cabinet_actions](cabinet_actions.md) · [country_ranks](country_ranks.md) · [formable_countries](formable_countries.md) · [government_reforms](government_reforms.md) · [government_types](government_types.md) · [laws](laws.md) · [parliament](parliament.md) · [policies](policies.md)

### Military & War
[casus_belli](casus_belli.md) · [hegemons](hegemons.md) · [levies](levies.md) · [subject_military_stances](subject_military_stances.md) · [unit_formation_preference](unit_formation_preference.md) · [unit_types](unit_types.md) · [wargoals](wargoals.md)

### Diplomacy & Interactions
[country_interactions](country_interactions.md) · [generic_actions](generic_actions.md) · [international_organizations](international_organizations.md) · [resolutions](resolutions.md) · [scripted_relations](scripted_relations.md) · [situations](situations.md) · [subject_types](subject_types.md)

### Economy & Trade
[buildings](buildings.md) · [employment_systems](employment_systems.md) · [goods](goods.md) · [pop_types](pop_types.md) · [production_methods](production_methods.md)

### Characters & Estates
[character_interactions](character_interactions.md) · [child_educations](child_educations.md) · [estates](estates.md) · [ethnicities](ethnicities.md) · [genes](genes.md) · [heir_selections](heir_selections.md) · [regencies](regencies.md) · [traits](traits.md)

### Technology & Progression
[advances](advances.md) · [age](age.md) · [disasters](disasters.md) · [diseases](diseases.md) · [institutions](institutions.md) · [missions](missions.md) · [technologies](technologies.md)

### Religion & Culture
[avatars](avatars.md) · [cultures](cultures.md) · [gods](gods.md) · [holy_sites](holy_sites.md) · [languages](languages.md) · [religions](religions.md) · [religious_aspects](religious_aspects.md) · [religious_focuses](religious_focuses.md) · [religious_schools](religious_schools.md) · [societal_values](societal_values.md)

### Scripted Infrastructure
[auto_modifiers](auto_modifiers.md) · [modifiers](modifiers.md) · [on_action](on_action.md) · [script_values](script_values.md) · [scripted_effects](scripted_effects.md) · [scripted_geography](scripted_geography.md) · [scripted_lists](scripted_lists.md) · [scripted_modifiers](scripted_modifiers.md) · [scripted_rules](scripted_rules.md) · [scripted_triggers](scripted_triggers.md) · [static_modifiers](static_modifiers.md)

### Geography & Map
[location_ranks](location_ranks.md) · [map_geography](map_geography.md) · [road_types](road_types.md) · [town_setups](town_setups.md)

### UI / Presentation
[alert_descriptions](alert_descriptions.md) · [artist_types](artist_types.md) · [artist_work](artist_work.md) · [attribute_columns](attribute_columns.md) · [customizable_localization](customizable_localization.md) · [death_reason](death_reason.md) · [effect_localization](effect_localization.md) · [historical_scores](historical_scores.md) · [scriptable_hints](scriptable_hints.md) · [scripted_country_names](scripted_country_names.md) · [scripted_guis](scripted_guis.md) · [trigger_localization](trigger_localization.md)

### AI
[ai_biases](ai_biases.md) · [ai_diplochance](ai_diplochance.md) · [ai_generic_actions](ai_generic_actions.md) · [ai_rivals](ai_rivals.md) · [ai_war](ai_war.md) · [scripted_diplomatic_objectives](scripted_diplomatic_objectives.md)

### Not yet annotated
game_rules · scenarios · achievements · game_concepts · music_player_tracks · tutorial · named_colors · coat_of_arms · flag_definitions

---

## Annotation Index

System metadata (stage, cluster, version) is maintained in [`_system_index.json`](_system_index.json).
System relationships are maintained in [`_system_edges.json`](_system_edges.json).

## System Relationships

The annotation project tracks cross-system relationships as a typed edge graph for future interactive visualization.

**Data files:**
- `_system_index.json` — node registry: system id, label, annotation path, stage, cluster, verified game version, summary
- `_system_edges.json` — edge list: source → target with typed relationships

Edge type definitions are in [`.claude/annotator.md`](../.claude/annotator.md#graph-maintenance).

## TODO
- [x] **Interactive graph viewer** — `graph.html`; Cytoscape.js; click node → annotation, edge-type filters, cluster focus, weight-scaled nodes, word-cloud layout
