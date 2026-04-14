# Europa Universalis V — Game System Annotations

A reference library documenting EU5's script systems for modders. Each annotation explains how a game system works, what fields it exposes, and how it connects to other systems. All annotations verified against v1.1.10.

## [→ Interactive System Graph](https://shemkent.github.io/EU5-Annotation-Modding/)

85 systems across 11 clusters — click any node to read its annotation. Filter by relationship type, zoom to a cluster, or search by name.

## Annotations

85 systems documented across 11 clusters (Government & Politics, Military & War, Diplomacy & Interactions, Economy & Trade, Characters & Estates, Technology & Progression, Religion & Culture, Scripted Infrastructure, UI/Presentation, Geography & Map, AI).

Each annotation covers:
- what the system does and where its files live
- key fields and their accepted values
- how the system interacts with others
- modding notes and gotchas

**Start here:** [`annotations/GAME_STRUCTURE_GUIDE.md`](annotations/GAME_STRUCTURE_GUIDE.md) — full system index grouped by cluster, with links to every annotation.

Machine-readable graph data: [`annotations/_system_index.json`](annotations/_system_index.json) · [`annotations/_system_edges.json`](annotations/_system_edges.json)

## Repo Layout

| Path | Contents |
|---|---|
| `index.html` | Interactive system graph viewer (GitHub Pages) |
| `annotations/` | Vanilla system reference — 85 systems, v1.1.10 |
| `mod/` | Mod directories, each with `.metadata/` for mod-specific notes |

## Mods

| Mod | Description |
|---|---|
| `mod/better_piracy_workaround/` | Reduces pirate unit durability to manageable levels |
