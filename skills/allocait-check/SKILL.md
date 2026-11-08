---
name: allocait-check
description: Check a change the user describes against their own Allocait rules before they place it elsewhere. Use when the user names shares to add or remove, or a dollar amount, and wants to know what that change does to their allocation bands and position size ranges. Reports rule facts only.
---

This checks a change against the rules the user wrote and returns facts. It does not approve, block or place an order. Allocait output states facts about the user's own rules. It is not advice or an approval. A result inside every range does not say a change is advisable. Allocait cannot approve, stop or place an order.

## What to call

`check_trades` (a read tool; it writes nothing, and it works on a read-only connection):

- `portfolio_id`: from `list_portfolios`.
- `scenario_id` (optional): the scenario to check against. Left out, the portfolio's active scenario is used. If `active_scenario_id` is null the call is refused: pass the scenario the user names, or ask which.
- `trades` (1 to 50): each `{ symbol, side, shares }` or `{ symbol, side, notional }`, with exactly one of `shares` or `notional` (dollars), greater than 0. `side` is the direction of the change the user asked about: `BUY` adds shares, `SELL` removes them. Several changes in one call are a basket, applied together. A `SELL` larger than the shares held is refused, never trimmed, unless the portfolio has Allow shorts on: then a `SELL` can take a holding below zero, a `BUY` can take a short above it, and `shares_after` in the result is signed (negative is a short). An Options holding never goes below zero. For a symbol the portfolio doesn't hold, add `asset_class` (a name, alias or id).
- `prices`: `{ "SYMBOL": number }` for every non-cash symbol the portfolio holds and every symbol in `trades`, except a held option with a mark (it counts at the user's own mark) or one that expired (it counts 0). These are the user's prices. Allocait supplies none and fills none. Ask for any the user hasn't given, or use a source they trust. Never use a remembered or estimated price.

If the user names no change, do not call `check_trades`. Say you can check a specific change they describe.

Steps: `list_portfolios` for the `portfolio_id`. Call `check_trades` with the change exactly as the user described it and every price the user has given (send `prices: {}` if none). If it answers `PRICES_INCOMPLETE`, the message names every symbol that still needs a price: ask the user for all of them in one message, then call again. Use the amounts the user gave. Don't change them to make a check pass.

## Reading the result

All symbols in the examples below are fictional (ALFA, BRAV, GOLF), with made-up numbers.

- `summary` (the first key): the counts in one line, how many rules were outside their range before and after the change and how many the change newly or further put outside or brought back inside. Lead with it.
- `newly_or_further_outside_count`: how many rules the change puts outside their range (`newly_outside`) or leaves further outside (`further_outside`). It answers "did this change add a breach or deepen one". 0 means neither.
- `within_rules`: a different question: is the whole book inside every rule after the change, with cash at 0 or more. A breach that already existed keeps it `false`, and so does `cash.sufficient: false`. `false` with `rules_judged_count: 0` means no rule was checked. Treat `false` as "not confirmed". `true` says nothing about anything no rule covers, and it is never an approval.
- `rules`: only the rules outside their range before or after the change. Each row has `kind` (`class_band`, `position_size` or `short_position_size`), `asset_class`, `symbol` for a size rule, the range (`min_pct`, `max_pct`), `before_pct`, `after_pct`, `gap_before_pts`, `gap_after_pts` and `change`. The four `change` words are `newly_outside`, `further_outside`, `still_outside` and `back_inside`. They describe a rule's distance from its range. They say nothing about the trade. A `position_size` row is a long against its class's size range and a `short_position_size` row a short against its short size range, both by size without a sign. A symbol sold through zero is judged on the side it ends on, so a breach of the range it left is not reported as cleared.
- `not_covered`: symbols that no band or size range looks at (`NO_CLASS_RULE` or `NO_SIZE_RANGE`; `NO_SIZE_RANGE` names a changed symbol whose side has no range: no position size for a long, no short size for a short), including holdings the change does not touch (`traded: false`). Say so, so a `true` isn't read as "a rule looked at this symbol".
- `exposure_after`: how much of the book gains if prices rise once the change is applied. It has `long_pct`, `short_pct`, `net_pct`, `gross_pct` (percents of `total_value`), `options_counted` and `options_not_counted`, at the user's prices and each holding's recorded `moves` (1 unless the user set it). It is a reading, not a verdict: no rule judges it and `within_rules` doesn't read it. Options count at their premium, a call as long and a put as short. Report the figures as figures ("net 77%, gross 95%"), say they are at premium when `options_counted` is above 0, name the symbols when `options_not_counted` is above 0 (a priced option whose symbol is not an option symbol is left out), and never call a figure high, low, safe or too much.
- `cash`: `before`, `after` and `sufficient`.
- `positions_updated_at`: always report it with the result. The book is what Allocait has recorded. Allocait doesn't see the live account.
- `basis`: what the result is computed from. Pass it along when the user asks.

Report in the user's terms and their numbers, one short line per rule. For example:

> At the prices you gave:
> - Adding 41 ALFA takes Equities from 37.3% to 41.02%, above your 39% maximum, and ALFA from 37.3% to 41.02%, above its 39.5% maximum.
> - Selling 11 GOLF leaves Commodities 0.61 points below your 9.5% minimum, further than before.
> - Nothing judged the size of GOLF: it has no size range.
>
> Your positions were last updated 2026-10-09.

## A trade in an expired option

A held option past its expiry counts 0, so a trade in it can't change the book and `check_trades` doesn't check it: the call is refused with `INVALID_INPUT`, naming the trade and the write that closes it. Report that message. Don't send a price to get around it. Recording what the option became is `allocait-portfolio-upkeep`, after the user says what happened.

## A failed call is no result

If `check_trades` returns `INVALID_INPUT`, `PRICES_INVALID`, `NOT_FOUND` or `INTERNAL_ERROR`, or can't be called at all, say there is no result and why, in plain words. (`PRICES_INCOMPLETE` is not a dead end: ask for the prices it names.) Retry `INTERNAL_ERROR` once. If it fails again, tell the user and give them its `request_id`; file a report with `send_feedback` only if the user asks. Don't work out the percentages by hand from the snapshot, and don't describe what the result "probably" would be.

## When the user asks whether to buy

Run the check and report the rule facts. Don't answer yes or no, and don't tell the user to buy, sell or hold anything. Say that the decision is theirs and that Allocait cannot approve, stop or place an order. If the user asks what share count would sit inside a size range they set, that's `calculate_position_size` (see `allocait-portfolio-upkeep`), which returns an amount computed from their range. Never offer a number of your own.

## Standing text for the user's agent

These are the sentences Allocait shows people to paste into an agent's standing instructions. The same wording applies here.

> Use only the numbers I give you. Never suggest, round or fill in a number for my rules. If I have no number for something, leave it out and say so.

> Before I place a trade, call check_trades with the change I describe and the prices I give you. Tell me which of my rules it puts outside range and which it leaves further outside. Do not tell me to buy, sell or hold anything. Do not change the amounts to make a check pass. If a call fails, say there is no result. Allocait can't approve, stop or place an order.

## What this skill never does

- Never writes. Recording a trade the user made is `allocait-portfolio-upkeep`; changing a rule is `allocait-policy`.
- Never calls a result approved, cleared, or advisable, and never calls an exposure figure high, low, safe or risky.
- Never supplies, rounds or adjusts a price, a share count or a rule.
