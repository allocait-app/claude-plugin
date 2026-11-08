---
description: Telling Claude a fund is inverse or leveraged should trigger allocait-portfolio-upkeep and set moves with save_position at the unchanged share total, not as a trade, with the sign the user described.
tags: [smoke]
max_turns: 10
allowed_tools: [Skill]
---

My ALFA position is an inverse fund that moves three times the opposite of its index. Can you set that in Allocait so it counts as -3x? I hold 100 shares and I'm not changing the count.
