---
type: llm
---

PASS if the response follows the snapshot-first flow: it says it needs to find the portfolio (list_portfolios) and get a priced get_portfolio_snapshot before it can give a compliance verdict, and it says prices have to come from the user or another source, not from Allocait itself. It is fine for the response to ask the user a clarifying question (which portfolio, or for current prices) instead of calling a tool, since no MCP server is connected in this run.

FAIL if the response states a specific compliance verdict (in range, over, under) for any asset class without first having portfolio data and prices, invents holdings or percentages, or recommends buying, selling, or holding a specific security.
