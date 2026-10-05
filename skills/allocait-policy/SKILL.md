---
name: allocait-policy
description: Write or import a POLICY.md rule file for an Allocait scenario, or edit its allocation bands and position-size ranges directly. Use when the user wants to set up their rules, paste an investment policy statement, or edit their allocation bands.
---

A scenario's rules are the allocation band (and optional position-size range) Allocait checks each asset class against. **Allocait never picks a range for you and never recommends an allocation.** Every number that ends up in a rule is the user's own; your job is to capture it accurately, show a diff before writing, and never invent or round a figure the user did not give you.

## Read the rules first

`get_policy` (read) takes a `scenario_id`, or a `portfolio_id` for that portfolio's active scenario. It returns every asset class with its id and aliases, each rule's band (`band_min_pct`/`band_max_pct`), target (`target_pct`) and size range (`size_min_pct`/`size_max_pct`), the same rules as POLICY.md text, and the 5 latest restore points. With neither argument, it returns the asset classes only. Call it before any change, and to find a `restore_point_id` in a new chat.

## Two ways to set rules

- **`save_class_rule`**: one asset class's band, target or size range, keeping every other rule. Use this when the user changes one class ("set Equities to 50-70%").
- **POLICY.md** (`preview_policy` / `apply_policy` / `restore_policy`): the whole rule set for one scenario as one text file. Use this when the user has notes, a spreadsheet dump, or an investment policy statement to convert, or wants a portable file they can read and edit outside Allocait.

`apply_policy` is **absolute-state**: the file becomes the scenario's entire rule set, and any class not in the file loses its rule. Neither tool renames, reorders, or deletes an asset class.

## The POLICY.md format

Where your client offers MCP resources, `allocait://policy/spec` has the full spec and worked examples; claude.ai doesn't offer them. The short version below is enough to write a valid file, and `get_policy` returns the scenario's current file to start from:

```markdown
---
version: 1
name: Core
classes:
  Equities: { target: 60, min: 50, max: 70, position_size: { min: 2, max: 8 } }
  Fixed Income: { target: 30, min: 20, max: 40 }
  Cash: { min: 2, max: 10 }
---

Free text. Allocait ignores everything below the second --- line.
```

- Frontmatter only: `version` (always 1), optional `name`, required `classes` (at least one entry).
- Each class needs a `min`, a `max`, or both; that is its band. A class with only a `target` is invalid, Allocait will not pick the other bound for you.
- `target`, when given, must sit between `min` and `max`.
- A `min` above 0 on a class the user holds nothing in is a breach as soon as the rule is written: Allocait judges an empty class at 0%. If the user means "no floor until I buy in", the `min` is 0; ask rather than assume.
- `position_size: { min, max }` is optional: the range one single holding in that class should sit in, as % of the whole portfolio. `max` is required; omit `min` for a cap with no floor (`{ max: 5 }` = no single holding over 5%, no minimum).
- Known class names (use these when they fit): Cash, Equities, ETFs, Funds, Fixed Income, Currencies, Crypto, Commodities, Options. A name resolves in one order everywhere: the user's own class with that name, then the system class, then an alias (Stocks/Equity -> Equities, Bonds -> Fixed Income, Forex/FX -> Currencies), which also goes to the user's own class of that name first. A name close to a known one but not matching (e.g. "Global Equities") becomes its own custom class unless the user says to fold it into the matching one.
- An unknown key in a rule, a tab character, or content before the first `---` is a parse error, not a silent skip.

## The interview prompt, not your own numbers

The user can run the `draft_policy` prompt (optional `notes` argument) from their client's prompt menu; you can't run it yourself. If they haven't, ask the same plain questions it would, rather than choosing numbers yourself:

- No notes: ask a few plain questions, one at a time (which asset classes, and a min and max, plus an optional target and size range, for each).
- With notes (pasted text, a spreadsheet dump, an investment policy statement): read the ranges out of that text instead of asking. Never invent a number that is not in the notes.

## Preview before you write, always

`preview_policy` (read, `scenario_id` + `policy` text) parses the file and returns a diff: every class that would be added, changed, or removed, old value next to new, plus `classes_to_create` for any brand-new custom class the file implies. `written_as` on a changed entry shows the name exactly as written in the file when it differs from the resolved class name (e.g. an alias match). **Always call `preview_policy` and show the user the diff before `apply_policy`** with the identical text; `apply_policy` carries no dry-run of its own and writes on every successful call.

Both `preview_policy` and `apply_policy` enforce the same caps before parsing/applying: the file must be at most 102,400 bytes (100 KiB) and describe at most 500 classes. A parse error (missing frontmatter, a class with no min/max, an unknown key, a duplicate class name, a target outside its own band, ...) fails with `POLICY_INVALID` and every issue found, before anything reaches the database. A warning (e.g. targets or minimums summing over 100%) never blocks the preview or the write; it comes back in `issues` alongside the diff, so surface it to the user but do not treat it as a failure.

`apply_policy` saves a **restore point** of the scenario's prior rules before writing and returns its id as `restore_point_id`. Mention `restore_policy` (takes a `restore_point_id`) as the undo path. It puts the rules back exactly as they were at that point, so changes made after it are undone too, and it saves a fresh restore point first, so a restore can itself be undone. A class the original import created gets deleted on restore only if nothing still references it (no position rule, position, or watchlist item).

## `save_class_rule`: one class, everything else kept

Takes `scenario_id` (required), `asset_class` (a name, alias or id), and the fields to change: `band_min_pct`, `band_max_pct`, `target_pct`, `size_min_pct`, `size_max_pct`. A field you leave out keeps its value.

- A new rule needs both band fields, and `band_min_pct` must be below `band_max_pct`.
- `target_pct` must sit inside the band, or the call fails with `TARGET_OUT_OF_BAND`.
- Set the size fields together, or clear both with `null`.
- Set both band fields to `null` to remove the class's rule.
- An unknown class name returns `NOT_FOUND` with the valid names. Pass `create_class: true` only when the user asked for a new class; the result then says `created_class`.

It returns the rule before and after, and a `restore_point_id`, so `restore_policy` undoes it like an import. Show the user the before and after.

## Error codes

`POLICY_INVALID` (parse failed, every issue listed), `TARGET_OUT_OF_BAND` (a target outside its own band on either write path), `NOT_FOUND` (a `scenario_id` you do not own, or an unknown class name, with the valid names), `INVALID_INPUT` (a field that's missing or out of range), `READ_ONLY_KEY` or `OAUTH_WRITE_NOT_GRANTED` (the connection can only read; don't retry). Show the message to the user; do not retry a `POLICY_INVALID` without fixing what it names.
