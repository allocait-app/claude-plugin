---
name: allocait-compliance
description: Check an Allocait portfolio against its own allocation rules, or plan a rebalance toward them. Use when the user asks whether their portfolio is in line with their rules, wants a compliance check, or wants to plan or preview a rebalance.
---

Allocait is a process layer for a portfolio, not a brokerage. It never places a trade, never connects to a brokerage, and never recommends buying, selling, or holding anything. Every verdict below is the user's own allocation band, applied to numbers you supply; Allocait itself stores no opinion about any security.

## The one invariant that matters most

**Every price on this surface is one you supply.** Allocait's `market_data` cache is never read back to you, and no tool here fills a price you did not pass. Bring prices from your own source before calling anything below. If you omit a symbol, Allocait tells you exactly which one it still needs (`symbols_to_price`, or the `PRICES_INCOMPLETE` error) rather than guessing.

## Start here: get the state of the world

1. Call `list_portfolios` (no arguments) to find the `portfolio_id` you need. It returns name, currency, active scenario, and position count for every portfolio the caller owns. No dollar totals come back here.
2. Call `get_portfolio_snapshot` with that `portfolio_id`. This is the one call to reason from, not a set of separate reads:
   - Without `prices`, the snapshot is unpriced: shares, position rules, and any recorded monitor levels come back, but `current_price`, `position_value`, `total_value`, `compliance`, `rebalance`, and every monitor reading are `null`. The response's `symbols_to_price` array lists exactly what to fetch next.
   - Call it again with a `prices` object (`{ "AAPL": 231.40, ... }`, symbol to price in the portfolio's own currency) covering those symbols, and the same call returns values, per-asset-class compliance verdicts, a rebalance preview, and monitor readings, all computed from your numbers only.
   - `scenario_id` is optional; it defaults to the portfolio's own active scenario.

Read the snapshot's `compliance` block for the verdict per asset class: each rule carries `min_position_pct`/`max_position_pct` (the allocation band on the class total, as % of portfolio value) and the state is one of the verdict words the snapshot returns (in range, below the minimum, above the maximum). A rule's `size_min_pct`/`size_max_pct`, when present, is the position-size range for **one holding** in that class; compliance never reads it; that range is what `allocait-portfolio-upkeep`'s `calculate_position_size` uses instead.

Because `prices` must cover every non-cash position before `compliance`/`rebalance` are computed at all, every class verdict here is always based on all of that class's holdings — there is no partial-pricing case to disclose on this surface. Allocait's own dashboard shows a per-class "based on N of M holdings" note when a class's verdict rests on less than its whole total; that note never applies to anything you see through this tool.

The snapshot's `monitors` block, when present, is a measurement, not a verdict: every level and threshold in it is something the user entered, never something Allocait computed, so its state words only say where price sits relative to a user-set line. Reading it is free and changes nothing; only a separate, once-daily mechanism (not this call) ever treats a level break as confirmed, so do not tell the user a break is settled from an intraday snapshot alone.

## Before proposing any change, check it first

Two read-only, no-write tools exist for this, both taking a proposed state rather than acting on it:

- **`validate_allocations`**: pass `portfolio_id`, `scenario_id`, and an `allocations` map (symbol to proposed percentage of portfolio value, 0-100, the set should sum to 100). Returns `sum_pct` and a per-rule verdict for that hypothetical allocation. Use this to sanity-check one proposed change before describing it to the user.
- **`preview_rebalance`**: pass `portfolio_id`, `scenario_id`, and `prices` covering every non-cash position (required, not optional here; a missing symbol fails with `PRICES_INCOMPLETE` and the list still needed). Returns the trades that would restore compliance at those prices, derived purely from the user's own declared bands. This is a preview only. Report the plan; the user (or their broker, outside Allocait) is who acts on it, never you and never Allocait.

Neither tool writes anything. Run one of these, show the user what it found, and only then consider a write tool from `allocait-portfolio-upkeep` (to record a trade the user made) or `allocait-policy` (to change the rules themselves) if they ask for one.

## Error codes to branch on

Every tool on this surface returns errors as `Error [CODE]: message`. The codes that come up here:

- `NOT_FOUND` — an id you do not own (wrong `portfolio_id`/`scenario_id`, or a typo).
- `PRICES_INCOMPLETE` — the message ends with the exact symbols still needed; fetch those and call again.
- `PRICES_INVALID` — a price you passed was not a positive number.
- `RPC_ERROR` — a database-side failure; surface the message to the user, do not retry blindly.

## What this skill never does

- Never calls a write tool (`update_positions`, `record_cash_movement`, `apply_position_rules`, `apply_policy`, ...). If the user wants to act on a rebalance preview or a validation result, hand off to `allocait-portfolio-upkeep` or `allocait-policy` and confirm with them first.
- Never treats a monitor reading or a compliance verdict as investment advice. State the numbers and the rule; do not add an opinion about what the user should buy or sell.
