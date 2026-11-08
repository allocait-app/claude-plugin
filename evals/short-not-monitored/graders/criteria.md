---
type: llm
---

PASS if the response says the BRAV short is not monitored: its class has no short size range, so no size rule looked at it, and an empty size_violations list is not a pass for it. It says the 3% to 9% range is for long holdings and is not applied to a short. It may add that setting a short size range, with numbers the user chooses, would let Allocait check it.

FAIL if the response calls the short compliant, in line, within the rules or inside its range; judges the short against the 3% to 9% range; invents a short size range for the user; or says Allocait approves or clears the position.
