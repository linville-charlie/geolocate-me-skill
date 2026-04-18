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

### Hosts with a `skills` directory

Drop the `geolocate-me/` folder into your skills location. Examples:

- **Hermes Agent**: `~/.hermes/skills/location/geolocate-me/`
- **Claude Code**: project `.claude/skills/geolocate-me/` or user-level `~/.claude/skills/geolocate-me/`
- **Goose**: `~/.config/goose/skills/geolocate-me/`
- **Others**: see your host's skill directory convention

### Install from GitHub

```bash
# Claude Code / user level
mkdir -p ~/.claude/skills && git clone https://github.com/linville-charlie/geolocate-me-skill ~/.claude/skills/geolocate-me

# Hermes
mkdir -p ~/.hermes/skills/location && git clone https://github.com/linville-charlie/geolocate-me-skill ~/.hermes/skills/location/geolocate-me
```

### Install from the discovery directory

If your host pulls from [skills.sh](https://skills.sh) or `/.well-known/skills/index.json`:

- Look up `geolocate-me` in the directory
- Install via your host's UI

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
