# Geolocate Me Skill

A cross-platform agent skill for the [Geolocate Me](https://geolocateme.app) MCP server. Teaches your agent to answer location-aware questions — *"where am I?"*, *"where did I park?"*, *"how long was I at the office?"* — by calling the right tools and responding with human-readable places instead of raw coordinates.

Compatible with any host that implements the [agentskills.io](https://agentskills.io) open standard: Claude, Claude Code, Cursor, Goose, Hermes Agent, OpenHands, Letta, nanobot, and more.

## What it does

- Recognizes trigger phrases ("where am I", "where did I park", "reconstruct my day") and picks the right tool.
- Translates natural language time ranges ("yesterday at 3pm", "last week") into `from`/`to` parameters.
- Summarizes location history into human-readable segments (home → commute → office → home) rather than ping-by-ping dumps.
- Infers home / office, detects anomalies, estimates commute times — see `references/recipes.md`.

## Prerequisites

1. **The Geolocate Me app** installed on your phone: [iOS](https://apps.apple.com/us/app/geolocate-me/id6761958740) · [Android](https://play.google.com/store/apps/details?id=com.guleki.geolocateme)
2. **A Device ID + Bot Token** from the app (Settings → Copy All Credentials)
3. **A compatible MCP client** configured with the Geolocate Me server — see `references/setup.md`

## Install

### Recommended — via the `skills` CLI

One command, works across every agent skills host:

```bash
npx skills add linville-charlie/geolocate-me-skill
```

The CLI auto-detects which of your installed agents (Claude Code, Cursor, Goose, Codex, Hermes Agent, and 40+ others) should receive the skill, copies it into the right directory, and you're done. Pass `-g` for user-level install, or `-a claude-code` to target a specific agent. See [skills.sh](https://skills.sh) and [github.com/vercel-labs/skills](https://github.com/vercel-labs/skills) for more options.

### Manual install

Drop the `geolocate-me/` folder into your host's skills location:

- **Claude Code**: project `.claude/skills/geolocate-me/` or user-level `~/.claude/skills/geolocate-me/`
- **Hermes Agent**: `~/.hermes/skills/location/geolocate-me/`
- **Goose**: `~/.config/goose/skills/geolocate-me/`
- **Others**: see your host's skill directory convention

Or clone directly:

```bash
# Claude Code / user level
mkdir -p ~/.claude/skills && git clone https://github.com/linville-charlie/geolocate-me-skill ~/.claude/skills/geolocate-me
```

### Auto-discovery from geolocateme.app

If your host supports `/.well-known/skills/index.json` discovery, point it at `https://geolocateme.app` and the skill will appear in its install list.

## Verify

After installing the skill and configuring the MCP server, ask:

> "Where am I right now?"

You should get a street-address response. If not, check `references/setup.md` troubleshooting.

## Privacy

- The skill itself is static markdown — it contains no secrets.
- Your Device ID and Bot Token authorize read access to your phone's location history. Keep them private.
- Full privacy policy: <https://geolocateme.app/privacy>

## Contributing

Issues and PRs welcome. Keep changes in line with the [agentskills.io](https://agentskills.io/specification) spec.

## License

MIT — see `LICENSE`.

---

Built by [Guleki](https://guleki.com) · Product site: <https://geolocateme.app> · Spec: <https://agentskills.io>
