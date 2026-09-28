# Allocait

Allocait is a process layer for a portfolio: it records holdings, a user's own written allocation rules, and the decisions taken against them. It never places a trade and never recommends buying, selling, or holding anything. Every check this plugin runs goes against rules the user supplied, not a model of the market.

## Use it

Ask Claude to check a portfolio against its rules, plan a rebalance, size a new position, record a trade made outside Allocait, or write a POLICY.md rule file. Claude calls the Allocait MCP connector's tools, such as `get_portfolio_snapshot` and `preview_rebalance`, and asks before any write a tool marks destructive. Prices always come from the caller; Allocait never returns or infers a price of its own.

## Set up (Claude Code)

Allocait's OAuth connector is still being built. Until it ships, this plugin connects with an API key instead: open Settings > Developer in the Allocait app, create an agent API key, then enter it when Claude Code prompts for "Allocait API key" after you enable the plugin. Claude Code sends that key only as the `x-api-key` header on calls to Allocait's server, and stores it in your system's secure credential store, not in a file.

On claude.ai and in Cowork, the bundled connector needs the OAuth sign-in flow this plugin does not carry yet. Add the connector there once Allocait's directory listing drops this interim API key field for OAuth.

## Data

This plugin's skills send your prompts and the connector's tool results through the conversation, the way any MCP tool does. The Allocait connector reads and writes only your own portfolios, scenarios, positions, and rules in your Allocait account. Claude never sees another user's data, and Allocait never executes a trade or moves money on your behalf. See https://allocait.app/privacy for the full privacy policy.

## Support

Email support@allocait.app. Agents connected through this plugin can also report tool or API problems directly with the `submit_agent_feedback` tool.

## License

The MIT license in this folder covers only this plugin's own files (the manifests, skills, and eval cases). It does not extend to Allocait's application source, which lives in a separate, private repository.
