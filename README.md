<div align="center">

<img src="assets/logo.svg" alt="Tradernet MCP" width="280">

# Tradernet MCP for Claude Desktop

**Connect Claude Desktop to the Tradernet API over [Model Context Protocol](https://modelcontextprotocol.io/).**

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Version](https://img.shields.io/badge/version-1.0.0-green.svg)](claude_desktop_config.json)
[![MCP](https://img.shields.io/badge/MCP-Streamable%20HTTP-51AF3D.svg)](https://modelcontextprotocol.io/)
[![Platform](https://img.shields.io/badge/platform-Claude%20Desktop-D97757.svg)](https://claude.ai/download)
[![Node.js](https://img.shields.io/badge/bridge-mcp--remote-339933.svg)](https://www.npmjs.com/package/mcp-remote)
[![GitHub stars](https://img.shields.io/github/stars/tradernet-api/tradernet-mcp-claude?style=social)](https://github.com/tradernet-api/tradernet-mcp-claude/stargazers)

[Tradernet API docs](https://tradernet.com/tradernet-api/mcp) ·
[Report issue](https://github.com/tradernet-api/tradernet-mcp-claude/issues)

</div>

---

> **Warning:** Every call runs against a **real Tradernet account** — real data, orders and money. Use a dedicated test account, not your production one.

## Overview

Once connected, Claude gets ~**61 tools**: quotes, portfolio, orders, tariffs, reports and more.

Claude Desktop has no rules / commands / hooks system like Cursor. This repository ships the **MCP connection config** (via the `mcp-remote` bridge) and an optional **Agent Skill** for SID refresh.

## Features

| Capability | Description |
|------------|-------------|
| **MCP bridge** | `claude_desktop_config.json` — stdio `mcp-remote` → Streamable HTTP endpoint |
| **Auth headers** | `X-TN-API-Key`, `X-TN-API-Secret`, `X-TN-SID` from environment variables |
| **SID skill** | `tn-mcp-refresh-sid` — runbook when session tools return `401` |
| **No secrets in repo** | Config uses `${TN_*}` placeholders only |

## Table of contents

- [Why this is a config, not a "plugin"](#why-this-is-a-config-not-a-plugin)
- [Why mcp-remote](#why-mcp-remote)
- [Requirements](#requirements)
- [Secrets](#secrets-env-only-never-in-the-config)
- [Install](#install)
- [Quick start](#quick-start)
- [Refreshing the SID](#refreshing-the-sid)
- [Security](#security)
- [Data notes](#data-notes)
- [Related integrations](#related-integrations)
- [License](#license)

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
3. [Node.js](https://nodejs.org/) (for `npx mcp-remote`).
4. [Claude Desktop](https://claude.ai/download).

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

1. Clone this repo or copy `claude_desktop_config.json`.
2. Merge the `TN` entry into Claude Desktop's config (**Settings → Developer → Edit Config**):
   - macOS: `~/Library/Application Support/Claude/claude_desktop_config.json`
   - Windows: `%APPDATA%\Claude\claude_desktop_config.json`
   - Linux: `~/.config/Claude/claude_desktop_config.json`
3. Make sure the secrets are exported (see above).
4. Fully quit and relaunch Claude Desktop.
5. In a new chat the `TN` server and its tools should be available.

### Optional: install the Skill

Add `skills/tn-mcp-refresh-sid/` as an Agent Skill (**Settings → Capabilities → Skills**)
so the assistant knows how to re-authenticate when the SID expires.

## Quick start

1. Ask: *"check the Tradernet connection"* → the assistant calls
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

## Related integrations

| Client | Repository | What you get |
|--------|------------|--------------|
| **Cursor** | [tradernet-mcp](https://github.com/tradernet-api/tradernet-mcp) | Full plugin: rules, commands, hook, skill |
| **Claude Desktop** (this repo) | [tradernet-mcp-claude](https://github.com/tradernet-api/tradernet-mcp-claude) | `mcp-remote` config + optional SID skill |
| **Codex** | [tradernet-mcp-codex](https://github.com/tradernet-api/tradernet-mcp-codex) | Codex plugin with MCP config and safety skill |

## License

[MIT](LICENSE) © Tradernet
