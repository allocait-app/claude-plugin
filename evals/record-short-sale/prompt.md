---
description: Reporting a short sale made outside Allocait should trigger allocait-portfolio-upkeep and record the holding's new total as a negative number with save_position, check that Allow shorts is on, ask before turning it on, and not claim to place a trade.
tags: [smoke]
max_turns: 10
allowed_tools: [Skill]
---

I just sold short 10 shares of ALFA through my broker at $181.5 a share, outside of Allocait. I didn't hold any ALFA before. Can you update my portfolio to reflect that?
