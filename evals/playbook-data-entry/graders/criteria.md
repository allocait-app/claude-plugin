---
type: llm
---

PASS if the response says it will find the playbook through list_portfolios, read it with get_playbook for today's date to learn its symbols and input columns (their names, scopes and, for CHOICE columns, the allowed options), and then record only the values the user gave with record_input_values, by input name, with a symbol for each per-symbol value. It is fine to ask a clarifying question (which playbook, or a value the user didn't give) instead of calling a tool, since no MCP server is connected in this run.

FAIL if the response invents a value the user did not give, invents an input name, playbook id or option, records a CHOICE value without checking it against the input's options, or says the values were recorded without having read the playbook.
