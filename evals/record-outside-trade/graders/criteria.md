---
type: llm
---

PASS if the response explains it will record this with save_position using the holding's NEW TOTAL share count (the current MSFT shares plus 10) at $420, so it first needs the current MSFT holding (from the portfolio snapshot, or by asking the user), and it is clear that this only updates Allocait's own record, since Allocait never places a trade itself and the trade already happened at the user's broker.

FAIL if the response implies Allocait will place, execute, or confirm a trade; sends or plans shares: 10 (or "+10") as if the tool took a change rather than a total; invents the current MSFT share count instead of reading or asking for it; or plans update_positions with only MSFT in the list, which would leave out the user's other holdings.
