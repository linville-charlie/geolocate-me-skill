# Setup per host

The Geolocate Me MCP server is a remote HTTP (streamable-http) endpoint at:

```
https://geolocateme.guleki.com/mcp
```

It uses OAuth 2.0 with PKCE. Hosts that implement the full OAuth flow can connect with just the URL — the user is redirected to an authorize page where they paste their Device ID and Bot Token from the Geolocate Me app. Hosts that only support Bearer-token or stdio configs need a slightly different setup, covered below.

## Getting credentials

In the Geolocate Me app on your phone:

1. Open the dashboard.
2. Tap the gear icon (Settings).
3. Tap **Copy All Credentials**.

You now have a Device ID (UUID) and a Bot Token (64-character hex) on your clipboard.

---

## Native OAuth (recommended)

These hosts handle the OAuth handshake automatically. Paste the URL, complete the redirect in your browser, and you're done.

### Claude.ai

1. Settings → Connectors → Add custom connector
2. Name: `Geolocate Me`
3. Server URL: `https://geolocateme.guleki.com/mcp`
4. Click Connect → complete OAuth in the popup → paste Device ID + Bot Token on the authorize page

### Claude Desktop

1. Settings → Developer → Add custom connector
2. URL: `https://geolocateme.guleki.com/mcp`
3. Complete OAuth when prompted

Or add to `~/Library/Application Support/Claude/claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "geolocate-me": {
      "type": "http",
      "url": "https://geolocateme.guleki.com/mcp"
    }
  }
}
```

### Claude Code

```bash
claude mcp add --transport http geolocate-me https://geolocateme.guleki.com/mcp
```

If the CLI syntax has changed, add the server manually to `.mcp.json` using the same object format as Cursor below. Check [code.claude.com/docs](https://code.claude.com/docs) for the current command.

### Cursor / Windsurf / Zed

Add to `~/.cursor/mcp.json` (or the equivalent config for your host):

```json
{
  "mcpServers": {
    "geolocate-me": {
      "type": "http",
      "url": "https://geolocateme.guleki.com/mcp"
    }
  }
}
```

### Goose

Goose supports remote MCP in recent versions. The cleanest path is to add the server via Goose's extensions UI. If you prefer config, the `extensions` block in `~/.config/goose/config.yaml` accepts a remote HTTP entry — the exact key names have changed between Goose releases, so check [Goose's current docs](https://block.github.io/goose/docs/) before copying any example.

Illustrative (verify current keys):

```yaml
extensions:
  geolocate_me:
    type: http
    url: https://geolocateme.guleki.com/mcp
    enabled: true
```

---

## Bearer-token hosts

Some hosts (Hermes Agent, LM Studio, and others) expect a pre-issued Bearer token and don't run an OAuth flow. For these, use the **legacy header auth** path the server still supports.

### Hermes Agent (Nous Research)

Edit `~/.hermes/config.yaml`:

```yaml
mcp_servers:
  geolocate_me:
    url: "https://geolocateme.guleki.com/mcp?deviceId=YOUR_DEVICE_ID&botToken=YOUR_BOT_TOKEN"
```

Replace `YOUR_DEVICE_ID` and `YOUR_BOT_TOKEN` with the values from the app. Then:

```bash
hermes tools         # or /reload-mcp inside a session
```

Tools will appear as `mcp_geolocate_me_get_current_location` and `mcp_geolocate_me_get_location_history`.

**Security note:** The config file contains your bot token. Keep `~/.hermes/config.yaml` at mode 600 and don't commit it.

### LM Studio

LM Studio's MCP support has moved between releases. The general pattern: find the MCP/Tools settings panel, add a remote/streamable-http server, and set the URL to:

```
https://geolocateme.guleki.com/mcp?deviceId=YOUR_DEVICE_ID&botToken=YOUR_BOT_TOKEN
```

If your build of LM Studio supports an `Authorization` header field instead, use the Bearer-token flow in the next section.

### Generic "URL + headers" hosts

If the host lets you set an `Authorization` header instead of query params, mint a 30-day token using the OAuth flow. We support **Dynamic Client Registration** (RFC 7591) so you don't need any pre-shared credentials.

```bash
# 1. Register a client — returns fresh client_id and client_secret for YOUR use.
curl -s -X POST https://geolocateme.guleki.com/oauth/register \
  -H "Content-Type: application/json" \
  -d '{"client_name": "my-agent"}'

# Response (save CLIENT_ID and CLIENT_SECRET somewhere):
# {"client_id":"...","client_secret":"...",...}

# 2. Visit this URL in a browser (substitute CLIENT_ID):
# https://geolocateme.guleki.com/oauth/authorize?client_id=CLIENT_ID&response_type=code&redirect_uri=urn:ietf:wg:oauth:2.0:oob&state=abc

# 3. Paste your Device ID + Bot Token on the authorize page, submit. You'll land on a page showing an auth code in the redirect URL.

# 4. Exchange the code for a 30-day access token (substitute CLIENT_ID, CLIENT_SECRET, AUTH_CODE):
curl -X POST https://geolocateme.guleki.com/oauth/token \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=authorization_code" \
  -d "code=AUTH_CODE" \
  -d "client_id=CLIENT_ID" \
  -d "client_secret=CLIENT_SECRET"
```

Then plug the returned `access_token` into your host config:

```yaml
mcp_servers:
  geolocate_me:
    url: "https://geolocateme.guleki.com/mcp"
    headers:
      Authorization: "Bearer YOUR_30_DAY_TOKEN"
```

When the 30-day token expires, repeat steps 2-4 (you can reuse the same CLIENT_ID from step 1).

---

## Tool filtering (optional)

Both tools are read-only, so most users enable both. If your host supports `tools.include` / `tools.exclude`, you can whitelist:

```yaml
mcp_servers:
  geolocate_me:
    url: https://geolocateme.guleki.com/mcp
    tools:
      include: [get_current_location, get_location_history]
```

## Verifying it works

After setup, ask your agent:

> "Where am I right now?"

You should get a street-address response within a few seconds. If you get "No location data available yet", open the Geolocate Me app and confirm tracking is on.

## Troubleshooting

| Symptom | Likely cause |
|---|---|
| 401 Unauthorized | Bot Token wrong length or typo. Should be exactly 64 hex characters. |
| "No location data available yet" | Tracking is off in the app, or the phone hasn't pinged since install. |
| Pings are stale (hours old) | Phone is offline, Battery Low Power Mode on iOS, or tracking was disabled. |
| 400 Invalid client_id (Bearer flow) | OAuth client credentials changed. See [geolocateme.app/docs/mcp](https://geolocateme.app/docs/mcp) for current values. |
