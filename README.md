# Tradernet MCP — Claude Desktop

Connect Claude Desktop to the Tradernet API over [Model Context Protocol](https://modelcontextprotocol.io/).
Once connected, the assistant gets ~61 tools: quotes, portfolio, orders,
tariffs, reports and more.

> Every call runs against a **real Tradernet account** — real data, orders and
> money. Use a dedicated test account, not your production one.

## Why this is a config, not a "plugin"

Claude Desktop has no rules / commands / hooks system like Cursor. The only thing
you wire up is the **MCP server** itself, plus an optional Agent Skill. This repo
ships:

| File | Purpose |
|------|---------|
| `claude_desktop_config.json` | MCP server `TN` via the `mcp-remote` bridge |
| `skills/tn-mcp-refresh-sid/` | optional Agent Skill: runbook to refresh the SID |
| `assets/logo.svg` | logo |

## Why `mcp-remote`

Tradernet MCP is a remote **Streamable HTTP** endpoint that authenticates with
static headers (`X-TN-API-Key`, `X-TN-API-Secret`, `X-TN-SID`). Claude Desktop's
config launches **stdio** servers, so we bridge to the remote endpoint with
[`mcp-remote`](https://www.npmjs.com/package/mcp-remote) and inject the headers
from environment variables.

## Requirements

1. A Tradernet account with API access.
2. An API key pair — [tradernet.com/tradernet-api/auth-api](https://tradernet.com/tradernet-api/auth-api)
   (`apiSecret` is shown only once).
3. Node.js (for `npx mcp-remote`).
4. Claude Desktop.

## Secrets (env only, never in the config)

The config ships only `${...}` placeholders. Keep the actual values in your
environment. Example `~/.config/tn-mcp/credentials.env` (chmod 600):

```bash
export TN_API_KEY="your-apiKey"
export TN_API_SECRET="your-apiSecret"
export TN_SID=""                       # optional, for session tools only
export TN_LOGIN="user@example.com"     # optional, for auth_by_login
export TN_PASSWORD="your-password"     # optional
```

Source it from your shell profile so the variables exist when the app launches:

```bash
[ -f ~/.config/tn-mcp/credentials.env ] && source ~/.config/tn-mcp/credentials.env
```

> Launching Claude Desktop from the Dock/Start menu may not inherit your shell
> environment. Start it from a terminal after `source`, or export the variables
> in a login profile, otherwise `${TN_API_KEY}` resolves to empty.

## Install

1. Merge the `TN` entry from `claude_desktop_config.json` into Claude Desktop's
   config (Settings → Developer → Edit Config):
   - macOS: `~/Library/Application Support/Claude/claude_desktop_config.json`
   - Windows: `%APPDATA%\Claude\claude_desktop_config.json`
   - Linux: `~/.config/Claude/claude_desktop_config.json`
2. Make sure the secrets are exported (see above).
3. Fully quit and relaunch Claude Desktop.
4. In a new chat the `TN` server and its tools should be available.

### Optional: install the Skill

Add `skills/tn-mcp-refresh-sid/` as an Agent Skill (Settings → Capabilities →
Skills) so the assistant knows how to re-authenticate when the SID expires.

## Quick start

1. Ask: "check the Tradernet connection" → the assistant calls
   `reference_get_market_status`, then `quotes_get` for `AAPL.US`, then
   `portfolio_get`.
2. If a session tool returns `401` → refresh the SID (see below).

## Refreshing the SID

Session tools (`get_sid_info`, ...) need a valid `X-TN-SID`. It expires (~14 days)
and then returns `401 Invalid credentials`. To refresh:

1. Call `auth_by_login` with `login` / `password` and `remember_me: 1`.
2. Put the returned SID into `TN_SID` in `credentials.env`.
3. `source` the file and **fully relaunch** Claude Desktop (the header value is
   read at startup).

HMAC tools (`quotes_get`, `portfolio_get`, ...) do not need a SID. See the
`tn-mcp-refresh-sid` skill for the full runbook.

## Security

- Secrets live only in environment variables — never in the config or chat.
- Claude Desktop has **no write-guard hook**. Be deliberate with write/trade
  tools — `orders_put`, `orders_delete`, `orders_set_stop_loss`, `tariff_select`,
  list/alert changes — they mutate a real account and real money. Confirm ticker,
  price, quantity and side before placing an order.
- Read-only tools (quotes, portfolio, reports) are safe to call.

## Data notes

- Numbers often come back as strings (`"p": "10.08600000"`) — cast before math.
- `quotes_get_hloc` candle order is `[HIGH, LOW, OPEN, CLOSE]`; the ticker is a
  string (`"AAPL.US"`), not an array.
- Large responses — use pagination and filters (`max_count`, date ranges).

## Related

- Cursor plugin: [github.com/tradernet-api/tradernet-mcp](https://github.com/tradernet-api/tradernet-mcp)
- API docs: [tradernet.com/tradernet-api](https://tradernet.com/tradernet-api)

## License

MIT.
