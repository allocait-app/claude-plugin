---
name: allocait-research
description: Keep an Allocait user's research current - watchlists, inputs (tracked levels and values), playbooks, monitors and charts. Use when the user wants to add or remove a watchlist symbol, fill in a playbook, record a level or reading, or set up an input, playbook, monitor or chart.
---

Allocait keeps the user's own research next to their rules. Every value in it is something the user recorded: Allocait never derives a level, a range or a reading from market data, and you don't either. Record what the user tells you; if a value is missing, ask for it.

## How every save works

- `save_<thing>` creates when you leave out the id and changes the record when you pass it.
- A field you send replaces the stored value, `null` clears it, and a field you leave out keeps its value.
- A list you send (`items`, `options`, `inputs`, `symbols`, `conditions`) is the whole list, in order. If it would drop stored members, the call is refused with `CONFIRMATION_REQUIRED` and `would_remove`. Confirm with the user, then send the call again with exactly those members in `remove`. `remove` alone, without the list, drops just those members; the result's `removed` names what went, and a name that matched nothing isn't in it.
- Creating with a name that's taken returns `ALREADY_EXISTS` with the existing id. Change that record instead of creating another.
- Reads return each record with the field names its save takes, so read, edit and save.
- A delete (`delete_input`, `delete_monitor`, `delete_chart`, `delete_playbook`, `delete_watchlist`) can't be undone. Name what will be removed and wait for the user's yes before you call one.

## Watchlists

1. `list_watchlists` returns each watchlist's id, name and symbols in order.
2. To add or remove one symbol, call `save_watchlist` with `watchlist_id` and `add: [{symbol, asset_class?}]` or `remove: [symbol]`. Every other symbol stays. Adding a symbol that's already there, or removing one that isn't, changes nothing. An added symbol goes at the end.
3. To reorder or replace the list, send `items` in the new order, every symbol you want to keep included.
4. To create one, call `save_watchlist` with `name` and optional `items`. A watchlist holds up to 200 symbols, and an account up to 50 watchlists.

"Add BRAV to my Tech watchlist" is `list_watchlists`, then `save_watchlist {watchlist_id, add: [{symbol: "BRAV"}]}`. Never resend the list without a symbol the user didn't ask to remove.

## Inputs: what the user tracks

An input is a value the user tracks: a level, a reading, a rating or a note. `list_inputs` returns each one's name, `data_type` (`NUMBER`, `DECIMAL`, `PERCENTAGE`, `TEXT`, `CHOICE`), `scope` (`PORTFOLIO`, or `SYMBOL` for one value per symbol) and, for CHOICE, its `options`. Name one input to also get its recorded values.

`save_input` creates or changes one. For a CHOICE input, `options` is the whole list in order. The data type and scope can't change once values are recorded. `carry_forward: true` marks a value that holds until the user changes it, such as a level: a playbook then shows its latest value on or before the date, not only a value recorded that day. `optional: true` keeps an input's blank cells out of `needs_input`, for a value the user doesn't always have. `list_inputs` returns both flags on each input, and `emphasis` (`none`, `lower`, `higher`, `not_higher` or `not_lower`): the change against the previous entry the user asked a playbook to mark. The user sets it in the app; you can read it and can't set it. `delete_input` also removes the input from the playbooks and charts that use it and names them; a monitor that uses it blocks the delete with `IN_USE`.

## Fill in a playbook

A playbook is a grid: symbols down the side, inputs across the top.

1. `list_portfolios` names each portfolio's playbooks as `[{id, name, date}]`, newest first. One playbook serves every day: `date` is the sheet's own date, today (US Eastern) for a sheet that follows today. `save_playbook` pins a day with `default_date`, and `default_date: null` goes back to following today. Don't create a playbook for a new day.
2. `get_playbook` with `playbook_id` and the `date` (for "today", pass today's date) returns its symbols, its columns with their `data_type`, `scope`, `carry_forward`, `optional` and `options`, and the values for that date. Without `date` it reads the sheet's own date: today (US Eastern) unless a date is pinned, which can be an earlier day.
3. Each value carries `date`, the day it was recorded. A column with `carry_forward: true` shows its latest value on or before the date, and a value from an earlier day has `carried: true`. A column without it shows only what was recorded on the date. `needs_input` lists the cells with no value, except the blank cells of an `optional` input. A value also carries `previous_value` and `previous_date`, the entry recorded just before it (left out for the first), and `since`, the first date of the unbroken run of equal values that ends at it. Report them as facts, such as "was 3 on Oct 9, 5 since Oct 16", and don't call a change good or bad. A value with `emphasized: true` meets the rule the user set on its input, such as lower than the previous entry. Say that the user's rule holds, and leave out any view on whether it is good or bad. A value saved again for the same date replaces the old one, so there is no earlier version of it.
4. Ask the user for the values in `needs_input`. Don't fill a value they didn't give you, and leave a carried value alone unless the user gives a new one.
5. `record_input_values` with `entries: [{input, date, value, symbol?}]`, by input name. Pass `symbol` only for SYMBOL-scope inputs. A CHOICE value must be one of the input's options; a number is a plain number (220, not $220). A value for the same input, symbol and date replaces the old one, and `value: null` deletes it. On a `carry_forward` column, deleting that day's value shows the earlier one again. Send one entry for each input, symbol and date: a repeat in one call is refused as `INVALID_INPUT` and nothing is written.

`save_playbook` sets which symbols a playbook covers (`source`: `positions`, `watchlist` with `watchlist_id`, or `custom` with `symbols`) and which inputs are its columns (`inputs`, in order).

## Monitors and charts

`list_portfolios` names each portfolio's monitors and charts; `get_portfolio_snapshot` returns their full settings.

- `save_monitor` compares prices with a level the user recorded (`mode`: `ABOVE`, `BELOW`, `NEAR`, with `input`) or a range (`RANGE`, with `input` as the low end and `high_input` as the high end), for the symbols its `source` names. `threshold_pct` is 0-100; without it no `near_*` state is reported. `ABOVE` breaks on a close above the level and `BELOW` on a close below it. `conditions` only mark a symbol Met or Not met in the app and filter nothing.
- `save_chart` plots `inputs` and `level_markers` (input names) for its symbols over `days` (7, 14, 30, 60, 90 or 180).

A monitor reading is a measurement against the user's own line, never a signal. Each reading also names the date of the level it measured against (`recorded_low_date`, and `recorded_high_date` for a range). A level is the newest value the user recorded however old, so give the date when you report a reading, and leave it to the user to say whether it is too old. Don't describe it as a reason to buy or sell. Readings need prices, and the prices are the user's: Allocait supplies none. Ask for any price you need, and never use a remembered or estimated price.

## Error codes

`CONFIRMATION_REQUIRED` (confirm, then resend with `remove`), `ALREADY_EXISTS` (use the existing id), `NOT_FOUND` (an id or name that isn't the user's; for names, `valid_values` lists the choices), `INVALID_INPUT` (for example, a CHOICE value that isn't an option; the message lists them), `IN_USE`, `TIER_LIMIT_EXCEEDED` (a watchlist, symbol or monitor limit; don't retry), `FEATURE_NOT_AVAILABLE` (the account doesn't include this), `READ_ONLY_KEY` or `OAUTH_WRITE_NOT_GRANTED` (the connection can only read; don't retry). Tell the user in plain words what went wrong and what happens next.
