---
name: allocait-compliance
description: Check an Allocait portfolio against its own allocation rules, or work out the amounts that close the gap to them. Use when the user asks how far their portfolio is from their rules, wants a compliance check, or wants a rebalance preview.
---

Allocait checks a portfolio against the user's own rules. It places no trade, connects to no brokerage and recommends nothing. Every verdict here is the user's own band or range, applied to prices the user supplies.

For a run with no one present, follow `allocait-brief` instead: it asks for no prices.

## Prices

Every price is the user's. Allocait supplies none and fills none. For a price the user hasn't given, ask the user or use a source they trust. Never use a remembered or estimated price: a wrong price gives a wrong result.

On a signed-in connection the snapshot carries `standing` even with no prices: the user's percents against their own ranges, read from Allocait's last quote at `as_of` (`as_of` is null when no stored quote was used). Answer "where do I stand" from it, say where it comes from and give `as_of`. It is not a price: never state or work out a price from it. When `positions` and `classes` are null nothing could be read: `not_read` names the holdings and `why_not_read` says why, so report no sizes. `standing.key` defines the labels. The `compliance` block, dollar values, rebalance amounts and checks still need the user's prices.

## Start here: get the state of the world

1. Call `list_portfolios` (no arguments) to find the `portfolio_id` you need. It returns each portfolio's name, currency and active scenario, its playbooks, monitors and charts, and every scenario. No dollar totals come back here.
2. Call `get_portfolio_snapshot` with that `portfolio_id`. This is the one call to reason from, not a set of separate reads:
   - Without `prices`, the snapshot is unpriced: shares, rules, monitor settings, playbooks and charts come back, but values, `exposure`, `compliance` and every monitor reading are `null`. `holdings_to_price` lists the prices that values and the verdict need; `symbols_to_price` adds symbols only a monitor reads. A held option with a mark is the one exception: it is priced at the user's own mark and isn't listed for its own price (a monitor that reads positions can still list it; see below).
   - Call it again with a `prices` object (`{ "ALFA": 181.5, ... }`, a fictional example, symbol to price in the portfolio's own currency) covering `holdings_to_price` (add `symbols_to_price` only for monitor readings). The same call then returns values, the compliance verdict and monitor readings, all computed from your numbers only. It has no `rebalance` block unless you add `include_rebalance: true`, which you do only when the user asks what it would take to get back inside a band.
   - `scenario_id` is optional; it defaults to the portfolio's active scenario. If `active_scenario_id` is null there is no scenario to judge by: use the one the user names, or ask which, and never pick one yourself. Setting the active scenario is `save_portfolio`, and only when the user asks.

The first key, `summary`, gives the counts in one line; lead with it. Answer from the snapshot's `compliance` block:

- `status`: `compliant`, `partial` (no breach, but some holdings are in `unchecked`), `violations` or `unjudged` (for example, a portfolio worth $0). `issue_count` counts the breaches.
- `class_violations`: each asset class whose total, Cash included, is outside its allocation band (`band_min_pct`/`band_max_pct`, as % of portfolio value). A class total is the net of its holdings, so a short subtracts, and a total below zero counts as 0. A class with a band and no holdings counts as 0%, so it is listed whenever its minimum is above 0.
- `size_violations`: each holding outside its class's range for the side it is on, judged by its size without a sign. A long is judged against the position size range (`size_min_pct`/`size_max_pct`, for **one holding** in that class), a short (a negative `shares`) against the short size range (`short_size_min_pct`/`short_size_max_pct`, for **one short** in that class). Each row carries `side` (`long` or `short`); a short's `current_pct` is positive and its `current_value` is negative, and the row's `size_min_pct`/`size_max_pct` are the range used. A class with no range for that side sets no limit, so a short in a class with no short size range is not monitored and is never listed here.
- `unchecked`: holdings no rule could judge, each with its reason (for example, a holding with no asset class). Mention them; don't treat them as compliant.

Example answer: "At the prices you gave, two things sit outside your rules. Equities is 3.1 points above its 63% maximum, and ALFA is 0.9 points over its 7.5% size maximum. Every other rule is inside its range."

The snapshot's `exposure` block says how much of the portfolio gains if prices rise: `long_pct`, `short_pct`, `net_pct` and `gross_pct` as percents of portfolio value, by asset class (`classes`) and per holding (`holdings`, each with its `moves`). It is a reading, not a verdict: no rule judges it. It is `null` while any holding is unpriced, even where Allocait could have priced it, so ask for the missing prices; don't work the figures out by hand. Options count at their premium (a call long, a put short, 0 once expired), so when `options_counted` is above 0 say the figures are at premium: a premium understates how far an option moves. `options_not_counted_symbols` names priced options left out because their symbol is not an option symbol.

**Options.** An option counts at its `mark`, the premium per share the user recorded (`mark_on` is the day it was set; report it as a date and don't call a mark old). A held option with a mark needs no entry in `prices`; a price you pass for it replaces the mark for that call only. An option with no mark is unpriced and listed in `symbols_to_price`. A monitor that reads positions can list a marked option too, with that monitor's reading `unpriced`. From the day after its expiry an option has `expired_on`, counts 0, has no size verdict and never makes a read incomplete (`expired_options` counts them). Tell the user it expired and that what it became is theirs to say. Don't close it or guess whether it expired worthless, was sold or was exercised; recording it is `allocait-portfolio-upkeep`.

Each breach's `status` is `ABOVE_MAX` or `BELOW_MIN`, and `outside_by` gives `edge`, `pct_points` (percentage points of portfolio value outside the band or range) and `value` (dollars outside, at your prices). Report it as where the rule stands. It names no trade; amounts come only from `rebalance`, when the user asks.

Verdicts are computed only once `prices` cover every non-cash holding (except a marked or expired option), so each class verdict rests on all of that class's holdings.

## Work out the amounts that close a gap

When the user asks what it would take to get back inside a band, call the snapshot with `prices` and `include_rebalance: true`. Its `rebalance` block holds the arithmetic:

- `trades`: the pro rata amounts that bring each class to its target (the rule's target, or the band's midpoint), largest first, each with `class_target_pct`. Say each row's `direction` (`ADD` or `REDUCE`) and its amount. `action` (`BUY` or `SELL`) is the wire word; don't repeat it as an instruction. A short or an option gets no row. A class's gap is spread over its long holdings outside Options only, so one long can carry the whole gap. A class with no priced long holding there has no row and shows in `unresolved` if it stays outside its band.
- `resulting_classes`: each class's share after those trades, Cash included. Sale proceeds stay in Cash.
- `unresolved`: any class still outside its band after the trades, such as a class with nothing to buy into. For a class that holds only shorts and options, `no_priced_holdings` means it has nothing to trade, not that a price is missing. Trades don't resize holdings, so report the size verdicts separately.
- `not_rebalanced`: every short and option held, once each, as `{symbol, asset_class, reason}`, with `reason` `short` or `option`. They are left as they are; do not suggest a trade for them.

This is arithmetic on the user's own bands, not an order and not a suggestion to buy, sell or hold. Report the amounts as amounts. What happens next is the user's decision, made outside Allocait.

## To see what a change does, check it first

To check a specific change, such as 40 more shares of a symbol, use `allocait-check` (`check_trades`). Two more read-only tools take a proposed state rather than acting on it:

- **`check_allocations`**: pass `portfolio_id` and an `allocations` map (symbol to proposed percentage of portfolio value; an entry can be negative, which is a short, or above 100; `is_valid` needs them to sum to 100). `scenario_id` is optional and defaults to the active scenario. Returns `sum_pct`, class band breaches (`violations`), and holdings outside their size range (`size_violations`, each with `side`; a negative entry is judged by its size without the minus sign against the class's short size range) for that proposal; `is_valid` is true only when the sum is 100, both lists are empty and `unknown_symbols` is empty. Propose 0 for an exit; a class the proposal leaves empty counts as 0% against its band, the same verdict `compliance` gives once the exit is recorded. For a symbol the user doesn't hold yet, pass `asset_classes` (symbol to class name or id) so it is judged like a holding.
- **`calculate_position_size`**: the share count that sits inside the size range the user set for one holding. See `allocait-portfolio-upkeep`.

Neither tool writes anything. Run one, show the user what it found, and only then consider a write from `allocait-portfolio-upkeep` (to record a trade the user made) or `allocait-rules` (to change the rules) if they ask for one.

## Monitors

The snapshot's `monitors` block (priced calls only) says where a price sits against a line the user recorded. It is a measurement, not a verdict. Only the once-daily close check confirms a break (`close_state` `CONFIRMED`, in `standing.monitors`), so don't call a break settled from an intraday reading.

## Errors

- `PRICES_INCOMPLETE`: the message names every symbol still needing a price (also in `missing`). Ask for those, then call again.
- `PRICES_INVALID`: a price was negative or not a number. 0 is valid, for example an option that expired worthless.
- `NOT_FOUND`: an id you do not own.
- `INTERNAL_ERROR`: retry once, then tell the user and give them its `request_id`. File a report with `send_feedback` only if the user asks.

## What this skill never does

- Never calls a write tool. If the user wants to act on the rebalance amounts or a validation result, hand off to `allocait-portfolio-upkeep` or `allocait-rules` and confirm with them first.
- Never treats a monitor reading or a compliance verdict as investment advice. State the numbers and the rule; do not add an opinion about buying, selling or holding.
