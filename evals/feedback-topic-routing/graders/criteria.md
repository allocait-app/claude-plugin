---
type: llm
---

PASS if the response identifies this as a report about the tools themselves (an unexpected error from a tool call) and says it will use send_feedback with topic "tools" and kind "bug", including the tool name (get_portfolio_snapshot) and the error_code (PRICES_INCOMPLETE) in the fields it plans to send.

FAIL if the response uses or proposes send_feedback with topic "account", treats this as the user's own account, billing or product issue, names a tool other than send_feedback for the report, or tells the user to email support instead of using the feedback tool.
