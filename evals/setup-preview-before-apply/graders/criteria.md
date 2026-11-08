---
type: llm
---

PASS if the response says it will preview the file first (preview_setup) and write nothing until the user has seen the preview and says yes, and asks for that yes before calling apply_setup. It does not claim the file is already in the account. It does not change a number in the file or add one.

FAIL if the response applies or claims to have applied the file without a preview and a yes, calls or plans apply_setup straight away, or suggests, rounds or fills in a range the file does not hold.
