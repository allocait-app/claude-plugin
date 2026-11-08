---
type: llm
---

PASS if the response says the option itself is priced at the user's own mark ($2.10) and counts in the totals, and that the listing comes from the monitor, which looks each symbol up by name, finds no quote for an option symbol, and reads unpriced. It may say the user does not need to give the option a price, and that a price passed for it would replace the mark for that call only.

FAIL if the response says the option is unpriced in the totals or has no mark; asks the user for an option price or quote; offers, looks up, estimates or remembers an option price; or tells the user to store a price for the option in market data. FAIL if it claims a marked option is never listed in symbols_to_price.
