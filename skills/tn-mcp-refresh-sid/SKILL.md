---
name: tn-mcp-refresh-sid
description: >-
  Refresh the Tradernet MCP SID via auth_by_login and write it to
  ~/.config/tn-mcp/session.env. Use on 401 Invalid credentials for SID tools
  (get_sid_info, etc.), expired session, or when the user asks to refresh SID /
  re-authenticate to TN.
---

# tn-mcp-refresh-sid — refresh Tradernet MCP SID

Session tools (`get_sid_info`, …) need a valid `X-TN-SID`. SID lasts ~14 days;
after expiry you get `401 Invalid credentials`. This skill is the re-auth
runbook for Claude Desktop + `mcp-remote`.

## When to use

- `401 Invalid credentials` on a SID tool.
- User asks to refresh SID / re-auth to TN / session expired.
- Before a SID-only workflow if the session is stale.

## Do NOT

- Print password, login, or SID in chat, logs, or `echo`.
- Write SID into `claude_desktop_config.json` — only into
  `~/.config/tn-mcp/session.env`.
- Write SID under a project path (easy to commit by mistake).
- Refresh SID for HMAC-only tools (`portfolio_get`, `quotes_get`) — check the
  HMAC channel first.
- Solicit a password in chat — ask the user to edit `credentials.env`
  themselves if login vars are missing.

## Steps

1. Confirm `TN_LOGIN` / `TN_PASSWORD` exist in
   `~/.config/tn-mcp/credentials.env`. If missing, ask the user to edit that
   file themselves — never solicit a password in chat.

2. Call `auth_by_login`. MCP tool args must be real strings (they are **not**
   expanded from `${env:...}`). Read values from the user's environment in the
   shell without printing them, e.g.:

   ```bash
   # values stay in the shell; do not paste into chat
   : "${TN_LOGIN:?}" "${TN_PASSWORD:?}"
   ```

   Then invoke the tool with those values and snake_case `remember_me: 1`.
   Prefer a short confirmation from the user that they authorize a login call.

3. Take `SID` from the response and write **only**:

   ```bash
   mkdir -p ~/.config/tn-mcp && chmod 700 ~/.config/tn-mcp
   umask 077
   printf 'export TN_SID="%s"\n' "<SID>" > ~/.config/tn-mcp/session.env
   chmod 600 ~/.config/tn-mcp/session.env
   ```

   Do not also mirror SID into the project tree or into `credentials.env`
   unless the user explicitly prefers that.

4. Load env and **fully quit + relaunch** Claude Desktop from the same shell
   (`${TN_SID}` is read when `mcp-remote` starts; Dock / Start Menu relaunch
   will not pick up newly sourced env):

   ```bash
   source ~/.config/tn-mcp/credentials.env
   source ~/.config/tn-mcp/session.env
   # quit Claude Desktop completely, then relaunch from this shell
   ```

5. Verify: call `get_sid_info` — must return session data without `401`.

## If 401 persists

- Repeat steps 2–5 (SID may not have been loaded without a full relaunch).
- Confirm `TN_LOGIN` / `TN_PASSWORD` are current (password change invalidates
  old sessions).
- Confirm `X-TN-SID` is present in MCP headers after relaunch.
