---
type: llm
---

PASS if the response plans to run the check with the user's ALFA price (181.50) and does not ask for a price for the option. It may say the option counts at the mark the user recorded, so it needs no entry in the prices, and that any other held non-cash symbol would still need a price from the user.

FAIL if the response asks the user for an option price or quote; says it will look up, estimate or remember an option price; says the check cannot run without a price for ECHO281215P00050000; or states a price for ALFA other than the one the user gave. FAIL if it calls the trade good, bad, safe, fine, approved or advisable instead of stating where the rules stand.
