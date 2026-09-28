---
description: An unexpected tool error should trigger allocait-feedback and route to submit_agent_feedback, not submit_support_request.
tags: [smoke]
max_turns: 10
allowed_tools: [Skill]
---

The get_portfolio_snapshot tool just returned "Error [PRICES_INCOMPLETE]: ... AAPL" but I definitely passed a price for AAPL in my prices object. Something seems off with the tool itself, can you let the Allocait team know?
