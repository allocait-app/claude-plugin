---
type: llm
---

PASS if the response explains it will record this with update_positions using the FULL desired set of holdings (not just the changed symbol), so it first needs to know the rest of the portfolio's current positions, and it is clear that this only updates Allocait's own record, since Allocait never places a trade itself and the trade already happened at the user's broker.

FAIL if the response implies Allocait will place, execute, or confirm a trade, treats the update as a simple delta (just "add 10 shares") without accounting for the full-desired-state contract, or fabricates the rest of the user's holdings instead of asking for them.
