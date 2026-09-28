---
name: allocait-policy
description: Write or import a POLICY.md rule file for an Allocait scenario, or edit its allocation bands and position-size ranges directly. Use when the user wants to set up their rules, paste an investment policy statement, or edit their allocation bands.
---

A scenario's rules are the allocation band (and optional position-size range) Allocait checks each asset class against. **Allocait never picks a range for you and never recommends an allocation.** Every number that ends up in a rule is the user's own; your job is to capture it accurately, show a diff before writing, and never invent or round a figure the user did not give you.

## Two ways to set rules; prefer POLICY.md when the user has text to work from

- **POLICY.md** (`preview_policy` / `apply_policy` / `restore_policy`): the whole rule set for one scenario as one text file. Use this when the user has notes, a spreadsheet dump, or an investment policy statement to convert, or wants a portable file they can read and edit outside Allocait.
- **`apply_position_rules`**: the same "full desired rule set" write, expressed as structured JSON instead of a file. Use this only when the user is dictating bands one at a time with no document to import, and a POLICY.md round-trip would just be overhead.

Both are **absolute-state**: whatever you send becomes the scenario's entire rule set. Any asset class rule the scenario currently has that you do not include is removed. Neither tool ever deletes, renames, or reorders an asset class on its own; a class name in the input that matches nothing existing becomes a new custom class.

## The POLICY.md format

Read `allocait://policy/spec` (or the `draft_policy` prompt) for the authoritative spec and worked examples before writing one by hand. The short version:

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
- `position_size: { min, max }` is optional: the range one single holding in that class should sit in, as % of the whole portfolio. `max` is required; omit `min` for a cap with no floor (`{ max: 5 }` = no single holding over 5%, no minimum).
- Known class names (use these when they fit): Cash, Equities, ETFs, Funds, Fixed Income, Currencies, Crypto, Commodities, Options. A name close to one of these but not exact (e.g. "Global Equities") becomes its own custom class unless the user says to fold it into the matching one. A name that matches a known alias (Stocks/Equity -> Equities, Bonds -> Fixed Income, Forex/FX -> Currencies) resolves to that system class on import, but only when the user has no custom class of their own with that exact name already.
- An unknown key in a rule, a tab character, or content before the first `---` is a parse error, not a silent skip.

## The interview prompt, not your own numbers

Use the `draft_policy` MCP prompt (optional `notes` argument) to write a new POLICY.md rather than composing YAML from a conversation yourself:

- No `notes`: it asks the user a few plain questions, one at a time (which asset classes, and a min/max, and optionally a target and size range, for each).
- With `notes` (pasted text, a spreadsheet dump, an investment policy statement): it reads ranges out of that text instead of asking, the same way it would read any pasted document. It still never invents a number that is not in the notes.

## Preview before you write, always

`preview_policy` (read, `scenario_id` + `policy` text) parses the file and returns a diff: every class that would be added, changed, or removed, old value next to new, plus `classes_to_create` for any brand-new custom class the file implies. `written_as` on a changed entry shows the name exactly as written in the file when it differs from the resolved class name (e.g. an alias match). **Always call `preview_policy` and show the user the diff before `apply_policy`** with the identical text; `apply_policy` carries no dry-run of its own and writes on every successful call.

Both `preview_policy` and `apply_policy` enforce the same caps before parsing/applying: the file must be at most 102,400 bytes (100 KiB) and describe at most 500 classes. A parse error (missing frontmatter, a class with no min/max, an unknown key, a duplicate class name, a target outside its own band, ...) fails with `POLICY_INVALID` and every issue found, before anything reaches the database. A warning (e.g. targets or minimums summing over 100%) never blocks the preview or the write; it comes back in `issues` alongside the diff, so surface it to the user but do not treat it as a failure.

`apply_policy` saves a **restore point** of the scenario's prior rules before writing (reason `import`) and returns its id as `restore_point_id`. Mention `restore_policy` (takes a `restore_point_id`) as the undo path; it is itself undoable, since it saves a fresh restore point (reason `restore`) before reverting. A class the original import created gets deleted on restore only if nothing still references it (no position rule, position, or watchlist item).

To see a scenario's current rules as text, read the `allocait://scenarios/{scenario_id}/policy` resource (returns "No rules yet." for an empty scenario rather than a broken file) or `allocait://policy/restore-points` for every saved restore point across scenarios, newest first.

## `apply_position_rules`: the structured-JSON alternative

Takes `scenario_id` and a `rules` array, each item: `asset_class_id`, `min_position_pct`, `max_position_pct` (required; the allocation band), optional `target_allocation_pct` (must sit within that rule's own min/max or the whole call fails with `TARGET_OUT_OF_BAND` and writes nothing), and optional `size_min_pct`/`size_max_pct` (send both or neither; the position-size range for one holding). This has **no restore point** of its own, unlike `apply_policy`; read the rules back from the `allocait://position-rules` (or `allocait://scenarios/{scenario_id}/position-rules`) resource first if you need to preserve fields you are not changing, since anything you omit is removed, and a rule sent without both size keys has its size range cleared.

## Error codes

`POLICY_INVALID` (parse failed, every issue listed), `TARGET_OUT_OF_BAND` (a target outside its own min/max on either write path), `NOT_FOUND` (a `scenario_id` you do not own), `RPC_ERROR` (database failure). Show the message to the user; do not retry a `POLICY_INVALID` without fixing what it names.
