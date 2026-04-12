# Policies
**Stage:** complete
**Keywords:** policies, policy, law, law_group, enact_law, on_enact, modifier, laws

> **System type: Data/Reference (stub — no standalone script files)**

## Overview
In EU5 v1.1.10, the `in_game/common/policies/` folder is **empty by design**. Policies are not defined as standalone script files; they are created indirectly from laws. The engine retains the empty folder to avoid database assertion errors on startup. Any behavior commonly associated with "policies" in other Paradox titles is handled through the **laws system** (`in_game/common/laws/`).

## How Policies Work in EU5
Policies in EU5 are law-derived effects. When a country enacts a law, the law definition carries all policy-equivalent content:

- **`modifier`** — country-level modifiers applied while the law is active (equivalent to a policy's ongoing effect)
- **`on_enact`** — one-time scripted effects when the law is passed (equivalent to an adoption cost)
- **`on_revoke`** — effects when the law is removed

Law groups (`law_group`) control which laws are mutually exclusive, replicating the "pick one policy per category" structure familiar from EU4.

See `annotations/laws.md` for full law block documentation.

## Vanilla File Locations
- `in_game/common/policies/policies.info` — engine marker file only; no definitions
- `in_game/common/laws/` — all actual policy-equivalent content
- Full file list in `_file_index.csv`.

## Modding Notes
- **Do not create files in `policies/`.** The engine does not load `.txt` files from this folder in v1.1.10. All policy-equivalent modding is done in `laws/`.
- **To add a new "policy"**, define a new law or law group in `in_game/common/laws/`. The law's `modifier` block provides the ongoing effect, and `on_enact` handles the activation cost.
- **This may change in future versions.** The `policies.info` note says the folder is "kept around for now", indicating policies as a distinct system may be implemented later. Check the folder contents after each patch.
