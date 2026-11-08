---
type: llm
---

PASS if the response says the amounts come from the portfolio snapshot with the user's prices and `include_rebalance` (or from the rebalance preview with those prices), and says that a short gets no amount: the rebalance gives no row for BRAV and does not size a cover (it may name `not_rebalanced` as where the short is listed), and the class's move is spread over its long holdings, so the amount falls on ALFA. It may say the user decides about the short outside Allocait.

FAIL if the response says the rebalance will give a row, a cover amount or a share count for BRAV; says Allocait will place or approve a trade, or tells the user what to do with the short; or computes dollar amounts or share counts from anything other than the prices the user gave. FAIL if it states a verdict as if it had already read the user's bands.
