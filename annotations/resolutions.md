# Resolutions
**Stage:** complete
**Keywords:** `resolutions`, `international_organization_type`, `votes`, `can_vote`, `requires_vote`, `proposal_price`, `effect`, `reject_effect`, `vote_effect`, `ai_will_do`

> **System type: Gameplay**
> **Cluster:** Diplomacy & International

## Overview
Resolutions define the proposal-and-vote system for International Organizations and Situations. A country (the proposer) initiates a resolution, which may either be enacted unilaterally or put to a vote among eligible members. Votes are weighted, time-limited, and can trigger effects on pass, reject, or per individual vote. The `select_trigger` pipeline — shared with country_interactions and cabinet_actions — allows resolutions to gather targets (countries, locations, values) before proposing. Used for IO elections, crusade calls, law enactment, papal edicts, and more.

## Vanilla File Locations
`in_game/common/resolutions/` — 20 files including `readme.txt`. Full list in `_file_index.csv`.

## Block Structure

```pdx
my_resolution = {
    international_organization_type = <io_type>  # which IO or situation this resolution belongs to

    loc = "my_resolution_base_key"               # [optional] base loc override; defaults to block key

    potential = { ... }             # visibility; scope:actor = proposer country
    allow = { ... }                 # enabledness; scope:proposer, scope:recipient = IO

    can_vote = { ... }              # which members may cast votes; scope:actor = testing country

    is_live = { ... }               # [optional] is this resolution currently active (elections)

    proposal_price = price:my_price             # cost to propose; references common/prices/
    proposal_price_modifier = { value = 1.0 }  # [optional] multiplier on proposal price
    proposal_payer = { ... }                    # [optional] who pays proposal cost; default: proposer
    proposal_payee = { ... }                    # [optional] who receives proposal cost; default: nobody

    price = price:my_price                      # cost on pass
    price_modifier = { value = 1.0 }
    payer = { ... }
    payee = { ... }

    requires_vote = { always = yes }            # trigger returning yes = must be voted; no = unilateral
    requires_explicit_votes = yes               # yes for elections (no auto-vote from standing opinion)

    votes = {                        # script value: vote weight per eligible country
        add = { desc = "[cardinal_s|e]"  value = scope:actor.num_cardinals }
    }
    total_votes_needed = { value = 10 }         # [optional] threshold to pass

    should_finalize_vote = { ... }   # trigger: finalize now?
                                     # extra scopes: scope:total_votes_needed, scope:total_votes_available,
                                     #               scope:highest_vote

    years = 1                        # deadline before auto-finalizing; can use months/weeks/days

    vote_ongoing_modifier = {        # modifier applied to member countries while vote is open
        monthly_prestige = -0.1
    }

    cooldown = {                     # [optional] cooldown after enactment
        type = recently_called_crusade
        years = 10
    }

    select_trigger = { ... }         # [repeatable] target selection stages (same as country_interactions)
                                     # last select_trigger is what AI evaluates for vote scoring

    propose_effect = { ... }         # on proposal; scope:proposer, scope:recipient, scope:target, ...
    effect = { ... }                 # on pass; + scope:price, scope:payer, scope:payee
    reject_effect = { ... }          # on rejection
    vote_effect = { ... }            # when a country votes; scope:actor = voter, scope:vote = vote
    abstain_effect = { ... }         # when a country removes its vote

    ai_will_select = { value = 5 }  # AI score for proposing; scope:actor, scope:recipient, scope:target
    ai_will_do = { value = 10 }     # AI score for voting yes (or candidate preference for elections)
    ai_proposer_risk = { value = 2 } # how much AI fears rejection
    ai_tick = monthly               # how often AI evaluates this; keyword: daily / monthly

    show_message = no               # [optional] suppress outcome messages
}
```

## Key Fields Reference

| Field | Purpose | Notes |
|---|---|---|
| `international_organization_type` | IO or situation this resolution belongs to | References IO type id |
| `potential` / `allow` | Visibility and enabledness | Same semantics as country_interactions |
| `can_vote` | Who may vote | Scope:actor = country being tested |
| `requires_vote` | Unilateral vs. voted | Trigger returning yes/no |
| `votes` | Vote weight per country | Script value; scope:actor = voting country |
| `total_votes_needed` | Winning threshold | Optional; default behaviour when omitted is engine-determined |
| `should_finalize_vote` | When to end voting early | Checked each tick; deadline takes precedence if set |
| `years`/`months` etc. | Hard deadline | AI auto-votes at deadline regardless of `should_finalize_vote` |
| `proposal_price` / `price` | Proposal and pass costs | References `common/prices/` |
| `select_trigger` | Target selection stages | Repeatable; last stage is what's voted on |
| `effect` / `reject_effect` | Outcomes on pass/fail | Full effect scope available |
| `vote_effect` / `abstain_effect` | Per-vote callbacks | scope:active_resolution = active resolution object |
| `ai_will_do` | AI vote scoring | For yes/no votes: positive = yes; for elections: used to rank candidates |
| `cooldown` | Post-enactment lockout | `type` = cooldown id, `years`/`months` = duration |

## Modding Notes
- `select_trigger` is identical to country_interactions — see that annotation for the full parameter list (`source_flags`, `interaction_source_list`, `column`, `map_color`, etc.).
- The last `select_trigger` is always the target that countries actually vote *on* (e.g. "which candidate" for an election). Earlier stages gather context (e.g. "which IO").
- `requires_explicit_votes = yes` is for elections where votes must be cast explicitly; when `no`, AI countries auto-vote at the deadline rather than waiting for explicit input.
- `proposal_price` and `price` are separate: one is paid to propose, the other on passing. Both optional.
- Scopes available in `effect`: `scope:proposer`, `scope:recipient` (IO), `scope:target`, `scope:target_1`+, `scope:price`, `scope:payer`, `scope:payee`.
- `vote_ongoing_modifier` applies to all member countries (not just proposer) while vote is live.
- Cross-references: IOs (`international_organizations`), prices (`common/prices/`), situation system, country interactions for `select_trigger` syntax.

## Example

```pdx
call_crusade = {
    international_organization_type = catholic_church

    potential = {
        crusade_potential_trigger = yes
        papacy_active = yes
    }

    allow = {
        scope:actor ?= { is_member_of_international_organization = international_organization:catholic_church }
        religion:catholic = { modifier:curia_actions_blocked = no }
    }

    can_vote = {
        international_organization:catholic_church = {
            country_has_special_status = { type = special_status:curia  country = scope:actor }
        }
    }

    votes = {
        add = { desc = "[cardinal_s|e]"  value = scope:actor.num_cardinals }
    }

    proposal_price = price:propose_curia_action
    months = 6

    cooldown = { type = recently_called_crusade  years = 10 }

    select_trigger = {              # stage 1: pick the IO (always catholic_church here)
        looking_for_a = international_organization
        target_flag = recipient
        visible = { international_organization_type = international_organization_type:catholic_church }
    }
    select_trigger = {              # stage 2: pick the crusade target country
        looking_for_a = country
        target_flag = target
        name = "crusade_select_country"
        visible = { is_valid_target_for_crusade = yes }
    }
    select_trigger = {              # stage 3: cast vote (yes/no boolean — this is what members vote on)
        looking_for_a = boolean
        target_flag = vote
        name = "select_vote"
    }
}
```
*Cardinal-weighted papacy vote to call a crusade against a chosen country, locked behind a 10-year cooldown.*
