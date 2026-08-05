# Tradernet MCP for Claude Desktop

**Connect Claude Desktop to the Tradernet API over [Model Context Protocol](https://modelcontextprotocol.io/).**

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Version](https://img.shields.io/badge/version-1.0.0-green.svg)](claude_desktop_config.json)
[![MCP](https://img.shields.io/badge/MCP-Streamable%20HTTP-51AF3D.svg)](https://modelcontextprotocol.io/)
[![Platform](https://img.shields.io/badge/platform-Claude%20Desktop-D97757.svg)](https://claude.ai/download)
[![Node.js](https://img.shields.io/badge/bridge-mcp--remote-339933.svg)](https://www.npmjs.com/package/mcp-remote)
[![GitHub stars](https://img.shields.io/github/stars/tradernet-api/tradernet-mcp-claude?style=social)](https://github.com/tradernet-api/tradernet-mcp-claude/stargazers)

[Landing](https://mcp.tradernet.com/) ·
[API docs](https://tradernet.com/tradernet-api/mcp) ·
[Get API key](https://tradernet.com/tradernet-api/auth-api) ·
[Report issue](https://github.com/tradernet-api/tradernet-mcp-claude/issues)

---

> **Warning:** Every call runs against a **real Tradernet account** — real data, orders and money. Use a dedicated test account, not your production one.

## Three steps

If you opened this page yourself (GitHub / GitHub Pages) — do only this:

1. **Log in** to your Tradernet account:  
   [tradernet.com/login](https://tradernet.com/login)
2. **Create an API key** (`apiSecret` is shown only once):  
   [tradernet.com/tradernet-api/auth-api](https://tradernet.com/tradernet-api/auth-api)
3. **Feed this page to your AI** — open [Claude Desktop](https://claude.ai/download) and send this URL (or paste the whole page):  
   `https://github.com/tradernet-api/tradernet-mcp-claude`  
   Ask it to connect Tradernet MCP using this guide.

That is enough. The assistant will help merge config, env and smoke checks.
Details below are for the assistant (or for you if you prefer to do it by hand).

## Connect in one prompt (for the AI)

Use this if the user already completed **Three steps** (logged in + has an API key)
and asked you to connect Tradernet MCP from this page.

The product landing may mention **Claude Code**; this repository documents
**Claude Desktop** via the `mcp-remote` stdio bridge. Do not rename or treat
this as a full Claude Code package unless a separate working Claude Code
config is added.

### Credentials file

Save keys locally (**never paste secrets into chat**). The human edits the file:

```bash
mkdir -p ~/.config/tn-mcp && chmod 700 ~/.config/tn-mcp
cat > ~/.config/tn-mcp/credentials.env <<'EOF'
export TN_API_KEY="your-apiKey"
export TN_API_SECRET="your-apiSecret"
# optional — SID / login tools:
# export TN_LOGIN="user@example.com"
# export TN_PASSWORD="your-password"
EOF
chmod 600 ~/.config/tn-mcp/credentials.env
```

### Load env and start Claude Desktop from that shell

Dock / Start Menu launches often **do not** inherit shell env. `${TN_*}`
placeholders are read when Claude Desktop (and `mcp-remote`) start:

```bash
source ~/.config/tn-mcp/credentials.env
[ -f ~/.config/tn-mcp/session.env ] && source ~/.config/tn-mcp/session.env
# fully quit Claude Desktop, then relaunch from this environment
```

Optional persistent load in `~/.zshrc` / `~/.bashrc`:

```bash
[ -f ~/.config/tn-mcp/credentials.env ] && source ~/.config/tn-mcp/credentials.env
[ -f ~/.config/tn-mcp/session.env ] && source ~/.config/tn-mcp/session.env
```

### Agent checklist

Paste into Claude Desktop if the user did not already feed this whole page:

```
Connect Tradernet MCP using https://github.com/tradernet-api/tradernet-mcp-claude

1) Do not print or request secrets in chat. Confirm ~/.config/tn-mcp/credentials.env
   exists with TN_API_KEY and TN_API_SECRET. If missing — create placeholders and stop;
   tell me to fill values myself.

2) Merge the TN server from this repo's claude_desktop_config.json into Claude Desktop
   config (Settings → Developer → Edit Config) without removing other servers.
   Use mcp-remote → https://tradernet.com/mcp/tn with headers from env TN_*.

   Paths:
   - macOS: ~/Library/Application Support/Claude/claude_desktop_config.json
   - Windows: %APPDATA%\Claude\claude_desktop_config.json
   - Linux: ~/.config/Claude/claude_desktop_config.json

3) Ask me to fully quit Claude Desktop and relaunch from a terminal where
   credentials.env (and session.env if present) are sourced.
   Dock/Start Menu launch is not enough if env was missing.

4) Verify:
   - reference_get_market_status
   - quotes_get with { "tickers": ["AAPL.US"] }
   - portfolio_get
   Report which step failed.
```

Done when those three calls succeed. You get ~**61 tools**: quotes, portfolio,
orders, news, alerts, watchlists, analytics, tariffs/plans, reports and more.

Same public endpoint for every client: `https://tradernet.com/mcp/tn`.

## Why this is a config (not a Cursor-style plugin)

Claude Desktop has **no** rules / commands / hooks / write-guard like Cursor.
This repository ships:

| File | Purpose |
|------|---------|
| `claude_desktop_config.json` | MCP server `TN` via the `mcp-remote` stdio bridge |
| `skills/tn-mcp-refresh-sid/` | optional Agent Skill: runbook to refresh the SID |

### Why `mcp-remote`

Tradernet MCP is a remote **Streamable HTTP** endpoint authenticated with
static headers (`X-TN-API-Key`, `X-TN-API-Secret`, `X-TN-SID`). Claude Desktop
launches **stdio** servers, so we bridge with
[`mcp-remote`](https://www.npmjs.com/package/mcp-remote) and inject headers
from environment variables (`${TN_*}` placeholders — no secrets in the file).

## Manual install

### Requirements

1. A Tradernet account with API access.
2. An API key pair — [tradernet.com/tradernet-api/auth-api](https://tradernet.com/tradernet-api/auth-api)
   (`apiSecret` is shown only once).
3. [Node.js](https://nodejs.org/) (for `npx mcp-remote`).
4. [Claude Desktop](https://claude.ai/download).

### Merge the MCP config

1. Clone this repo or copy `claude_desktop_config.json`.
2. Merge the `TN` entry into Claude Desktop's config
   (**Settings → Developer → Edit Config**) without removing other servers:
   - macOS: `~/Library/Application Support/Claude/claude_desktop_config.json`
   - Windows: `%APPDATA%\Claude\claude_desktop_config.json`
   - Linux: `~/.config/Claude/claude_desktop_config.json`
3. Export secrets (see credentials above).
4. Fully quit and relaunch Claude Desktop from a shell where env is sourced.
5. In a new chat the `TN` server and its tools should be available.

### Optional: install the Skill

Add `skills/tn-mcp-refresh-sid/` as an Agent Skill
(**Settings → Capabilities → Skills**) so the assistant knows how to
re-authenticate when the SID expires.

## Auth notes

| Channel | Headers | Typical tools |
|---------|---------|---------------|
| **HMAC** | `X-TN-API-Key` + `X-TN-API-Secret` | `quotes_get`, `portfolio_get`, orders, reports |
| **SID** | `X-TN-SID` (after `auth_by_login`) | `get_sid_info` and other session tools |

- `portfolio_get` works **only** via HMAC.
- Claude Desktop / `mcp-remote` read `${TN_*}` when the process **starts**.
  After changing env or SID: `source` the files, **fully quit** Claude Desktop,
  relaunch from that shell. Dock / Start Menu relaunch is not enough to pick up
  new env.
- Optional SID cache: `~/.config/tn-mcp/session.env` (prefer this over mixing
  SID into `credentials.env`; see skill `tn-mcp-refresh-sid`).
- `auth_by_login` MCP args are snake_case (`remember_me: 1`). Tool args are
  **not** auto-expanded from `${env:...}` — never paste passwords into chat.
- Live orders may also need `orders_open_security_session` +
  `orders_confirm_security_session` before `orders_put`.

### Refreshing the SID

Session tools need a valid `X-TN-SID`. It expires (~14 days) and then returns
`401 Invalid credentials`. Prefer the skill runbook; short version:

1. Call `auth_by_login` with `login` / `password` and `remember_me: 1`
   (values from the user's env — never solicit the password in chat).
2. Write the returned SID to `~/.config/tn-mcp/session.env` (mode 600):

   ```bash
   mkdir -p ~/.config/tn-mcp && chmod 700 ~/.config/tn-mcp
   umask 077
   printf 'export TN_SID="%s"\n' "<SID>" > ~/.config/tn-mcp/session.env
   chmod 600 ~/.config/tn-mcp/session.env
   ```

3. `source` credentials + session files, fully quit Claude Desktop, relaunch
   from that shell.

HMAC tools (`quotes_get`, `portfolio_get`, …) do not need a SID.

## Security

Claude Desktop has **no write-guard hook**. Confirm with the user before any
mutating or auth tool, including:

- `orders_put`, `orders_delete`, `orders_set_stop_loss`
- `orders_open_security_session`, `orders_confirm_security_session`
- `tariff_select`
- watchlist mutations (`quotes_add_list`, `quotes_delete_list`, and
  add/delete/update/select ticker tools)
- `alerts_add`, `alerts_delete`
- `reports_upload_file`
- `switch_to_connected_user`
- `auth_by_login`, `auth_by_sms`, `get_auth_sms`

Also:

- Secrets live only in env / local files — never in the config, repo or chat.
- Quote/market reads are fine for smoke checks. Portfolio and reports can
  contain PII — treat responses carefully. `reports_upload_file` is a **write**,
  not a read-only reports call.

## Data notes

- Numbers often come back as strings (`"p": "10.08600000"`) — cast before math.
- `quotes_get`: `tickers` is a **list**, even for one symbol
  (`{ "tickers": ["AAPL.US"] }`).
- `quotes_get_hloc`: ticker is a **string** (`"AAPL.US"`); candle order is
  `[HIGH, LOW, OPEN, CLOSE]`.
- Large responses — use pagination and filters (`max_count`, date ranges).

## Related integrations

| Client | Repository | What you get |
|--------|------------|--------------|
| **Cursor** | [tradernet-mcp-cursor](https://github.com/tradernet-api/tradernet-mcp-cursor) | One-prompt MCP + full plugin (rules, commands, hook, skill) |
| **Claude Desktop** (this repo) | [tradernet-mcp-claude](https://github.com/tradernet-api/tradernet-mcp-claude) | `mcp-remote` config + optional SID skill |
| **Codex** | [tradernet-mcp-codex](https://github.com/tradernet-api/tradernet-mcp-codex) | Codex plugin with MCP config and safety skill |

Product overview: [mcp.tradernet.com](https://mcp.tradernet.com/).

The old GitHub slug `tradernet-api/tradernet-mcp` redirects to the Cursor repo.

## License

[MIT](LICENSE) © Tradernet
