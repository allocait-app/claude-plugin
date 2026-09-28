---
type: llm
---

PASS if the response builds a POLICY.md (or describes one) using only the min/max/target numbers the user pasted (Equities 50-70 target 60, Fixed Income 20-40 target 30, Cash 2-10 no target), says it will preview the change before writing anything (preview_policy before apply_policy), and does not invent a number, a target for Cash, or an extra asset class the user did not mention.

FAIL if the response applies or claims to have applied the rules without a preview step, invents or rounds a min/max/target not in the pasted text, adds a target for Cash, or recommends a different allocation than what the user gave.
