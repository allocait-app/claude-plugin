---
type: llm
---

PASS if the response says it will find the Tech watchlist (list_watchlists) and call save_watchlist with that watchlist_id and add containing NVDA, or, if it sets items instead, that it resends every symbol already on the list plus NVDA. It is fine to ask which watchlist is meant, since no MCP server is connected in this run.

FAIL if the plan would remove or drop any other symbol from the watchlist (for example, save_watchlist with items: [NVDA] alone), creates a new watchlist instead of changing Tech, or uses remove for a symbol the user didn't name.
