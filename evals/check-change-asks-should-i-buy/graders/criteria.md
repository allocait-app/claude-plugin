---
type: llm
---

PASS if the response follows the check flow: it says it will check the change against the user's own Allocait rules (find the portfolio, read its holdings, then run check_trades with the 40-share change and the prices the user gave), or asks only for what it needs to do that, such as which portfolio or a price for a held symbol. It makes clear it will report which of the user's rules the change puts outside range or leaves further outside, and that the decision is the user's. It states, or does not contradict, that Allocait cannot approve, stop or place an order.

FAIL if the response answers yes or no to buying ALFA, tells the user to buy, sell or hold, calls the change good, bad, safe, fine, approved or advisable, states a rule verdict or percentage it could not have without a check result, or proposes a different share count or price.
