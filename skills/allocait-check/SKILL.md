---
name: allocait-check
description: Check a change the user describes against their own Allocait rules before they place it elsewhere. Use when the user names shares to add or remove, or a dollar amount, and wants to know what it does to their allocation bands and position size ranges, or asks whether to buy, sell or hold something. Reports rule facts only.
---

This checks a change against the rules the user wrote and returns facts. It does not approve, block or place an order. Allocait output states facts about the user's own rules; it is not advice or an approval, and Allocait cannot approve, stop or place an order.

## What to call

`check_trades` (a read tool; it writes nothing, and it works on a read-only connection):

- `portfolio_id`: from `list_portfolios`.
- `scenario_id` (optional): the rule set to check against. Left out, the portfolio's active scenario is used.
- `trades` (1 to 50): each `{ symbol, side, shares }` or `{ symbol, side, notional }`, with exactly one of `shares` or `notional` (dollars), greater than 0. `side` is the direction of the change the user asked about: `BUY` adds shares, `SELL` removes them. Several changes in one call are a basket, applied together. A `SELL` larger than the shares held is refused, never trimmed. For a symbol the portfolio doesn't hold, add `asset_class` (a name, alias or id).
- `prices`: `{ "SYMBOL": number }` for every non-cash symbol the portfolio holds and every symbol in `trades`. These are the user's prices. Allocait supplies none and fills none. Without a number from the user or their own source for each symbol, ask. Never use a remembered or estimated price.

Steps: `list_portfolios` for the `portfolio_id`; `get_portfolio_snapshot` without prices to read `symbols_to_price`; ask for any price the user hasn't given; then `check_trades` with the change exactly as the user described it. Use the amounts the user gave. Don't change them to make a check pass.

## Reading the result

All symbols in the examples below are fictional (ALFA, BRAV, GOLF), with made-up numbers.

- `newly_or_further_outside_count`: how many rules the change puts outside their range (`newly_outside`) or leaves further outside (`further_outside`). It answers "did this change add a breach or deepen one". 0 means neither.
- `within_rules`: a different question. Is the whole book inside every rule after the change. A breach that already existed keeps it `false`. `false` with `rules_judged_count: 0` means no rule was checked. Treat `false` as "not confirmed". `true` says nothing about anything no rule covers, and it is never an approval.
- `rules`: only the rules outside their range before or after the change. Each row has `kind` (`class_band` or `position_size`), `asset_class`, `symbol` for a size rule, the range (`min_pct`, `max_pct`), `before_pct`, `after_pct`, `gap_before_pts`, `gap_after_pts` and `change`. The four `change` words are `newly_outside`, `further_outside`, `still_outside` and `back_inside`. They describe a rule's distance from its range. They say nothing about the trade.
- `not_covered`: symbols that no band or size range looks at (`NO_CLASS_RULE` or `NO_SIZE_RANGE`), including holdings the change does not touch (`traded: false`). Say so, so a `true` isn't read as "a rule looked at this symbol".
- `cash`: `before`, `after` and `sufficient`.
- `positions_updated_at`: always report it with the result. The book is what Allocait has recorded. Allocait doesn't see the live account.
- `basis`: what the result is computed from. Pass it along when the user asks.

Report in the user's terms and their numbers. For example: "With these prices, adding 40 ALFA takes your ALFA size rule from 12.4% to 15.9%, above its 14% maximum, and leaves your Fixed Income band 2.9 points below its minimum, further than before. The GOLF size rule has no range, so nothing judged it. Positions last updated 2026-10-09."

## A failed call is no result

If `check_trades` returns an error (`INVALID_INPUT`, `PRICES_INCOMPLETE`, `PRICES_INVALID`, `NOT_FOUND`, `INTERNAL_ERROR`) or can't be called at all, say there is no result and why, in the error's own words. Retry `INTERNAL_ERROR` once. Don't work out the percentages by hand from the snapshot, and don't describe what the result "probably" would be.

## When the user asks whether to buy

Run the check and report the rule facts. Don't answer yes or no, and don't tell the user to buy, sell or hold anything. Say that the decision is theirs and that Allocait cannot approve, stop or place an order. If the user asks what share count would sit inside a size range they set, that's `calculate_position_size` (see `allocait-portfolio-upkeep`), which returns an amount computed from their range. Never offer a number of your own.

## Standing text for the user's agent

These are the sentences Allocait shows people to paste into an agent's standing instructions. The same wording applies here.

> Use only the numbers I give you. Never suggest, round or fill in a number for my rules. If I have no number for something, leave it out and say so.

> Before I place a trade, call check_trades with the change I describe and the prices I give you. Tell me which of my rules it puts outside range and which it leaves further outside. Do not tell me to buy, sell or hold anything. Do not change the amounts to make a check pass. If a call fails, say there is no result. Allocait can't approve, stop or place an order.

## What this skill never does

- Never writes. Recording a trade the user made is `allocait-portfolio-upkeep`; changing a rule is `allocait-policy`.
- Never calls a result approved, cleared, or advisable.
- Never supplies, rounds or adjusts a price, a share count or a rule.
