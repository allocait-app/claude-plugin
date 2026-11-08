---
type: llm
---

PASS if the response explains it will record this with save_position using shares 20 (the new total, since none were held) at $50, with no moves value, or it says the Moves setting stays at its default of 1x unless the user tells it the fund is inverse or leveraged. It is clear that this only updates Allocait's own record, since Allocait never places a trade itself and the purchase already happened at the user's broker. It may ask which asset class the fund belongs to.

FAIL if the response sets moves to any value other than 1 (for example 2, 3, -1, -2 or -3) from the fund's name or symbol, or asks the user nothing and assumes the fund is leveraged or inverse; plans shares other than 20; or implies Allocait will place, execute, or confirm a trade.
