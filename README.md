# Allocait

Allocait is a process layer for a portfolio: it records holdings, a user's own written allocation rules, and the decisions taken against them. It never places a trade and never recommends buying, selling, or holding anything. Allocait output states facts about the user's own rules. It is not advice or an approval. Every check this plugin runs goes against rules the user supplied, not a model of the market.

## Use it

Ask Claude to check a portfolio against its rules, check a change before you place it elsewhere, work out the amounts that close a gap, size a position inside your range, record a trade made outside Allocait, change a rule or write a POLICY.md rule file, keep a watchlist, fill in a playbook, or add a setup file you were sent, after a preview you approve, and undo it later. Claude calls the Allocait MCP connector's tools, such as `get_portfolio_snapshot`, `check_trades`, `save_position` and `save_watchlist`. The `allocait-check` skill covers the change check: it reports which of your rules a change puts outside range or leaves further outside, with the time your positions were last updated. A failed call is no result, and Allocait cannot approve, stop or place an order. A write removes a holding, a rule or a list member only when the call names it. Undoing a setup file import removes only what that import added that you haven't changed or used. Prices always come from the caller; Allocait returns no price of its own. A connection you sign in to yourself may also return `standing`, percents and labels read from the last stored quote at the time shown.

### Example prompts

Each example uses fictional symbols and placeholder numbers. Give Claude the prices, because Allocait supplies none.

- "Check my Main Portfolio against my rules. Prices: ALFA 181.5, BRAV 94.3, CHAR 62.7." Claude reads the portfolio and reports the verdict for each asset class and each holding outside its size range.
- "Before I place it elsewhere: what does adding 40 shares of ALFA at 181.5 do to my rules? BRAV is 94.3 and CHAR is 62.7." Claude calls `check_trades` and reports which rules the change puts outside range or leaves further outside. It gives no yes or no.
- "I bought 10 more shares of BRAV at 94.3 through my broker. Record it." Claude reads the current BRAV holding and saves the new total with `save_position`. This records the change in Allocait only. It moves no money and places no order.

## Set up

This plugin connects by signing in. It stores no API key.

1. **Claude Code.** Install the plugin, run `/mcp`, choose Allocait, then select Authenticate and sign in to Allocait in your browser.
2. **claude.ai and Cowork.** Open the plugin's Connectors tab, select Connect for Allocait, and sign in. On a Team or Enterprise organization, an Owner adds the Allocait connector first, then members connect with their own accounts.
3. **On the consent screen,** choose Read only or Read and change. Read only lets Claude read and check. Read and change also lets it save to your Allocait account.

You need an Allocait account with an active plan. The sign-in screen shows which access your plan offers.

Another MCP client can connect with an API key instead. This route is for clients other than this plugin, which never reads, sends or stores an API key. Create a key in the Allocait app under Settings > Agents, then add the server with the key as a header. Replace the placeholder with your own key:

```bash
claude mcp add --transport http allocait https://api.allocait.app/functions/v1/mcp --header "x-api-key: YOUR_ALLOCAIT_API_KEY"
```

The [connection guide](https://docs.allocait.app/connect-an-agent) covers both ways.

## Troubleshooting

- **Claude doesn't list the Allocait tools.** The connector isn't connected yet. In Claude Code run `/mcp`. On claude.ai and Cowork open the plugin's Connectors tab. Then sign in.
- **A call is refused because the account has no active plan.** Agent access is off until the account has an active plan. Manage the account in the Allocait app, then try again.
- **Claude can read but a save is refused.** The connection is Read only. In Allocait, set the app's access to Read and change under Settings > Agents > Connected apps, then start a new chat.
- **A check or snapshot asks for prices.** Allocait never supplies prices. Give Claude a price for every symbol you hold and every symbol in the change, and ask again. Without them a call returns no verdict.
- **Anything else.** Email support@allocait.app.

## Data

This plugin's skills send your prompts and the connector's tool results through the conversation, the way any MCP tool does. The Allocait connector reads and writes only your own portfolios, scenarios, positions, and rules in your Allocait account. Claude sees only what your account's tools return, and Allocait never executes a trade or moves money on your behalf. The plugin runs no scripts and fetches nothing else. The only server it contacts is `https://api.allocait.app/functions/v1/mcp`. See https://allocait.app/privacy for the full privacy policy.

One tool sends free text to a person. `send_feedback` files a short report with the Allocait team: a one-line summary, optional details, and the name of the tool and its error code when there is one. Claude uses it only when you ask it to, and leaves holdings and account details out of the report. A report about the account itself (`topic: account`) needs a Read and change connection. This plugin is not intended for people under 18, and Allocait's Terms require users to be at least 18.

## Support

Email support@allocait.app, or see https://allocait.app/support. Agents connected through this plugin can also contact the team directly with the `send_feedback` tool: `topic: tools` for a problem with the tools, `topic: account` for help with the user's account (this needs a connection with Read and change).

## Releases

This repository is published from Allocait's private source. Two refs matter:

- **`main`** is what `/plugin marketplace add allocait-app/claude-plugin` installs in Claude Code. It carries `.claude-plugin/marketplace.json`.
- **`directory`** is the ref the Claude plugin directory follows. When submitting at claude.ai/directory/manage, enter this repository, leave the plugin path empty, and set the branch or tag to `directory`, not `main`. It holds the same files without the marketplace file, so `.claude-plugin/` contains only `plugin.json`. A commit lands there only on a deliberate release, because the directory scans every new commit on the ref it follows and later passing versions can go live on their own. You may follow a tag instead: a tag stays on its commit until it is moved.

## License

The MIT license in this folder covers only this plugin's own files (the manifests, skills, and eval cases). It does not extend to Allocait's application source, which lives in a separate, private repository.
