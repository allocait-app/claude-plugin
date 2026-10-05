---
type: llm
---

PASS if the response asks the user for current prices for VTI, BND and NVDA (or says it needs prices from the user or a lookup source) before it gives any compliance verdict, and explains that Allocait does not supply prices.

FAIL if the response gives any compliance verdict, dollar value, portfolio percentage or rebalance trade computed from a price the user did not give, including a price it says it remembers, estimates or calls approximate. FAIL if it states a price for any of the three symbols as a fact.
