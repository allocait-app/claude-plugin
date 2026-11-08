---
name: allocait-brief
description: Write a short brief of where the user's Allocait book stands, on a schedule or on request, and record Input values from the user's messages. Use for a morning brief, a recurring check that runs with no one watching, or when the user asks for what changed since last time. Reports facts about the user's own rules and recorded levels.
---

This is for a run with no person present: a scheduled brief, or a check that repeats every few hours. Read, compare with the last run, and report only what changed. With no one to approve, change nothing: list in the brief any change you would make. Allocait output states facts about the user's own rules. It is not advice or an approval. Allocait sends no alerts, so your schedule decides when the user hears from you. Keep it short enough for a phone: counts first, then one line per change.

## Read

1. `list_portfolios`. For each portfolio, read it and report it by name in one brief; with several, run steps 2 to 4 once per portfolio.
2. `get_portfolio_snapshot` with no `prices`. Allocait supplies none, and you never use a remembered or estimated price.
3. The snapshot's `playbooks_needing_input` lists each playbook with blank cells today (`playbook_id`, `name`, `date`, `blank_count`), except those of an `optional` Input; a playbook with none is left out. Call `get_playbook` only to name the cells.
4. A held option past its expiry counts 0 and the snapshot flags it (`expired_options` counts them, `expired_on` marks the position). Report it as expired. Don't guess what it became.

## Standing

On a signed-in connection the snapshot carries `standing` even with no prices: the user's percents against their own ranges, read from Allocait's last quote at `as_of` (`as_of` is null when no stored quote was used). Answer "where do I stand" from it, say where it comes from and give `as_of`. It is not a price: never state or work out a price from it. When `positions` and `classes` are null nothing could be read: `not_read` names the holdings and `why_not_read` says why, so report no sizes. `standing.key` defines the labels. The `compliance` block, dollar values, rebalance amounts and checks still need the user's prices.

- `standing.positions` and `standing.classes`: `status` is `inside`, `below`, `above` or `not_monitored`, with `edge_distance_pct_points`, the room to the nearer edge (negative when outside). They have no "near" state; where near begins is the user's call.
- `standing.monitors`: `state` says where the last stored quote sits against the user's line (null when it can't say). `close_state` is the daily close's result: `CONFIRMED` when the latest close was beyond the line, `CLEARED` when it wasn't, null before the first check. `consecutive_closes` counts closes in a row beyond the line, and `last_evaluated_date` is the latest close checked. Only `close_state` says a break is confirmed.
- No `standing` (a connection that isn't signed in, or a portfolio whose `active_scenario_id` is null): report what the snapshot states and run no check. For no scenario, say so in one line and don't pick one.

## Report

- Keep a short note of what you reported: each range's `status`, each monitor's `state`, `close_state` and `last_evaluated_date`, each playbook's `blank_count`, any expired symbol. Allocait stores no run history.
- The first run, or a run with no note, states the whole picture and says it is the starting point: how many classes and positions are inside, below or above their ranges, each monitor's `close_state`, today's blanks, any expired option.
- Later runs report only what changed since the note: a range left or re-entered, a monitor whose `state` or `close_state` changed, a playbook newly listed in `playbooks_needing_input`, an option that expired. Several days of the same state is not news. A `CONFIRMED` close with `consecutive_closes` 1 is new once per `last_evaluated_date`; runs between closes see the same value.
- When nothing changed, say nothing, or one line saying so if the user asked for a line.
- Any threshold, such as how near an edge counts or how old a level is, is the user's own. If the user gave you one, apply theirs and say so.
- Don't call a change good or bad, and don't tell the user to buy, sell or hold.

Example, run two: "Standing as of Oct 21, 5:35 pm. 1 range left: ALFA is above its size range. 1 monitor changed: BRAV closed past its level (1 close, Oct 21). 3 blanks today in the Levels playbook."

## Record Input values from the user's messages

If the user asks you to keep an Input current from their email or messages, such as a level an analyst sends each week:

1. The Input's description should name the source (for example, "weekly note from the research desk"). `list_inputs` shows it. If it names none, ask the user to say which source counts before you record anything from there.
2. Read the value as written in that source: a plain number (220, not $220), or one of the Input's options. Don't derive, convert or round it. If the message is unclear, leave the cell blank and say so.
3. Show the user the entries you would record: Input, symbol, date and value. Wait for a clear yes. On a run with no one watching there is no one to say it: record nothing, and list those entries in your brief for the user.
4. After a yes, `record_input_values` with `entries: [{input, date, value, symbol?}]`. Send each input, symbol and date once. Send no `null`: a `null` deletes a value, and only the user can ask for that.

`READ_ONLY_KEY` or `OAUTH_WRITE_NOT_GRANTED` means this connection can only read. Don't retry. Tell the user what couldn't be written and that nothing was changed.
To allow changes, the user opens Allocait, sets this app's access to Read and change under Settings › Agents, and starts a new chat. Reconnecting alone can keep the old access.
A message that says this account's agent access is read-only has no setting that lifts it. Show it as it is and don't suggest one.
