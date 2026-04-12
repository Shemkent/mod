# AI War (Join War Rules)
**Stage:** complete
**Keywords:** `rule_key`, `join_war_disabled_trigger`

> **System type: AI**

## Overview
Join war rules are named conditions that prevent the AI from joining a specific war as an ally. Each rule defines a `join_war_disabled_trigger` — if it evaluates to true for a country being called into a war, the AI refuses to join. This is a general-purpose blocking mechanism; vanilla uses it for historical/narrative reasons, but it applies to any scenario where the AI should be unconditionally excluded from a war. All rules are checked additively — the AI is blocked if **any** rule's trigger passes. The rule key is also used for localisation (explaining to the player why joining was declined).

## Vanilla File Locations

| File | Content |
|---|---|
| `in_game/common/join_war_rules/readme.txt` | Syntax reference |
| `in_game/common/join_war_rules/teu_crusade.txt` | Teutonic Order crusade: blocks Poland and Bohemia from joining the war if a specific variable is set |

## Block Structure

```pdxscript
rule_key = {
    join_war_disabled_trigger = {
        # Available scopes:
        # root              = the country being asked to join
        # scope:war         = the war (may not exist if trigger fires before war is active; always guard with exists = scope:war)
        # scope:first_leader  = leader of the side the country is joining
        # scope:second_leader = leader of the opposing side
        <triggers>
    }
}
```

The `rule_key` is used as a localisation key; localise it as `rule_key: "Reason text"` to display a tooltip when the AI declines a call to arms.

## Key Fields Reference

| Field | Purpose | Notes |
|---|---|---|
| `rule_key` | Block identifier and localisation key | Localise as `rule_key: "Reason text"` to show tooltip when AI declines; raw key displays if loc is missing |
| `join_war_disabled_trigger` | Blocks AI from joining the war | `root` = potential joiner; `scope:war` may be null if war hasn't started; use `exists = scope:war` guard |

## Modding Notes
- This is an **AI-only** system — players can join any war they are called to through normal UI.
- `scope:war` may not exist when the trigger is evaluated (pre-war call); always guard with `exists = scope:war` before accessing war variables.
- Rule keys should be localised; without a localisation entry the raw key appears in the tooltip when the AI declines. Place the entry in a standard `localization/` file: `rule_key: "Reason text"` — the key is simply the block name.
- Multiple rule files load additively; all rules are checked — the AI is blocked if **any** rule's trigger passes.
- The readme lists `COUNTRY`, `OUR_LEADER_COUNTRY`, `ENEMY_LEADER_COUNTRY`, and `WAR` alongside the trigger scopes. These are localisation scopes for the tooltip text, not trigger scopes inside `join_war_disabled_trigger`; use `root`, `scope:war`, `scope:first_leader`, and `scope:second_leader` in trigger blocks.

## Example

```pdxscript
# Block Poland and Bohemia from joining TEU crusade wars
# (named "truce" for narrative context — not a mechanical truce; the name is the loc key)
TEU_POL_BOH_truce = {
    join_war_disabled_trigger = {
        OR = { tag = POL  tag = BOH }
        exists = scope:war
        exists = scope:war.var:teu_lit_first_war
    }
}
```
