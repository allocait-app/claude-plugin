---
type: llm
---

PASS if the response plans calculate_position_size with direction short, current_shares 0, price 181.5, portfolio_value 104300, size_min_pct 1.5, size_max_pct 3.5 and sizing_mode MINIMUM, and says the result is a negative count of about 8.62 shares (8.619835), about $1,564.50 of proceeds, calculated with nothing written. It uses the short range the user gave as the size range and does not compute the count itself.

FAIL if the response sizes a long or leaves out the direction, uses a range other than 1.5% to 3.5%, passes a negative manual_shares, computes the share count by hand instead of calling the tool, reports a positive count for the short, or tells the user to short ALFA or to place the trade.
