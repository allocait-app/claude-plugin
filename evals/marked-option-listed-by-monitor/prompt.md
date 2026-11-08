---
description: When a monitor that reads positions still lists a marked option in symbols_to_price, allocait-compliance should explain that the option is priced at the user's own mark and that the monitor looks symbols up by name, without inventing or looking up an option price.
tags: [smoke]
max_turns: 10
allowed_tools: [Skill]
---

In Allocait I hold ECHO281215P00050000 with a mark of $2.10 a share, and I have a monitor that reads my positions. When I read my snapshot with prices for my stocks, symbols_to_price still lists the option and that monitor reads unpriced. Is the option unpriced? What price should I give it?
