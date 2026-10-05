---
name: allocait-portfolio-upkeep
description: Record Allocait holdings and cash movements, or size a new position inside the user's own range. Use when the user reports a trade they made outside Allocait, a deposit or withdrawal, holdings they already own, or asks how big a new position should be.
---

Allocait never executes anything against a brokerage. These tools only update Allocait's own record of what the user holds, after they tell you about something that already happened (a trade they placed elsewhere, cash moved in the real world), or help them plan (how big a position should be). Never say a trade was placed, executed, or confirmed.

## `save_position`: one holding as it stands now

Use this for a single trade. Pass `portfolio_id`, `symbol`, and `shares`: the holding's **new total**, never the change. "I bought 10 more MSFT" with 40 already held is `shares: 50`, never `shares: 10` and never "+10".

- `price` is the price paid or received. It's required when shares change, and it's the user's price (Allocait never fills one).
- `shares: 0` closes the holding.
- Cash moves by the change in value. Other holdings stay as they are.
- Optional: `asset_class` (a name, alias or id) for a new holding, `sizing` (`MANUAL`, `MINIMUM`, `BALANCED`, `MAXIMUM`), and `comment` (why, recorded on the action).
- A new holding without `asset_class` is left unclassified and listed in `unclassified`; ask the user which class it belongs to.
- A repeat of the same call changes nothing, so it's safe to retry after a timeout.
- For a holding filed under the Options asset class, `shares` is contracts × 100 and `price` is the per-share premium, never a ×100 multiple.

Read the current holding first (from `get_portfolio_snapshot`) so you can compute the new total. If you don't know the current shares, ask the user rather than guessing.

## `update_positions`: the whole holdings list

Use this to reconcile many holdings at once, for example from a statement. Pass `portfolio_id` and `positions`: every non-cash holding as it should stand, each `{ symbol, shares, price?, asset_class?, sizing?, comment? }`.

- A holding you leave out is **refused, not closed**: the call fails with `WOULD_CLOSE_POSITIONS` naming it. Include it, or close it with an explicit `{ symbol, shares: 0, price }` row.
- `price` is needed on rows whose shares change; it can be left out on unchanged rows.
- The call is all or nothing: if one change fails (for example, insufficient cash), nothing is applied.

## `record_cash_movement`: deposits and withdrawals

Pass `portfolio_id`, `direction` (`DEPOSIT` or `WITHDRAW`), `amount` (positive), an optional `comment`, and an `idempotency_key` (any unique string for this one movement, reused on a retry) so a retry can't record it twice. A `WITHDRAW` larger than the cash balance is refused. This records money the user already moved; it never moves money anywhere.

## Holdings the user already owned

Allocait settles every buy against recorded cash. To record holdings the user owned before they started using Allocait, first deposit their value plus the cash balance with `record_cash_movement`, then record the holdings with `update_positions`.

## `calculate_position_size`: a calculation, no write

Use this for "how big should my position in X be". Nothing is written; it takes numbers and returns a target share count. Inputs: `current_shares` (0 for a new position), `price`, `portfolio_value` (the denominator for every percentage), `size_min_pct`/`size_max_pct` (the size range for one holding in that class, from the rule; not the class band), `sizing_mode`, `basis_points` (adjustment, 100 = 1%, default 0), `manual_shares`, and `cash` (for `can_afford`, default 0).

`sizing_mode` (default `MANUAL`):
- `MANUAL` sizes from `manual_shares`, or keeps the current size if `manual_shares` is left out.
- `MINIMUM` / `BALANCED` / `MAXIMUM` size to the bottom, midpoint, or top of the size range.

With no size range, the result is `UNEVALUATED`. The result gives the target share count, its cash impact, and whether it's affordable. Once the user has made the trade and tells you, record it with `save_position`.

## Actions

`list_actions` returns the portfolio's recorded trades and cash movements, newest first. `save_action_comment` records why a decision was made (`null` clears it).

## Error codes

`WOULD_CLOSE_POSITIONS` (holdings left out of `update_positions`; include them or close them with `shares: 0` rows), `INSUFFICIENT_CASH` (the message names the deposit needed), `PRICES_INCOMPLETE` (a changed holding without a price), `NOT_FOUND` (an id you do not own), `INVALID_INPUT` (for example, a `CASH` row: cash moves only with `record_cash_movement`), `READ_ONLY_KEY` or `OAUTH_WRITE_NOT_GRANTED` (the connection can only read; don't retry). Show the message to the user.
