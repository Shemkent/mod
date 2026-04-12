# Europa Universalis V — Game System Annotations

A reference library documenting EU5's script systems for modders. Each annotation explains how a game system works, what fields it exposes, and how it connects to other systems. All annotations verified against v1.1.10.

## Annotation System Map

Cross-cluster dependencies across the 80 annotated systems. Full edge data in [`annotations/_system_edges.json`](annotations/_system_edges.json).

```mermaid
graph LR
    GOV[Government & Politics]
    MIL[Military & War]
    DIP[Diplomacy & Interactions]
    ECO[Economy & Trade]
    CHR[Characters & Estates]
    TEC[Technology & Progression]
    REL[Religion & Culture]
    SCR[Scripted Infrastructure]
    GEO[Geography & Map]
    AI[AI]

    GOV --> CHR
    GOV --> DIP
    GOV --> REL
    CHR --> GOV
    CHR --> DIP
    CHR --> REL
    MIL --> ECO
    MIL --> DIP
    MIL --> GOV
    AI --> DIP
    DIP --> AI
    DIP --> REL
    DIP --> TEC
    DIP --> SCR
    DIP --> ECO
    ECO --> REL
    ECO --> CHR
    ECO --> SCR
    TEC --> SCR
    REL --> TEC
    GEO --> ECO
    GEO --> TEC
```

## Annotations

80 systems documented across 10 clusters (Government & Politics, Military & War, Diplomacy & Interactions, Economy & Trade, Characters & Estates, Technology & Progression, Religion & Culture, Scripted Infrastructure, Geography & Map, AI).

Each annotation covers:
- what the system does and where its files live
- key fields and their accepted values
- how the system interacts with others
- modding notes and gotchas

**Start here:** [`annotations/GAME_STRUCTURE_GUIDE.md`](annotations/GAME_STRUCTURE_GUIDE.md) — full system index grouped by cluster, with links to every annotation.

Machine-readable metadata (stage, cluster, summary per system): [`annotations/_system_index.json`](annotations/_system_index.json)

## Repo Layout

| Path | Contents |
|---|---|
| `annotations/` | Vanilla system reference — 80 systems, v1.1.10 |
| `mod/` | Mod directories, each with `.metadata/` for mod-specific notes |

## Mods

| Mod | Description |
|---|---|
| `mod/better_piracy_workaround/` | Reduces pirate unit durability to manageable levels |
