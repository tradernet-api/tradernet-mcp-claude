---
name: tn-mcp-refresh-sid
description: Refresh the Tradernet MCP session (SID) via auth_by_login and update the TN_SID environment variable. Use on "401 Invalid credentials" from session tools (get_sid_info, ...), on an expired session, or when asked to "refresh SID / re-login to TN".
---

# tn-mcp-refresh-sid — refresh the Tradernet MCP SID

Session tools (`get_sid_info` and other session operations) require a valid
`X-TN-SID`. The SID lives ~14 days and then expires, returning
`401 Invalid credentials`. This skill is the re-authentication runbook.

## When to use

- `401 Invalid credentials` on a session tool.
- The user asks to "refresh SID", "re-login to TN", "session expired".
- Before a batch of session calls if the SID has not been refreshed for a while.

## What NOT to do

- Never print the password or login into chat or via `echo`.
- Never hardcode the SID into `claude_desktop_config.json` — keep it in the
  `TN_SID` environment variable only.
- HMAC tools (`portfolio_get`, `quotes_get`) do not need a SID — re-auth is not
  required for them; check the channel instead.

## Steps

1. Make sure `TN_LOGIN` / `TN_PASSWORD` are set in the user's environment
   (e.g. `~/.config/tn-mcp/credentials.env`). If not, ask the user to configure
   them — never request the password in plain text.

2. Call the `auth_by_login` tool with arguments taken from the user's env
   (do not hardcode values):

   ```json
   { "login": "<TN_LOGIN>", "password": "<TN_PASSWORD>", "remember_me": 1 }
   ```

3. Take the `SID` from the response and write it into `TN_SID` in the user's
   secrets file (single line, overwritten on each login):

   ```bash
   # in ~/.config/tn-mcp/credentials.env
   export TN_SID="<SID>"
   ```

4. Reload the environment and fully relaunch Claude Desktop. The header value
   `${TN_SID}` is read when the MCP server starts, so saving the file is not
   enough — the app must be restarted:

   ```bash
   source ~/.config/tn-mcp/credentials.env
   # then quit Claude Desktop completely and start it again
   ```

5. Verify: call `get_sid_info` — it should return session data without `401`.

## If you still get 401

- Repeat steps 2–4 (the SID may not have been picked up without a full restart).
- On the public endpoint, if `auth_by_login` returns `code: 30`, try the
  internal endpoint if the user has access to it.
- Check that `TN_LOGIN` / `TN_PASSWORD` are current (a password change
  invalidates old sessions).
