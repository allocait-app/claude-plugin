---
name: allocait-portfolio-upkeep
description: Record in Allocait the trades and cash movements the user already made elsewhere, or size a position inside the user's own range. Use when the user reports a trade (buy, sale or short sale), an option's premium or expiry, an inverse or leveraged fund, a deposit or withdrawal, or holdings they already own, or asks what size would fit their range.
---

Allocait never executes anything against a brokerage. These tools only update Allocait's own record of what the user holds, after they tell you about something that already happened (a trade they placed elsewhere, cash moved in the real world), or help them size a position inside the range they set.

This records the change in Allocait only. It moves no money and places no order. Never say a trade was placed, executed or confirmed, and never say money moved.

## `save_position`: one holding as it stands now

Use this to record a single trade the user already made. Pass `portfolio_id`, `symbol`, and `shares`: the holding's **new total**, never the change. "I added 11 more ALFA" with 41 already held is `shares: 52`, never `shares: 11` and never "+11".

- `price` is the price the user says they paid or received. It's required when shares change, and it's the user's price (Allocait never fills one).
- `shares: 0` closes the holding. Closing a short covers it and takes cash, at `price`.
- A negative `shares` is a short. See Shorts below.
- Cash moves by the change in value. Other holdings stay as they are.
- Optional: `asset_class` (a name, alias or id) for a new holding, `moves` (see Moves below), `mark` (an option's premium, see Options below), `sizing` (`MANUAL`, `MINIMUM`, `BALANCED`, `MAXIMUM`), and `comment` (why, recorded on the action).
- A new holding without `asset_class` is left unclassified and listed in `unclassified`; ask the user which class it belongs to.
- A repeat of the same call changes nothing, so it's safe to retry after a timeout.

Read the current holding first (`holdings` in `list_portfolios`) so you can compute the new total. If you don't know the current shares, ask the user rather than guessing.

## Shorts

A short is a negative total, in the same field. "I sold short 11 more ALFA" with -41 held is `shares: -52`; opening a short from nothing is `shares: -11`; a sale of 153 with 101 held is `shares: -52`, one change at one `price`. `shares: 0` closes a short and debits cash, because closing it is the cover.

- A negative `shares` works only on a portfolio with Allow shorts on, and never for a holding filed under Options. With it off the call is refused with `INVALID_INPUT` and nothing is written (the message says "Allow shorts"; the `save_portfolio` argument is `allow_shorts`).
- **Allow shorts is the user's choice.** The user turns it on in the portfolio's edit dialog in the app, or you turn it on with `save_portfolio { portfolio_id, allow_shorts: true }`. Ask the user before you turn it on, and never turn it on only to get a refused write through.
- Turning it off while a short is held is refused with `IN_USE`, naming how many to close first. Close them, then turn it off.
- Short proceeds credit Cash, so no deposit is recorded for them. A cover that cash can't pay is `INSUFFICIENT_CASH`.
- In `check_allocations`, a negative entry is a short. That tool doesn't check the switch: recording one still needs Allow shorts on, and never an Options holding.
- A short is judged against its class's short size range (`short_size_min_pct`/`short_size_max_pct`), by its size without the minus sign, and is not monitored in a class that has none. Allocait records it; it computes no borrow, margin or profit.

## Moves: inverse and leveraged funds

`moves` is how many times its index a fund moves: `1`, `2`, `3`, `-1`, `-2` or `-3`. Negative is inverse, so `-3` is an inverse 3x fund. Every holding has one, and it is 1 unless the user set it. Allocait's exposure reading multiplies the holding's value by it.

- Set it only when the user says the fund is inverse or leveraged, and use the factor they give. Never work it out from a symbol or a fund's name. Allocait doesn't guess it, and neither do you. If the user doesn't say, leave `moves` out.
- Leave `moves` out to keep what is stored. A repeat call that leaves it out changes nothing, and positions read back carry `moves` on every row, so sending a list back as you read it is safe.
- Changing only `moves` is not a trade: the share count stays, no `price` is needed, no cash moves and no action is recorded. "ALFA is a 3x inverse fund" with 101 held is `{ symbol: "ALFA", shares: 101, moves: -3 }`.
- It doesn't apply to cash or to a holding filed under Options: a `moves` other than 1 there is refused with `INVALID_INPUT`. A fund with a set Moves that is moved into Options needs `moves: 1` in the same call.

## Options: the mark and expiry

An option is a holding filed under the Options asset class, with the option symbol (root, expiry as YYMMDD, `C` or `P`, then the strike in eight digits, for example `ECHO281215P00050000`). Three contracts at a $2.10 premium is `shares: 300` with `price: 2.10`.

- An option counts at its **mark**, the premium per share the user gave. Allocait looks up no option price, and neither do you. Use only a premium the user gives you. Never work one out or guess it.
- Leave `mark` out and a trade price becomes the mark. Send `mark` to set it, or to change only it: `{ symbol: "ECHO281215P00050000", shares: 300, mark: 1.05 }` with 300 held is not a trade, so no `price` is needed, no cash moves and no action is recorded. A repeat write of the same mark keeps its date.
- `mark` works only on a holding in Options. On any other holding it is refused with `INVALID_INPUT`. Leave it out to keep what is stored.
- An option with a mark needs no entry in `prices`. An option with no mark is an unpriced holding that the snapshot lists in `symbols_to_price`; ask the user for its premium.
- Allocait reads the expiry from the symbol. From the day after the expiry date (US Eastern), the snapshot shows `expired_on` on the position, `expired_options` counts it, and the option counts 0 whatever price is passed. Allocait changes nothing else and doesn't know what happened: the option may have expired worthless, been sold, or been exercised. **Ask the user what happened, and never guess.** Close it only after they say. For one that expired worthless, close it with `{ symbol, shares: 0, price: 0 }`: no cash moves and one action is recorded. `price: 0` is part of the row, because `{ symbol, shares: 0 }` is refused with "A price is needed for X, where shares change." A sale before expiry, or an exercise, is recorded as the trade it was.

## `replace_positions`: the whole holdings list

Use this to record many holdings at once, for example from a statement the user gives you. Pass `portfolio_id` and `positions`: every non-cash holding as the list now reads, each `{ symbol, shares, price?, asset_class?, moves?, mark?, sizing?, comment? }`. A short is a negative `shares` (see Shorts). A row that leaves `moves` or `mark` out keeps the stored value.

- A holding you leave out is **refused, not closed**: the call fails with `WOULD_CLOSE_POSITIONS` naming it. Include it, or close it with an explicit `{ symbol, shares: 0, price }` row, only if the user no longer holds it (`price: 0` for an expired option, after the user says what happened).
- `price` is needed on rows whose shares change; it can be left out on unchanged rows.
- The call is all or nothing: if one change fails (for example, insufficient cash), nothing is applied.

## `record_cash_movement`: deposits and withdrawals

Pass `direction` (`DEPOSIT` or `WITHDRAW`), a positive `amount` and an `idempotency_key`: one unique string per movement, reused on a retry so it can't be recorded twice. A `WITHDRAW` larger than the cash balance is refused.

## Holdings the user already owned

Allocait settles every buy against recorded cash. To record holdings the user owned before they started using Allocait, first deposit their value plus any cash not already in the balance with `record_cash_movement`, then record the holdings with `replace_positions`.

## `calculate_position_size`: a calculation, no write

Use this for "what size would fit my range". It writes nothing: it takes the numbers you pass and returns a target share count and its cash impact, and whether it is affordable when you pass `cash`. A `MINIMUM`, `BALANCED` or `MAXIMUM` call without its bound is refused. Name the user's size range when you report it. `size_min_pct`/`size_max_pct` are the size range for one holding in that class (the short size range for a short), from the rule in `get_rules`, not the class band. `portfolio_value` is the base for every percentage. `current_shares` is 0 for a new position and negative for a short the user holds; `manual_shares` is always a positive count, a short's too. `basis_points` (100 = 1%, either sign) is added to the target of every mode, so a positive value makes a short larger; it is ignored when `manual_shares` is given.

Every price is the user's. Allocait supplies none and fills none. For a price the user hasn't given, ask the user or use a source they trust. Never use a remembered or estimated price: a wrong price gives a wrong result.

`sizing_mode` (default `MANUAL`):
- `MANUAL` sizes from `manual_shares`. With no `manual_shares` it starts from the current size, so `basis_points` moves the holding by that much.
- `MINIMUM` / `BALANCED` / `MAXIMUM` size to the bottom, midpoint, or top of the size range, then add `basis_points`.

The target stays between 0% and 100% of the book. It is not held inside the size range: when `basis_points` takes it outside, the share count is still the one asked for and `compliance_status` is `BELOW_MIN` or `ABOVE_MAX`. Report that as the fact it is.

Sizing a short: pass `direction` `short`, the class rule's short size range as `size_min_pct` and `size_max_pct`, a negative `current_shares` for a short the user holds, and a positive `manual_shares` for MANUAL. The target share count, value and percent come back negative, and the verdict compares the absolute size with the range. Say the result is a calculation that writes nothing. With no size range, the result is `UNEVALUATED`. Once the user has made the trade and tells you, record it with `save_position`.

## Actions

`list_actions` returns the portfolio's recorded trades and cash movements, newest first. `save_action_comment` records why a decision was made (`null` clears it).

## When the connection can only read

`READ_ONLY_KEY` or `OAUTH_WRITE_NOT_GRANTED` means this connection can only read. Don't retry. Tell the user what couldn't be written and that nothing was changed.
To allow changes, the user opens Allocait, sets this app's access to Read and change under Settings › Agents, and starts a new chat. Reconnecting alone can keep the old access.
A message that says this account's agent access is read-only has no setting that lifts it. Show it as it is and don't suggest one.

## Error codes

`WOULD_CLOSE_POSITIONS` (holdings left out of `replace_positions`; include them, or close one with a `shares: 0` row only if the user no longer holds it), `INSUFFICIENT_CASH` (the message names how much more cash the trades need; record a deposit only if the user made one that isn't recorded, otherwise ask the user), `PRICES_INCOMPLETE` (a changed holding without a price), `NOT_FOUND` (an id you do not own), `INVALID_INPUT` (for example, a `CASH` row: cash moves only with `record_cash_movement`; a negative `shares` while Allow shorts is off; a negative `shares` for an Options holding; a `moves` outside 1, 2, 3, -1, -2 and -3, or other than 1 on cash or an Options holding; a `mark` on a holding outside Options; or a trade too large to record), `IN_USE` (turning Allow shorts off while a short is held), `READ_ONLY_KEY` or `OAUTH_WRITE_NOT_GRANTED` (the connection can only read; don't retry). Tell the user in plain words what went wrong and what happens next.
