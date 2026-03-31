# EU5 Game File Structure Guide

This guide maps the game's `/in_game/common/` directory for modding reference.

**Index of Major Systems:**
- [Advances](#advances)
- [Age](#age)
- [AI Diplomatic Chance](#ai-diplomatic-chance)
- [Alert Descriptions](#alert-descriptions) *(UI/Presentation)*
- [Artist Types](#artist-types) *(Data/Reference)*
- [Artist Work](#artist-work)
- [Attribute Columns](#attribute-columns) *(UI/Presentation)*
- [Auto Modifiers](#auto-modifiers)
- [Avatars](#avatars)
- [Armies & Military](#armies--military)
- [Buildings](#buildings)
- [Characters & Traits](#characters--traits)
- [Disasters & Events](#disasters--events)
- [Government & Politics](#government--politics)
- [Modifiers & Effects](#modifiers--effects)
- [Missions & Content](#missions--content)
- [Religion & Culture](#religion--culture)
- [Trade & Economy](#trade--economy)
- [War & Diplomacy](#war--diplomacy)

---

## Advances

See: [`advances/`](advances.md) — 100+ files; named by scope (country tag, culture, government type, country type, religion, estate, geography)
Keywords: `advance_id`, `age`, `potential`, `allow`, `requires`, `government`, `country_type`, `for`, `unlock_unit`, `unlock_building`, `unlock_law`, `unlock_levy`, `unlock_government_reform`, `unlock_production_method`, `modifier_while_progressing`

---

## Age

See: [`age/`](../age.md)
Keywords: `age_[id]`, `year`, `price_stability`, `max_price`, `hegemons_allowed`, `efficiency`, `unique`, `modifier`, `mercenaries`, `victory_card`, `war_score_from_battles`, `months_for_exploration_spread`, `max_ai_privilege_per_estate`, `min_ai_privilege_per_estate`

---

## AI Diplomatic Chance

See: [`ai_diplochance/`](ai_diplochance.md)
Keywords: `action_key`, `base`, `opinion`, `trust_in_actor`, `rank_difference`, `different_religion`, `different_culture`, `war_exhaustion`, `in_debt`, `yesman`, `enforced_demand`, `relative_strength`, `border_distance`

---

## Alert Descriptions
**Type: UI / Presentation** — display config only; trigger logic is elsewhere.
See: [`alert_descriptions/`](alert_descriptions.md)
Keywords: `alert_key`, `title`, `texture`, `priority`, `game_concept`

---

## Artist Types
**Type: Data/Reference** — named specializations filtering what art a country can produce.
See: [`artist_types/`](artist_types.md)
Keywords: `artist_type_id`, `potential`

---

## Artist Work
See: [`artist_work/`](artist_work.md)
Keywords: `work_type_id`, `captured`, `allow`, `location_modifier`, `country_modifier`, `religion_scale_modifier`

---

## Attribute Columns
**Type: UI/Presentation** — column layout and sort config for selection lists; tightly coupled to GUI widgets.
See: [`attribute_columns/`](attribute_columns.md)
Keywords: `object_type`, `column_id`, `widget`, `width`, `fixed_height`, `is_constant_width`, `sort`

---

## Auto Modifiers
See: [`auto_modifiers/`](auto_modifiers.md)
Keywords: `modifier_id`, `potential_trigger`, `limit`, `scales_with`, `requires_real`, `hide_effects`, `alert`, `category`, `type`

---

## Avatars
See: [`avatars/`](avatars.md)
Keywords: `avatar_id`, `god`, `potential`, `allow`, `country_modifier`, `location_modifier`, `on_activate`, `on_fully_activated`, `on_deactivate`

