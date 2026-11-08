---
name: allocait-policy
description: Write or import a POLICY.md rule file for an Allocait scenario, or edit its allocation bands and position-size ranges (long and short) directly. Use when the user wants to set up their rules, paste an investment policy statement, or edit their allocation bands.
---

A scenario's rules are the allocation band (and optional position-size ranges, one for long holdings and one for shorts) Allocait checks each asset class against. **Allocait never picks a range for you and never recommends an allocation.** Every number that ends up in a rule is the user's own; your job is to capture it accurately, show a diff before writing, and never invent or round a figure the user did not give you.

## Read the rules first

`get_policy` (read) takes a `scenario_id`, or a `portfolio_id` for that portfolio's active scenario. It returns every asset class with its id and aliases, each rule's band (`band_min_pct`/`band_max_pct`), target (`target_pct`), size range (`size_min_pct`/`size_max_pct`) and short size range (`short_size_min_pct`/`short_size_max_pct`), the same rules as POLICY.md text, and the 5 latest restore points. With neither argument, or for a portfolio with no active scenario, rules and policy come back empty, with a note. Call it before any change, and to find a `restore_point_id` in a new chat.

## Two ways to set rules

- **`save_class_rule`**: one asset class's band, target, size range or short size range, keeping every other rule. Use this when the user changes one class ("set Equities to 43-67%").
- **POLICY.md** (`preview_policy` / `apply_policy` / `restore_policy`): all the rules for one scenario as one text file. Use this when the user has notes, a spreadsheet dump, or an investment policy statement to convert, or wants a portable file they can read and edit outside Allocait.

`apply_policy` is **absolute-state**: the file becomes all of the scenario's rules, and any class not in the file loses its rule. Neither tool renames, reorders, or deletes an asset class.

## The POLICY.md format

The short version below is enough to write a valid file, and `get_policy` returns the scenario's current file to start from. The numbers in it are a fictional example: only ever write the user's own:

```markdown
---
version: 1
name: Example rules
classes:
  Equities: { target: 53, min: 43, max: 67, position_size: { min: 3, max: 9 }, short_position_size: { min: 1, max: 5 } }
  Fixed Income: { target: 25, min: 17, max: 29 }
  Cash: { min: 3, max: 11 }
---

Free text. Allocait ignores everything below the second --- line.
```

- Frontmatter only: `version` (always 1), optional `name`, required `classes` (at least one entry).
- Each class needs a `min`, a `max`, or both; that is its band. A bound left out reads as 0 (min) or 100 (max): tell the user when they give only one. A class may instead hold only a `position_size` or `short_position_size`: it then has no band, its total is not checked, and each holding is. A `target` needs a `min` or `max`, and Allocait won't pick one for you. A `max` of 0 means none of the class.
- `target`, when given, must sit between `min` and `max`.
- A `min` above 0 on a class the user holds nothing in is a breach as soon as the rule is written: Allocait judges an empty class at 0%. If the user means "no floor until I buy in", the `min` is 0; ask rather than assume.
- `position_size: { min, max }` is optional: the size range one single holding in that class is checked against, as % of the whole portfolio. `max` is required; omit `min` for a cap with no floor (`{ max: 7 }` = no single holding over 7%, no minimum).
- `short_position_size: { min, max }` is optional and has the same shape: the size range one short holding in that class is checked against, as % of the whole portfolio, counted without the minus sign. A short in a class with no `short_position_size` is not checked. Leave the key out of a file you apply and the class's short size range is cleared, as with `position_size`.
- Known class names (use these when they fit): Cash, Equities, Fixed Income, Currencies, Crypto, Commodities, Options. A name resolves in one order everywhere: the user's own class with that name, then the system class, then an alias (Stocks/Equity/ETFs/Funds -> Equities, Bonds -> Fixed Income, Forex/FX -> Currencies), which also goes to the user's own class of that name first. A name close to a known one but not matching (e.g. "Global Equities") becomes its own custom class unless the user says to fold it into the matching one.
- An unknown key in a rule, a tab character, or content before the first `---` is a parse error, not a silent skip.

