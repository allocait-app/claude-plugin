---
name: allocait-feedback
description: Report a problem with the Allocait tools, or ask the Allocait team for help with the user's own account, billing, or a product issue. Use when the user wants to report a bug, request a feature, flag a confusing or wrong tool, or ask a billing question about Allocait.
---

One tool, `send_feedback`, reaches the Allocait team, and its `topic` decides which queue the report lands in. Picking the right topic matters: `tools` is about **you** (the tools, their descriptions, the docs), and `account` is about **the user** (their account, billing, or the product). Never send a user's account or billing question as `topic: tools`, and never send a tool or API complaint as `topic: account`.

`send_feedback` works on read-only connections too, because it changes no portfolio data.

## `topic: tools`: something about the tools or docs is wrong

Use this when a tool behaved unexpectedly, a tool description or a doc (this skill included) is wrong or ambiguous, or you needed a capability that doesn't exist. This is feedback for the Allocait team about the tools, not a request on the user's behalf.

Fields:
- `kind` (required): `bug` (a tool behaved unexpectedly), `docs` (a description or doc is wrong or ambiguous), `missing_capability` (an operation you needed doesn't exist), or `other`.
- `summary` (required, 1-200 characters): one line.
- `details` (optional, up to 5000 characters): the fuller explanation, reproduction steps, or what you expected instead.
- `tool` (optional): the tool this concerns, for example `"get_portfolio_snapshot"`.
- `error_code` (optional): the code from the error this concerns, if any.
- `request_id` (optional): the `request_id` from an `INTERNAL_ERROR`, if there was one.
- `client` (optional): your own client's name, version or model, if you want it recorded.

## `topic: account`: the user's own account, billing, or product issue

Use this when the user asks you to report a problem or contact support on their behalf. Fields: `kind` (required: `bug`, `billing`, `feature_request`, or `other`), `summary` (required, one line), and `details` (required for this topic: what the user wants help with). It's filed as coming from an agent.

## Deciding which topic

- The user says "Allocait won't let me..." about their own account, a bill, or something in the app itself: `topic: account`.
- A tool call you made returned an error you did not expect, its description did not match what happened, or you could not find an operation you needed: `topic: tools`.
- If you can't tell which it is, ask the user briefly rather than guessing, since the two land in different queues.

Never attach portfolio data on your own. Only what you write in `summary` and `details` is sent, so don't paste the user's holdings or rules unless they're needed to reproduce the problem. Filing notifies the Allocait team, so don't also tell the user to email anyone unless the call itself fails.

## Error codes

`INVALID_INPUT` (a field over its limit, or `details` missing for `topic: account`; fix it and try again), `DAILY_LIMIT_EXCEEDED` (20 reports in 24 hours; try again later), `INTERNAL_ERROR` (retry once). Show the message to the user.
