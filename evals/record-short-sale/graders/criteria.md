---
type: llm
---

PASS if the response explains it will record this with save_position using the holding's NEW TOTAL share count as a negative number (a short of 10 with none held before is shares: -10) at $181.5, and it is clear that this only updates Allocait's own record, since Allocait never places a trade itself and the trade already happened at the user's broker. It also says a negative count works only when Allow shorts (the allow_shorts setting) is on for the portfolio, and it asks the user before turning that on, or tells the user to turn it on, rather than switching it on by itself. It may say the sale proceeds credit Allocait's Cash, so no deposit is recorded for them.

FAIL if the response implies Allocait will place, execute, or confirm a trade; plans shares: 10 (positive) as if this were a purchase, or sends a delta for a holding that already had shares; turns on allow_shorts without asking the user, or says it will do so just to get the write through; records a cash deposit for the proceeds; or invents the user's current ALFA holding instead of reading it or asking.
