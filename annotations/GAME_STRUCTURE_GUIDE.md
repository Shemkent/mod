# EU5 Game File Structure Guide

Covers `in_game/common/` (115 folders) and `main_menu/common/` (14 entries).
Annotation status is kept in `_file_index.csv`. This guide is the roadmap.

## Priority Tiers

**Tier 1 — Core gameplay** *(annotate first)*
Buildings, laws, government types & reforms, advances, religions, traits, technologies, production methods, disasters, estates & privileges, scripted effects & triggers, modifiers

**Tier 2 — Diplomacy & war**
Casus belli, wargoals, peace treaties, subject types, international organizations, country interactions, generic actions

**Tier 3 — Economy & population**
Goods, prices, goods demand, pop types, levies, employment systems, production methods, policies

**Tier 4 — Characters & culture**
Character interactions, child educations, cultures, culture groups, societal values, traits, trait flavor, ethnicities, genes

**Tier 5 — Religion**
Religions, religion groups, religious aspects/factions/figures/focuses/schools, gods, holy sites, avatars

**Tier 6 — Scripted infrastructure**
Script values, scripted lists, scripted modifiers, scripted rules, scripted geography, scripted relations, scripted country names, scripted diplomatic objectives, on_action, scripted_guis, scripted_hints, trigger/effect localization

**Tier 7 — AI**
AI diplomatic chance, biases, rival criteria, generic action AI lists, join war rules

**Tier 8 — UI / Presentation / Niche**
Alert descriptions, attribute columns, artist types & work, insults, death reason, designated heir reason, customizable localization, music player tracks, tutorial lessons, road types, topography, vegetation, climates, town setups, location ranks, country ranks, historical scores, persistent DNA

---

## Domain Clusters

### Government & Politics
| Folder | Likely System |
|---|---|
| `in_game/common/government_types/` | government_types |
| `in_game/common/government_reforms/` | government_reforms |
| `in_game/common/laws/` | laws |
| `in_game/common/policies/` | policies |
| `in_game/common/cabinet_actions/` | government_types |
| `in_game/common/parliament_types/` | government_types |
| `in_game/common/parliament_agendas/` | government_types |
| `in_game/common/parliament_issues/` | government_types |
| `in_game/common/regencies/` | government_types |
| `in_game/common/resolutions/` | government_types |
| `in_game/common/hegemons/` | government_types |
| `in_game/common/formable_countries/` | government_types |
| `in_game/common/country_ranks/` | government_types |
| `main_menu/common/game_rules/` | game_rules |
| `main_menu/common/scenarios/` | scenarios |

### Military & War
| Folder | Likely System |
|---|---|
| `in_game/common/unit_types/` | unit_types |
| `in_game/common/unit_categories/` | unit_types |
| `in_game/common/unit_abilities/` | unit_types |
| `in_game/common/unit_formation_preference/` | unit_types |
| `in_game/common/levies/` | levies |
| `in_game/common/recruitment_method/` | levies |
| `in_game/common/wargoals/` | wargoals |
| `in_game/common/casus_belli/` | casus_belli |
| `in_game/common/peace_treaties/` | wargoals |
| `in_game/common/join_war_rules/` | wargoals |
| `in_game/common/subject_military_stances/` | subject_types |

### Diplomacy & International
| Folder | Likely System |
|---|---|
| `in_game/common/subject_types/` | subject_types |
| `in_game/common/international_organizations/` | international_organizations |
| `in_game/common/international_organization_payments/` | international_organizations |
| `in_game/common/international_organization_land_ownership_rules/` | international_organizations |
| `in_game/common/international_organization_special_statuses/` | international_organizations |
| `in_game/common/country_interactions/` | country_interactions |
| `in_game/common/generic_actions/` | generic_actions |
| `in_game/common/generic_action_ai_lists/` | ai_generic_actions |
| `in_game/common/diplomatic_costs/` | country_interactions |
| `in_game/common/scripted_diplomatic_objectives/` | scripted_diplomatic_objectives |
| `in_game/common/scripted_relations/` | scripted_relations |
| `in_game/common/rival_criteria/` | ai_rivals |
| `in_game/common/insults/` | country_interactions |
| `in_game/common/situations/` | situations |

### Economy & Trade
| Folder | Likely System |
|---|---|
| `in_game/common/goods/` | goods |
| `in_game/common/prices/` | goods |
| `in_game/common/goods_demand/` | goods |
| `in_game/common/goods_demand_category/` | goods |
| `in_game/common/production_methods/` | production_methods |
| `in_game/common/building_types/` | buildings |
| `in_game/common/building_categories/` | buildings |
| `in_game/common/pop_types/` | pop_types |
| `in_game/common/employment_systems/` | employment_systems |
| `in_game/common/policies/` | policies |

### Estates & Characters
| Folder | Likely System |
|---|---|
| `in_game/common/estates/` | estates |
| `in_game/common/estate_privileges/` | estates |
| `in_game/common/character_interactions/` | character_interactions |
| `in_game/common/child_educations/` | child_educations |
| `in_game/common/heir_selections/` | heir_selections |
| `in_game/common/traits/` | traits |
| `in_game/common/trait_flavor/` | traits |
| `in_game/common/genes/` | genes |
| `in_game/common/persistent_dna/` | genes |
| `in_game/common/ethnicities/` | ethnicities |

