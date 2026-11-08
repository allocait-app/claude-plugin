# Allocait

Allocait is a rules engine for your investment portfolio, independent of any brokerage. You write your rules once: an allocation band for each asset class and, if you like, a size range for each position. Allocait keeps your holdings, cash, rules and research, and checks your holdings and proposed changes against those rules, for you and your AI agents. It never places a trade or moves money, and never recommends buying, selling, or holding anything. Allocait output states facts about the user's own rules. It is not advice or an approval.

## Use it

Ask Claude to check a portfolio against its rules, check a change before you place it elsewhere, work out the amounts that close a gap, size a position inside your range, record a trade made outside Allocait, change a rule or write a POLICY.md rule file, keep a watchlist, fill in a playbook, or add a setup file you were sent, after a preview you approve, and undo it later. Claude calls the Allocait MCP connector's tools, such as `get_portfolio_snapshot`, `check_trades`, `save_position` and `save_watchlist`. The `allocait-check` skill covers the change check: it reports which of your rules a change puts outside range or leaves further outside, with the time your positions were last updated. A failed call is no result, and Allocait cannot approve, stop or place an order. A write removes a holding, a rule or a list member only when the call names it. Undoing a setup file import removes only what that import added that you haven't changed or used. Signed in through this plugin, Claude can show where your portfolio stands with no prices needed: `standing` gives percents against your ranges, read from Allocait's last stored quote, with its time. Dollar values and change checks use prices you give, or that Claude looks up from a source you trust; Allocait hands out no price of its own.

### Example prompts

Each example uses fictional symbols and placeholder numbers.

- "Where does my Main Portfolio stand against my rules?" Claude reads `standing` and reports each asset class and each holding outside its range, with the time of the quote it read. No prices needed.
- "Before I place it elsewhere: what does adding 40 shares of ALFA at 181.5 do to my rules? BRAV is 94.3 and CHAR is 62.7." Claude calls `check_trades` and reports which rules the change puts outside range or leaves further outside. It gives no yes or no.
- "I bought 10 more shares of BRAV at 94.3 through my broker. Record it." Claude reads the current BRAV holding and saves the new total with `save_position`. This records the change in Allocait only. It moves no money and places no order.

## Set up

This plugin connects by signing in. It stores no API key.

1. **Claude Code.** Install the plugin, run `/mcp`, choose Allocait, then select Authenticate and sign in to Allocait in your browser.
2. **claude.ai and Cowork.** Open the plugin's Connectors tab, select Connect for Allocait, and sign in. On a Team or Enterprise organization, an Owner adds the Allocait connector first, then members connect with their own accounts.
3. **On the consent screen,** choose Read only or Read and change. Read only lets Claude read and check. Read and change also lets it save to your Allocait account.

You need an Allocait account with an active plan. The sign-in screen shows which access your plan offers.

The [connection guide](https://docs.allocait.app/connect-an-agent) covers other AI clients.

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
- **`directory`** is the ref the Claude plugin directory follows. When submitting at claude.ai/directory/manage, enter this repository, leave the plugin path empty, and set the branch or tag to `directory`, not `main`. It holds the same files without the marketplace file or `.gitignore`, so `.claude-plugin/` contains only `plugin.json`. A commit lands there only on a deliberate release, because the directory scans every new commit on the ref it follows and later passing versions can go live on their own. You may follow a tag instead: a tag stays on its commit until it is moved.

## License

The MIT license in this folder covers only this plugin's own files (the manifests, skills, and eval cases). It does not extend to Allocait's application source, which lives in a separate, private repository.
