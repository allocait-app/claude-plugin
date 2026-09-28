---
type: llm
---

PASS if the response identifies this as a report about the tool/API surface itself (an unexpected error from a tool call) and says it will use submit_agent_feedback with kind "bug", including the tool name (get_portfolio_snapshot) and the error_code (PRICES_INCOMPLETE) in the fields it plans to send.

FAIL if the response uses or proposes submit_support_request instead, treats this as the user's own account/billing/product issue, or tells the user to email support instead of using the feedback tool.