### Technology & Progression
| Folder | Likely System |
|---|---|
| `in_game/common/advances/` | advances |
| `in_game/common/technologies/` | technologies *(folder absent in v1.1.10; see advances)* |
| `in_game/common/age/` | age |
| `in_game/common/institution/` | institutions |
| `in_game/common/disasters/` | disasters |
| `in_game/common/missions/` | missions |
| `in_game/common/mission_task_defs/` | missions |
| `main_menu/common/achievements/` | achievements |
| `main_menu/common/achievement_groups.txt` | achievements |
| `main_menu/common/scenarios/` | scenarios |

### Religion & Culture
| Folder | Likely System |
|---|---|
| `in_game/common/religions/` | religions |
| `in_game/common/religion_groups/` | religions |
| `in_game/common/religious_aspects/` | religious_aspects |
| `in_game/common/religious_factions/` | religious_schools |
| `in_game/common/religious_figures/` | religious_schools |
| `in_game/common/religious_focuses/` | religious_focuses |
| `in_game/common/religious_schools/` | religious_schools |
| `in_game/common/gods/` | gods |
| `in_game/common/holy_site_types/` | holy_sites |
| `in_game/common/holy_sites/` | holy_sites |
| `in_game/common/avatars/` | avatars |
| `in_game/common/culture_groups/` | cultures |
| `in_game/common/cultures/` | cultures |
| `in_game/common/societal_values/` | societal_values |
| `in_game/common/language_families/` | languages |
| `in_game/common/languages/` | languages |

### Modifiers & Effects (Scripted Infrastructure)
| Folder | Likely System |
|---|---|
| `in_game/common/modifiers/` | modifiers *(conceptual guide, no single folder)* |
| `in_game/common/static_modifiers/` | static_modifiers *(folder absent in v1.1.10)* |
| `in_game/common/auto_modifiers/` | auto_modifiers |
| `in_game/common/scripted_modifiers/` | scripted_modifiers |
| `in_game/common/scripted_effects/` | scripted_effects |
| `in_game/common/scripted_triggers/` | scripted_triggers |
| `in_game/common/scripted_rules/` | scripted_rules |
| `in_game/common/script_values/` | script_values |
| `in_game/common/scripted_lists/` | scripted_lists |
| `in_game/common/on_action/` | on_action |
| `in_game/common/scriptable_hints/` | scriptable_hints |
| `in_game/common/scripted_guis/` | scripted_guis |
| `in_game/common/scripted_country_names/` | scripted_country_names |
| `in_game/common/scripted_geography/` | scripted_geography |
| `in_game/common/effect_localization/` | effect_localization |
| `in_game/common/trigger_localization/` | trigger_localization |
| `in_game/common/customizable_localization/` | customizable_localization |
| `main_menu/common/static_modifiers/` | static_modifiers |
| `main_menu/common/scripted_triggers/` | scripted_triggers |
| `main_menu/common/scripted_lists/` | scripted_lists |
| `main_menu/common/script_values/` | script_values |
| `main_menu/common/modifier_type_definitions/` | modifiers |
| `main_menu/common/modifier_icons/` | modifiers |

### AI
| Folder | Likely System |
|---|---|
| `in_game/common/ai_diplochance/` | ai_diplochance |
| `in_game/common/biases/` | ai_biases |
| `in_game/common/rival_criteria/` | ai_rivals |
| `in_game/common/generic_action_ai_lists/` | ai_generic_actions |
| `in_game/common/join_war_rules/` | ai_war |

### Geography & Map
| Folder | Likely System |
|---|---|
| `in_game/common/climates/` | map_geography |
| `in_game/common/topography/` | map_geography |
| `in_game/common/vegetation/` | map_geography |
| `in_game/common/road_types/` | map_geography |
| `in_game/common/location_ranks/` | map_geography |
| `in_game/common/town_setups/` | map_geography |
| `in_game/common/scripted_geography/` | scripted_geography |

### UI / Presentation / Data
| Folder | Likely System |
|---|---|
| `in_game/common/alert_descriptions/` | alert_descriptions |
| `in_game/common/attribute_columns/` | attribute_columns |
| `in_game/common/artist_types/` | artist_types |
| `in_game/common/artist_work/` | artist_work |
| `in_game/common/music_player_tracks/` | music_player_tracks |
| `in_game/common/tutorial_lesson_chains/` | tutorial |
| `in_game/common/tutorial_lessons/` | tutorial |
| `in_game/common/historical_scores/` | historical_scores |
| `in_game/common/country_description_categories/` | ui_data |
| `in_game/common/death_reason/` | ui_data |
| `in_game/common/designated_heir_reason/` | ui_data |
| `main_menu/common/game_concepts/` | game_concepts |
| `main_menu/common/named_colors/` | ui_data |
| `main_menu/common/coat_of_arms/` | ui_data |
| `main_menu/common/flag_definitions/` | ui_data |

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

- [ ] **Mermaid diagram** — generate static flowchart from `_system_index.json` + `_system_edges.json` once annotation pass is substantially complete
- [ ] **Interactive graph viewer** — D3/Cytoscape/vis.js consuming both JSON files; features: click node → annotation, filter by edge type, cluster zoom, version-stale coloring
- [ ] **main_menu systems** — `game_rules`, `scenarios`, `achievements`, `game_concepts`, `named_colors`, `coat_of_arms`, `flag_definitions` not yet annotated
- [ ] **Tutorial/UI stubs** — `tutorial_lessons`, `tutorial_lesson_chains`, `music_player_tracks` skipped (not modding-relevant)
