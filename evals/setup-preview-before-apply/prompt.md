---
description: A setup file someone sent should fire allocait-setup, be previewed before anything is written, and be applied only after a clear yes.
tags: [smoke]
max_turns: 10
allowed_tools: [Skill]
---

A colleague sent me this setup file for Allocait. Can you add it to my account?

```markdown
---
format: allocait.setup-file
version: 1
id: example-setup
revision: 1
name: Example setup
author: Example Author
description: One scenario with a range for two classes. Every name and number is made up.
disclosure: These numbers are the author's own, made up for the example.
scenarios:
  - name: Example scenario
    classes:
      Equities: { min: 11, max: 73, position_size: { min: 3, max: 9 } }
      Cash: { min: 7, max: 31 }
---
```
