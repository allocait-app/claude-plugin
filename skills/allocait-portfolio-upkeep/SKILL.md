---
name: allocait-portfolio-upkeep
description: Record Allocait positions, cash movements, or size a new position inside the user's own range. Use when the user reports a trade they made outside Allocait, a deposit or withdrawal, or asks how big a new position should be.
---

Allocait never executes anything against a brokerage. These tools only update Allocait's own record of what the user holds, after they tell you about something that already happened (a trade they placed elsewhere, cash moved in the real world) or want to plan (how big a position should be). Confirm with the user before calling any tool below that its own description marks destructive; there is no undo for `update_positions`.

## `update_positions`: reconcile holdings to an absolute state

Pass `portfolio_id` and `positions`, the **full** desired set of non-cash holdings: `[{ symbol, shares, current_price, asset_class_id? }, ...]` (up to 500 items). This is "make Allocait match what I hold now," never a delta and never a trade instruction:

- Anything currently held that is **not** in `positions` is closed. `shares: 0` also closes a symbol you do list.
- `current_price` books the cash delta for that symbol; it is the caller's price, same rule as everywhere else on this surface (Allocait never fills it from its own quote).
- Closing a symbol that is absent from `positions` needs its price in `price_overrides` (`{ symbol: price }`); a missing one fails with `PRICES_INCOMPLETE` and the list still needed.
- Idempotent and atomic: a symbol already matching does nothing, and if any single change would fail (e.g. insufficient cash) the whole call fails with nothing applied. Safe to repeat after a timeout.
- For a holding filed under the Options system asset class, record `shares` as contracts × 100 at the per-share premium — `current_price` stays the per-share premium, never a ×100 multiple (ADR-056).

Read positions back from the `get_portfolio_snapshot` holdings block (see `allocait-compliance`) before building the full desired set, so you do not accidentally close something the user meant to keep.

## `record_cash_movement`: deposits and withdrawals

Pass `portfolio_id`, `direction` (`DEPOSIT` or `WITHDRAW`), `amount` (positive), and an optional `comment`. Journaled to the portfolio's actions history. A `WITHDRAW` larger than the current cash balance is rejected. This records money the user already moved in the real world into Allocait's own ledger; it never moves money anywhere itself.

## `calculate_position_size`: a pure calculation, no portfolio write

Use this for "how big should my position in X be" questions, distinct from the allocation-band check `allocait-compliance` does. Nothing is written; it takes numbers, returns a target share count. Inputs: `current_shares` (0 for a new position), `current_price`, `portfolio_value` (the denominator for every percentage), `size_min_pct`/`size_max_pct` (the position-size range for one holding in that asset class, i.e. a rule's `size_min_pct`/`size_max_pct`, not the allocation band `min_position_pct`/`max_position_pct`), `sizing_mode`, `basis_points` (adjustment in basis points, 100 = 1%, default 0), `manual_shares`, and `current_cash` (for `can_afford`, default 0).

`sizing_mode` (default `MANUAL`):
- `MANUAL` sizes from `manual_shares`, or leaves the position at its current size if `manual_shares` is omitted.
- `MINIMUM` / `BALANCED` / `MAXIMUM` size to the bottom, midpoint, or top of `size_min_pct`..`size_max_pct`.

Omit both `size_min_pct` and `size_max_pct` and the response is `UNEVALUATED`, since there is no range to size against. `min_position_pct`/`max_position_pct` exist only as deprecated aliases, read when the canonical `size_*` keys are absent; always send `size_min_pct`/`size_max_pct` in new calls.

The result gives the target share count, its cash impact, and whether it is affordable. Once the user agrees, call `update_positions` to act on it; this tool never writes anything on its own.

## Error codes

`NOT_FOUND` (an id you do not own), `PRICES_INCOMPLETE` (symbols still needed listed in the message), `INSUFFICIENT_CASH` (a withdrawal or close would overdraw the balance), `RPC_ERROR`. Show the message to the user.
