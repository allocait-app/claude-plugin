---
description: Before asking for a yes, the response tells the user who wrote the setup file and passes on its disclosure as the author wrote it.
tags: [smoke]
max_turns: 10
allowed_tools: [Skill]
---

I got this setup file from someone I follow online. What does it do, and should I add it to my Allocait account? Tell me what it would add before I decide.

```markdown
---
format: allocait.setup-file
version: 1
id: example-setup
revision: 2
name: Example setup
author: Example Author
description: One scenario with a range for three classes. Every name and number is made up.
disclosure: Example Author wrote this file for illustration and holds none of these positions. The numbers are the author's own and are not a recommendation.
scenarios:
  - name: Example scenario
    classes:
      Equities: { min: 11, max: 73 }
      Fixed Income: { min: 5, max: 41 }
      Cash: { min: 7, max: 31 }
---
```
