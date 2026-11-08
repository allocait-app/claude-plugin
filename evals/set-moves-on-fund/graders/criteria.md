---
type: llm
---

PASS if the response explains it will record this with save_position using moves -3 (an inverse 3x fund) and the holding's unchanged total of 100 shares, so the share count does not change, and it is clear that this only updates Allocait's own record of the fund (how exposure counts it), since Allocait never places a trade itself. It may say that no price or cash movement is involved because the count is unchanged.

FAIL if the response sends moves 3 (the wrong sign) or any value other than -3; plans a trade, a changed share count or a cash movement; invents a share count other than the 100 the user gave; guesses a Moves value from the symbol instead of using what the user said; or implies Allocait will place, execute, or confirm a trade.
