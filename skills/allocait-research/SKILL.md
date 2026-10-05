---
name: allocait-research
description: Keep an Allocait user's research current - watchlists, inputs (tracked levels and values), playbooks, monitors and charts. Use when the user wants to add or remove a watchlist symbol, fill in a playbook, record a level or reading, or set up an input, playbook, monitor or chart.
---

Allocait keeps the user's own research next to their rules. Every value in it is something the user recorded: Allocait never derives a level, a range or a reading from market data, and neither should you. Record what the user tells you; if a value is missing, ask for it.

## How every save works

- `save_<thing>` creates when you leave out the id and changes the record when you pass it.
- A field you send replaces the stored value, `null` clears it, and a field you leave out keeps its value.
- A list you send (`items`, `options`, `inputs`, `symbols`, `conditions`) is the whole list, in order. If it would drop stored members, the call is refused with `CONFIRMATION_REQUIRED` and `would_remove`. Confirm with the user, then send the call again with exactly those members in `remove`.
- Creating with a name that's taken returns `ALREADY_EXISTS` with the existing id. Change that record instead of creating another.
- Reads return each record with the field names its save takes, so read, edit and save.

## Watchlists

1. `list_watchlists` returns each watchlist's id, name and symbols in order.
2. To add or remove one symbol, call `save_watchlist` with `watchlist_id` and `add: [{symbol, asset_class?}]` or `remove: [symbol]`. Every other symbol stays. Adding a symbol that's already there, or removing one that isn't, changes nothing. An added symbol goes at the end.
3. To reorder or replace the list, send `items` in the new order, every symbol you want to keep included.
4. To create one, call `save_watchlist` with `name` and optional `items`. A watchlist holds up to 200 symbols, and an account up to 50 watchlists.

"Add NVDA to my Tech watchlist" is `list_watchlists`, then `save_watchlist {watchlist_id, add: [{symbol: "NVDA"}]}`. Never resend the list without a symbol the user didn't ask to remove.

## Inputs: what the user tracks

An input is a value the user tracks: a level, a reading, a rating or a note. `list_inputs` returns each one's name, `data_type` (`NUMBER`, `DECIMAL`, `PERCENTAGE`, `TEXT`, `CHOICE`), `scope` (`PORTFOLIO`, or `SYMBOL` for one value per symbol) and, for CHOICE, its `options`. Name one input to also get its recorded values.

`save_input` creates or changes one. For a CHOICE input, `options` is the whole list in order. The data type and scope can't change once values are recorded. `delete_input` also removes the input from the playbooks and charts that use it and names them; a monitor that uses it blocks the delete with `IN_USE`.

## Fill in a playbook

A playbook is a grid: symbols down the side, inputs across the top.

1. `list_portfolios` names each portfolio's playbooks as `[{id, name, date}]`, newest first. A playbook is one dated sheet: for a new day the app copies the previous one under a new name, with that day's date.
2. `get_playbook` with `playbook_id` and the `date` (for "today", pass today's date) returns its symbols, its columns with their `data_type`, `scope` and `options`, and the values already recorded.
3. Ask the user for the missing values. Don't fill a value they didn't give you.
4. `record_input_values` with `entries: [{input, date, value, symbol?}]`, by input name. Pass `symbol` only for SYMBOL-scope inputs. A CHOICE value must be one of the input's options; a number is a plain number (220, not $220). A value for the same input, symbol and date replaces the old one, and `value: null` deletes it.

`save_playbook` sets which symbols a playbook covers (`source`: `positions`, `watchlist` with `watchlist_id`, or `custom` with `symbols`) and which inputs are its columns (`inputs`, in order).

## Monitors and charts

`list_portfolios` names each portfolio's monitors and charts; `get_portfolio_snapshot` returns their full settings.

- `save_monitor` compares prices with a level the user recorded (`mode`: `ABOVE`, `BELOW`, `NEAR`, with `input`) or a range (`RANGE`, with `input` as the low end and `high_input` as the high end), for the symbols its `source` names. `threshold_pct` is 0-100.
- `save_chart` plots `inputs` and `level_markers` (input names) for its symbols over `days` (7, 14, 30, 60 or 90).

A monitor reading is a measurement against the user's own line, never a signal. Don't describe it as a reason to buy or sell.

## Error codes

`CONFIRMATION_REQUIRED` (confirm, then resend with `remove`), `ALREADY_EXISTS` (use the existing id), `NOT_FOUND` (an id or name that isn't the user's; for names, `valid_values` lists the choices), `INVALID_INPUT` (for example, a CHOICE value that isn't an option; the message lists them), `IN_USE`, `TIER_LIMIT_EXCEEDED` (a watchlist, symbol or monitor limit; don't retry), `FEATURE_NOT_AVAILABLE` (the plan has lapsed), `READ_ONLY_KEY` or `OAUTH_WRITE_NOT_GRANTED` (the connection can only read). Show the message to the user.
