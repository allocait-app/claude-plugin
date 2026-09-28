---
name: allocait-feedback
description: Report a problem with the Allocait agent surface, or file a support request about the user's own account, billing, or a product issue. Use when the user wants to report a bug, request a feature, flag a confusing or wrong tool, or ask a billing question about Allocait.
---

Two different tools exist here, and picking the right one matters: one is about **you** (the agent surface itself), the other is about **the user** (their account, billing, or the product). Never send a user's account or billing question through `submit_agent_feedback`, and never send a tool/API complaint through `submit_support_request`.

## `submit_agent_feedback`: something about the tools, resources, or docs is wrong

Use this when a tool or route behaved unexpectedly, a tool description or a doc (this skill included, or `docs/openapi.yaml`) is wrong or ambiguous, or you needed a capability that does not exist (e.g. a missing batch endpoint). This is feedback *for the Allocait team about the MCP/API surface*, not a request on the user's behalf.

Fields:
- `kind` (required): `bug` (a tool/route behaved unexpectedly), `docs` (a description or doc entry is wrong or ambiguous), `missing_capability` (an operation you needed does not exist), or `other`.
- `summary` (required, 1-200 characters): one line.
- `details` (optional, up to 5000 characters): the fuller explanation, reproduction steps, or what you expected instead.
- `tool_or_route` (optional, up to 100 characters): e.g. `"submit_support_request"` or `"POST /compliance/check"`.
- `error_code` (optional, up to 100 characters): the `error_code` from a response this concerns, if any.
- `request_id` (optional, up to 100 characters): any identifier you already had for this exchange; Allocait does not issue one back on this surface today.
- `client` (optional, up to 200 characters): free text identifying your own client (name/version/model), if you want it recorded.

Never attaches any portfolio data automatically, only what you write in `summary`/`details`; do not paste the user's holdings or rules into this unless they are directly relevant to reproducing the bug. Filing this notifies the Allocait team directly (it also reads back through the `allocait://agent-feedback` resource), so do not additionally tell the user to email anyone unless the tool call itself fails.

## `submit_support_request`: the user's own account, billing, or product issue

Use this when the user asks you to report a problem or contact support on their behalf, rather than something to fix or work around yourself. Fields: `subject` (required, 1-200 characters), `message` (required, 1-5000 characters), `category` (required: `bug`, `billing`, `feature_request`, or `other`), and optional `context` (a free-form object, e.g. which portfolio or page this concerns). Filed with source `agent`; reads back through the `allocait://support-requests` resource.

## Deciding which one to use

- The user says "Allocait won't let me..." about their own account, a bill, or something in the app itself -> `submit_support_request`.
- A tool call you made returned an error you did not expect, or its description did not match what happened, or you could not find an operation you needed -> `submit_agent_feedback`.
- If genuinely unsure which it is, ask the user briefly rather than guessing, since the two land in different queues.

## Error codes

`RPC_ERROR` (database failure) is the only code either tool returns beyond validation errors from the field limits above (e.g. a `summary` or `subject` over its character limit). Show the message to the user and offer to retry once the field is fixed.
