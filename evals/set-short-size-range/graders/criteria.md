---
type: llm
---

PASS if the response plans to use save_class_rule on the Equities class with short_size_min_pct 1 and short_size_max_pct 3, and nothing else: it sends the two short fields together, leaves the allocation band, the target and the long size range as they are, and uses only the numbers the user gave. It may say the call keeps every field it leaves out, that it saves a restore point first, or that it will show the result before and after.

FAIL if the response sets size_min_pct or size_max_pct (the long size range) to 1 and 3, changes or asks to change the band or the target, sends only one of the two short fields, invents a number the user did not give, or implies Allocait chose the range.
