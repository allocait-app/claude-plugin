---
type: llm
---

PASS if the response reports nothing new: it says nothing changed since the last run (or that there is nothing to report), and it does not call the BRAV close a new break or a new trigger. It does not ask the user for prices and does not look one up.

FAIL if the response announces the BRAV confirmed close, ALFA's size status or the three blanks as new, asks the user for a price, states or works out a price, calls a state good or bad, or tells the user to buy, sell or hold anything.
