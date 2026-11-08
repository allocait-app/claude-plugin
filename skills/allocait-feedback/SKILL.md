---
name: allocait-feedback
description: Report a problem with the Allocait tools, or ask the Allocait team for help with the user's own account, when the user asks. Use when the user wants to report a bug, request a feature, flag a confusing or wrong tool, or get help with their Allocait account.
---

`send_feedback` messages the Allocait team, who publish these tools. Use it only when the user asks you to report a problem or ask for help. Never file a report on your own, never file one after an automatic retry, and never file one to pass along an error the user has not asked you to report.

Only what you write in `summary` and `details` is sent. Do not include holdings, rules, balances or other account data. If the problem can't be described without them, describe it in general terms and leave the figures out.

## Pick the topic

- `tools`: a tool behaved unexpectedly, a tool description or a doc (this skill included) is wrong or ambiguous, or an operation the user needed doesn't exist. `kind` is `bug`, `docs`, `missing_capability` or `other`. Name the tool in `tool`, the code from its error in `error_code` and the `request_id` of an `INTERNAL_ERROR` in `request_id`. In `details` (up to 1000 characters) say what you called, what you expected and what came back.
- `account`: the user's own account, or something in the app itself ("Allocait won't let me..."). `kind` is `bug`, `billing`, `feature_request` or `other`. `details` is required: what the user wants help with. It's filed as coming from an agent.
- If you can't tell which, ask the user briefly: the two land in different queues. Never send an account question as `tools`, or a tool complaint as `account`.

`tools` works on a read-only connection. `account` doesn't: it returns `READ_ONLY_KEY` or `OAUTH_WRITE_NOT_GRANTED`, so tell the user to use Contact support in the app.

A successful call returns `{ topic, filed: true, message }`. Tell the user in your own words that the report was sent. Don't promise a reply or a fix, and don't also tell the user to email anyone unless the call itself fails.

## Errors

`INVALID_INPUT` (a field over its limit, or `details` missing for `account`; fix it and try again), `DAILY_LIMIT_EXCEEDED` (stop for now; the limit resets within a day), `INTERNAL_ERROR` (retry once, and if it fails again tell the user and give them its `request_id`; don't file a second report about the failed one).
