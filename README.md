# Allocait

Allocait is a process layer for a portfolio: it records holdings, a user's own written allocation rules, and the decisions taken against them. It never places a trade and never recommends buying, selling, or holding anything. Allocait output states facts about the user's own rules. It is not advice or an approval. Every check this plugin runs goes against rules the user supplied, not a model of the market.

## Use it

Ask Claude to check a portfolio against its rules, check a change before you place it elsewhere, work out the amounts that close a gap, size a position inside your range, record a trade made outside Allocait, change a rule or write a POLICY.md rule file, keep a watchlist, or fill in a playbook. Claude calls the Allocait MCP connector's tools, such as `get_portfolio_snapshot`, `check_trades`, `save_position` and `save_watchlist`. The `allocait-check` skill covers the change check: it reports which of your rules a change puts outside range or leaves further outside, with the time your positions were last updated. A failed call is no result, and Allocait cannot approve, stop or place an order. A write removes a holding, a rule or a list member only when the call names it. Prices always come from the caller; Allocait never returns or infers a price of its own.

## Set up (Claude Code)

This plugin connects with an API key. In the Allocait app, open Settings > Agents, select Create API key under API keys, and choose its access: Read only, or Read and change. On the Core plan, an agent connects read-only: it can read, check and validate. Read and change comes with Advanced. Enter the key when Claude Code prompts for "Allocait API key" after you enable the plugin. Claude Code sends that key only as the `x-api-key` header on calls to Allocait's server, and stores it in your system's secure credential store, not in a file.

You can also connect without a key: add `https://api.allocait.app/functions/v1/mcp` as a connector, sign in to Allocait, and choose Read only or Read and change on the consent screen (Read and change comes with Advanced). On claude.ai and in Cowork, the bundled connector uses that sign-in.

## Data

This plugin's skills send your prompts and the connector's tool results through the conversation, the way any MCP tool does. The Allocait connector reads and writes only your own portfolios, scenarios, positions, and rules in your Allocait account. Claude never sees another user's data, and Allocait never executes a trade or moves money on your behalf. See https://allocait.app/privacy for the full privacy policy.

## Support

Email support@allocait.app. Agents connected through this plugin can also contact the team directly with the `send_feedback` tool: `topic: tools` for a problem with the tools, `topic: account` for help with the user's account (this needs a connection with Read and change).

## License

The MIT license in this folder covers only this plugin's own files (the manifests, skills, and eval cases). It does not extend to Allocait's application source, which lives in a separate, private repository.