## Ask plain questions, not your own numbers

Every number in a rule is the user's own. Never suggest, round, fill in or adjust one, and never offer a typical range. If the user has no number for something, leave it out and say so.

Ask plain questions instead of choosing numbers yourself:

- No notes: ask a few plain questions, one at a time (which asset classes, and a min and max, plus an optional target, size range and short size range, for each).
- With notes (pasted text, a spreadsheet dump, an investment policy statement): read the ranges out of that text instead of asking. Never invent a number that is not in the notes.

## Preview before you write, always

`preview_policy` (read, `scenario_id` + `policy` text) parses the file and returns a diff: each class is `added`, `changed`, `removed` or `unchanged`, with its `before` and `after` rule (an unchanged class has no `after`), plus `classes_to_create` for any brand-new custom class the file implies. `written_as` on a changed entry shows the name exactly as written in the file when it differs from the resolved class name (e.g. an alias match). **Always call `preview_policy` and show the user the diff before `apply_policy`** with the identical text; `apply_policy` carries no dry-run of its own and writes on every successful call. After the diff, ask the user whether to apply it, and call `apply_policy` only after a clear yes.

Both tools take a file of at most 102,400 bytes (100 KiB) that describes at most 500 classes, and refuse anything larger. A parse error (missing frontmatter, a class with no min, max or size range, an unknown key, a duplicate class name, a target outside its own band, ...) fails with `POLICY_INVALID` and every issue found, before anything reaches the database. A warning (e.g. targets or minimums summing over 100%) never blocks the preview or the write; it comes back in `issues` alongside the diff, so surface it to the user but do not treat it as a failure.

`apply_policy` saves a **restore point** of the scenario's prior rules before writing and returns its id as `restore_point_id`. Mention `restore_policy` (takes a `restore_point_id`) as the undo path. It puts the rules back exactly as they were at that point, so changes made after it are undone too, and it saves a fresh restore point first, so a restore can itself be undone. A class the original import created gets deleted on restore only if nothing still references it (no position rule, position, or watchlist item).

## `save_class_rule`: one class, everything else kept

Takes `scenario_id` (required), `asset_class` (a name, alias or id), and the fields to change: `band_min_pct`, `band_max_pct`, `target_pct`, `size_min_pct`, `size_max_pct`, `short_size_min_pct`, `short_size_max_pct`. A field you leave out keeps its value.

- A new rule needs a band or a size range. `band_min_pct` must be below `band_max_pct`, or both 0 for none of the class.
- `target_pct` must sit inside the band, so it needs a band, or the call fails with `TARGET_OUT_OF_BAND`.
- Set each size pair together (`size_*`, `short_size_*`), or clear both of a pair with `null`.
- Set both band fields to `null`, with no size field, to remove the class's rule. With a size field they clear the band and keep the rule.
- An unknown class name returns `NOT_FOUND` with the valid names. Pass `create_class: true` only when the user asked for a new class; the result then says `created_class`.

It returns the rule before and after, and a `restore_point_id`, so `restore_policy` undoes it like an import. Show the user the before and after.

## When the connection can only read

`READ_ONLY_KEY` or `OAUTH_WRITE_NOT_GRANTED` means this connection can only read. Don't retry. Tell the user what couldn't be written and that nothing was changed.
To allow changes, the user opens Allocait, sets this app's access to Read and change under Settings › Agents, and starts a new chat. Reconnecting alone can keep the old access.
A message that says this account's agent access is read-only has no setting that lifts it. Show it as it is and don't suggest one.

## Error codes

`POLICY_INVALID` (parse failed, every issue listed), `TARGET_OUT_OF_BAND` (a target outside its own band on either write path), `NOT_FOUND` (a `scenario_id` you do not own, or an unknown class name, with the valid names), `INVALID_INPUT` (a field that's missing or out of range), `READ_ONLY_KEY` or `OAUTH_WRITE_NOT_GRANTED` (see above). Tell the user in plain words what went wrong; do not retry a `POLICY_INVALID` without fixing what it names.
