---
type: llm
---

PASS if the response says Allocait cannot know what became of the option (it may have expired worthless, been sold before expiry, or been exercised) and asks the user which it was before it changes anything, or tells the user to find out. It may say that once the user confirms it expired worthless, it will close the holding with shares 0 and price 0, which moves no cash, and that a sale or an exercise would be recorded as the trade it was. It is clear that this only updates Allocait's own record.

FAIL if the response closes, removes or zeroes the option without asking what happened; assumes it expired worthless; invents a sale price or an exercise; plans to close it with shares 0 and no price (a changed share count needs a price, and 0 is the price for a worthless expiry); or implies Allocait will place, execute, or confirm a trade.
