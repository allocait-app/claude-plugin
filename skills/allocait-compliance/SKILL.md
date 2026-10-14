---
name: allocait-compliance
description: Check an Allocait portfolio against its own allocation rules, or work out the amounts that close the gap to them. Use when the user asks how far their portfolio is from their rules, wants a compliance check, or wants a rebalance preview.
---

Allocait is a process layer for a portfolio, not a brokerage. It never places a trade, never connects to a brokerage, and never recommends buying, selling, or holding anything. Every verdict below is the user's own allocation band, applied to numbers you supply; Allocait itself stores no opinion about any security.

## The one invariant that matters most

**Every price on this surface is one you supply.** Allocait's `market_data` cache is never read back to you, and no tool here fills a price you did not pass. Bring prices from your own source before calling anything below. If you omit a symbol, Allocait tells you exactly which one it still needs (`symbols_to_price`, or the `PRICES_INCOMPLETE` error) rather than guessing.

No way to look up a price? Ask the user. Never use a remembered or estimated price: a wrong price gives a wrong verdict.

## Start here: get the state of the world

1. Call `list_portfolios` (no arguments) to find the `portfolio_id` you need. It returns each portfolio's name, currency and active scenario, its playbooks, monitors and charts, and every scenario. No dollar totals come back here.
2. Call `get_portfolio_snapshot` with that `portfolio_id`. This is the one call to reason from, not a set of separate reads:
   - Without `prices`, the snapshot is unpriced: shares, rules, monitor settings, playbooks and charts come back, but values, `compliance` and every monitor reading are `null`. The response's `symbols_to_price` array lists exactly what to fetch next.
   - Call it again with a `prices` object (`{ "ALFA": 181.5, ... }`, a fictional example, symbol to price in the portfolio's own currency) covering those symbols. The same call then returns values, the compliance verdict and monitor readings, all computed from your numbers only. It has no `rebalance` block unless you add `include_rebalance: true`, which you do only when the user asks what it would take to get back inside a band.
   - `scenario_id` is optional; it defaults to the portfolio's own active scenario, the same one the app checks against.

Answer from the snapshot's `compliance` block:

- `status` is the one-word answer: `compliant` (no breach and nothing unchecked), `partial` (no breach, but some holdings are in `unchecked`), `violations`, or `unjudged` (for example, a portfolio worth $0). `issue_count` counts the breaches.
- `class_violations`: each asset class whose total, Cash included, is outside its allocation band (`band_min_pct`/`band_max_pct`, as % of portfolio value). A class with a band and no holdings counts as 0%, so it is listed whenever its minimum is above 0.
- `size_violations`: each holding outside its class's position size range (`size_min_pct`/`size_max_pct`, for **one holding** in that class). A class with no size range sets no limit.
- `unchecked`: holdings no rule could judge, each with its reason (for example, a holding with no asset class). Mention them; don't treat them as compliant.

Each breach's status is `ABOVE_MAX` or `BELOW_MIN`. Each also carries `outside_by`: `edge` (`above_max` or `below_min`), `pct_points` (how far outside its band or range, in percentage points of portfolio value) and `value` (the dollar amount outside, at your prices). Report it as where the rule stands. It names no trade; amounts come only from the `rebalance` block, and only when the user asks for them.

`prices` must cover every non-cash position before `compliance` (and `rebalance`, when asked for) is computed at all, so every class verdict here is based on all of that class's holdings. Allocait's own dashboard can show a "based on N of M holdings" note for a partly priced class; that note never applies to anything you see through this tool.

## Work out the amounts that close a gap

When the user asks what it would take to get back inside a band, call the snapshot with `prices` and `include_rebalance: true`. Its `rebalance` block holds the arithmetic:

- `trades` (the wire name): the pro rata amounts that bring each class to its target (the rule's target, or the band's midpoint), largest first. Each amount carries `class_target_pct`.
- `resulting_classes`: each class's share after those trades, Cash included. Sale proceeds stay in Cash.
- `unresolved`: any class still outside its band after the trades, such as a class with nothing to buy into. Trades don't resize holdings, so report the size verdicts separately.

This is arithmetic on the user's own bands, not an order and not a suggestion to buy, sell or hold. Report the amounts as amounts. What happens next is the user's decision, made outside Allocait, never yours and never Allocait's.

## To see what a change does, check it first

To check a specific change, such as 40 more shares of a symbol, use `allocait-check` (`check_trades`). Two more read-only tools take a proposed state rather than acting on it:

- **`validate_allocations`**: pass `portfolio_id` and an `allocations` map (symbol to proposed percentage of portfolio value, 0-100; `is_valid` needs them to sum to 100). `scenario_id` is optional and defaults to the active scenario. Returns `sum_pct`, class band breaches (`violations`), and holdings outside their size range (`size_violations`) for that plan; `is_valid` is true only when the sum is 100, both lists are empty and `unknown_symbols` is empty. Propose 0 for an exit; a class the plan leaves empty counts as 0% against its band, the same verdict `compliance` gives once the exit is recorded. For a symbol the user doesn't hold yet, pass `asset_classes` (symbol to class name or id) so it is judged like a holding.
- **`calculate_position_size`**: the share count that sits inside the size range the user set for one holding. See `allocait-portfolio-upkeep`.

Neither tool writes anything. Run one, show the user what it found, and only then consider a write from `allocait-portfolio-upkeep` (to record a trade the user made) or `allocait-policy` (to change the rules) if they ask for one.

## Monitors

The snapshot's `monitors` block, when priced, is a measurement, not a verdict: every level and threshold in it is something the user entered, never something Allocait computed, so its state words only say where price sits relative to a user-set line. Reading it is free and changes nothing. Only a separate, once-daily check treats a level break as confirmed, so do not tell the user a break is settled from an intraday snapshot alone.

## Error codes to branch on

Errors read `Error [CODE]: message`, and the message ends with the next step. The codes that come up here:

- `NOT_FOUND`: an id you do not own (wrong `portfolio_id` or `scenario_id`, or a typo).
- `PRICES_INCOMPLETE`: the message ends with the exact symbols still needed; fetch those and call again.
- `PRICES_INVALID`: a price you passed was negative or not a number. 0 is valid, for example an option that expired worthless.
- `INTERNAL_ERROR`: a failure on Allocait's side; retry once, then report it with `send_feedback`.

## What this skill never does

- Never calls a write tool. If the user wants to act on the rebalance amounts or a validation result, hand off to `allocait-portfolio-upkeep` or `allocait-policy` and confirm with them first.
- Never treats a monitor reading or a compliance verdict as investment advice. State the numbers and the rule; do not add an opinion about buying, selling or holding.
