![Tapify logo](assets/logo.svg)

# Tapify for Claude

Tapify for Claude connects Claude to authorized Tapify business data through the
hosted Tapify MCP service. Bar owners and approved staff can ask about settled
dispensing sales, keg inventory, product prices, stations, taps, sessions,
digital menus, cards, customers, and operators. The connector is read-only and
uses the permissions selected during Tapify OAuth authorization.

## Install

Until the plugin is published in Claude's directory, install the reviewed bundle
from its source folder or ZIP:

1. In Claude.ai or Cowork, open **Customize → Plugins → Add → Upload plugin**.
2. Select a ZIP whose root is this plugin folder, or whose contents begin with
   `.claude-plugin/plugin.json`.
3. Open the plugin's **Connectors** tab and connect Tapify.
4. Sign in through Tapify OAuth, choose the intended business and stations, and
   approve only the read capabilities needed for your work.

On Team and Enterprise plans, an organization Owner adds the connector and each
member connects with their own Tapify account. After a public directory release,
users can install Tapify from **Customize → Plugins** and follow the same OAuth
connection flow.

To load the source bundle in Claude Code for local testing, run this from the
folder containing the plugin:

```sh
claude --plugin-dir ./tapify-claude-plugin
```

Then use `/mcp` to inspect the Tapify connection and complete OAuth when prompted.
No API key, access token, or custom header belongs in this plugin's files.

## What it can read

The connector currently exposes 31 read-only tools when all corresponding
capabilities are available and explicitly approved. Actual availability depends
on the user's Tapify membership, selected business and stations, current product
permissions, and OAuth consent.

| Permission | Data it permits |
| --- | --- |
| `tapify.stations.read` | Selected stations, location alias, and current authorized context |
| `tapify.taps.read` | Taps attached to selected stations |
| `tapify.sessions.read` | Bounded dispensing sessions at selected stations |
| `tapify.inventory.read` | Kegs and products currently attached to selected stations |
| `tapify.inventory.business.read` | Business-wide keg inventory, keg history, and product catalog |
| `tapify.cards.read` | Business-associated cards, balances, and card ledger entries |
| `tapify.users.read` | Related customer records, with masking preserved |
| `tapify.operators.read` | Operators assigned to the selected business |
| `tapify.analytics.read` | Bounded settled-charge summaries, day/hour buckets, and trends |
| `tapify.menu.read` | The active business-wide digital menu and its active contents |

See the [complete tool catalog](skills/tapify-read/references/tool-catalog.md) for
all tool names, coverage, units, pagination, ordering, and server limits.

Example requests:

- “What were yesterday's settled sales and poured volume for my selected stations?”
- “List all low-volume kegs in my authorized business inventory, including reserve stock.”
- “Show the active digital menu with item prices and availability.”

## Read-only boundaries

This plugin does not create, change, or delete Tapify records. It cannot add
credit, charge cards, change prices or menus, operate taps, control dispensing
hardware, or alter users and permissions. IDs supplied to tools are filters and
never grant access; Tapify re-evaluates membership, selected resources,
capabilities, API permissions, and masking on every request.

Results describe recorded API state rather than live device telemetry. Settled
wallet charges are not fiscal revenue, top-ups, refunds, or profit. Catalog prices
are RON bani per litre, menu prices are minor currency units per item, and volumes
are millilitres. List results use bounded cursor pagination and are not an atomic
snapshot. A complete list requires reading until `nextCursor` is null. Sales and
session windows have documented limits, and large records or pages can fail
instead of being silently truncated.

Customer, operator, and card fields may be masked. The plugin guidance tells
Claude to preserve that masking, avoid reconstructing personal data, and avoid
claiming that an incomplete or failed read is complete.

## Data handling and destinations

The bundle contains only Markdown, JSON, the license text, and the Tapify logo.
It has no hooks, scripts, executable dependencies, local server, background task,
analytics code, or independent data store.

When a user asks Claude to read Tapify data and has approved the required OAuth
capability, Claude sends the user-approved API request to the declared Tapify MCP
endpoint at `https://mcp.tapify.ro/mcp`. Request inputs can include resource IDs,
search terms, cursor values, date windows, and the authenticated authorization
context needed to enforce the user's access. Requested results return to Claude.
This bundle declares no other service destination and does not independently
store connector requests or results.

Tapify's hosted service may process and retain data as described in the
[Tapify Privacy Policy](https://tapify.ro/en/privacy/). This README does not
infer or add a retention period beyond that published policy. Claude and the
user's organization may separately process conversation and connector data under
their own settings and terms.

## Privacy and support

- Privacy: [Tapify Privacy Policy](https://tapify.ro/en/privacy/)
- Terms: [Tapify Terms and Conditions](https://tapify.ro/en/terms/)
- Support and privacy requests: [contact@tapify.ro](mailto:contact@tapify.ro)

Do not send access tokens, raw customer payloads, NFC data, or signing material
in support messages. Include only a request reference and a concise description
of the failed read.

## License and trademarks

The plugin's text and configuration guidance are licensed under MIT. This
license does not cover the hosted Tapify MCP/API services, their server
implementation, or any other Tapify software.

Tapify names, logos, and trademarks are excluded from the MIT grant. The bundled
logo is included only to identify this Tapify plugin; no broader trademark or
brand license is granted.
