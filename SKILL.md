---
name: geolocate-me
description: Answer any question about the user's real-world location or location history by calling the Geolocate Me MCP server. Use whenever the user asks where they are, where their phone is, where they parked, how long they spent at a place, when they left somewhere, what they did yesterday or last week, or to reconstruct a day, commute, or trip. Also activate for "am I at home", "am I at work", "how much time did I spend at the gym", or any location-aware scheduling question. Returns reverse-geocoded street addresses, not raw coordinates. Two read-only tools are exposed, get_current_location and get_location_history, sourced from the Geolocate Me iOS/Android app on the user's phone.
license: MIT
compatibility: Requires the Geolocate Me MCP server configured in the host (see references/setup.md) and the Geolocate Me app installed and tracking on the user's phone.
metadata:
  author: Guleki
  version: "1.0.1"
  website: https://geolocateme.app
  mcp_server: https://geolocateme.guleki.com/mcp
  tools: "get_current_location, get_location_history"
---

# Geolocate Me

This skill teaches you how to answer location-aware questions about the user by calling the Geolocate Me MCP server. The server exposes the user's phone GPS as read-only tools. Your job is to pick the right tool, pass the right time window, and respond with human-readable places — not raw coordinates.

## When to activate

Activate this skill when the user asks anything about where they are, where they've been, or how they spent time in physical space. Examples:

- "Where am I?" / "What's my current location?" / "Where's my phone?"
- "Where did I park?"
- "Where was I yesterday at 3pm?"
- "How long was I at the office today?"
- "What time did I leave home this morning?"
- "Reconstruct my day" / "Summarize where I went"
- "Have I been at the same place all day?"
- "Am I home yet?" / "Am I at work?"
- "Draft a travel log for last week"
- "How often do I go to the gym?"
- "What coffee shop did I stop at on Tuesday?"

Do **not** activate for general navigation, unrelated weather/news, or questions about other people. This skill only reads the authenticated user's phone location — nothing else.

## Tools

Tool names are referenced here without any host-specific prefix. Your host may expose them under a prefix — use whatever name your tool registry shows. Common examples:

| Host | Tool name you'll see |
|---|---|
| Claude.ai / Claude Desktop | `get_current_location` |
| Claude Code | `mcp__geolocate-me__get_current_location` |
| Cursor / Windsurf | `mcp_geolocate-me_get_current_location` |
| Hermes Agent | `mcp_geolocate_me_get_current_location` |
| Goose | varies by extension config |

All prefixes point at the same underlying tool.

### `get_current_location`

The most recent GPS ping from the user's phone.

- Parameters: none
- Returns: text line with `Lat`, `Lng`, accuracy (±meters), ISO timestamp, relative age (e.g. `3m ago`), and a reverse-geocoded street address.

Call this for any "where am I / where's my phone / am I at X" question.

### `get_location_history`

Historical pings for the user's phone. Returns up to 1000 entries, newest first.

- Parameters (all optional):
  - `hours` — return pings from the last N hours
  - `from` — ISO 8601 start timestamp, inclusive
  - `to` — ISO 8601 end timestamp, inclusive
- Returns: a count plus one line per ping, same format as `get_current_location`.

If the user asks about "yesterday", "last week", "this morning", convert that phrase to `from`/`to` based on the current date/time before calling. For rolling windows like "the last 6 hours", pass `hours` instead.

## Core patterns

### 1. Current location

> **User:** "Where am I?" / "Where's my phone?"

Call `get_current_location`. Respond with the **street address and relative age**, not coordinates. Example:

> You're at 14 Elizabeth St, Surry Hills (5 minutes ago, ±10m accuracy).

If the accuracy is poor (>100m), mention it. If the ping is old (>30min), say so.

### 2. Historical single point

> **User:** "Where was I yesterday at 3pm?"

Translate the question into a narrow `from`/`to` window (e.g. ±15 minutes around the stated time). Call `get_location_history`, take the ping closest to the target, respond with the address.

### 3. Rolling window

> **User:** "Show me where I've been in the last 6 hours."

Call `get_location_history` with `hours=6`. Summarize by grouping consecutive close-coord pings into stops. Don't paste every ping.

### 4. Dwell time

> **User:** "How long was I at the office today?"

Call `get_location_history` with `hours=24` (or `from=start of today`). Identify pings with the same reverse-geocoded address (or coordinates within ~50m). The dwell time is `last_such_ping.timestamp - first_such_ping.timestamp`.

### 5. Day reconstruction

> **User:** "Reconstruct my day" / "Where did I go today?"

Call `get_location_history` with `from=start of today`. Segment into stops by collapsing consecutive close-coord pings. Present as a timeline:

> Today so far:
> - 7:00–8:40am — Home (123 Example St)
> - 8:40–9:15am — in transit
> - 9:15am–12:30pm — Office (456 Main St)
> - ...

See `references/recipes.md` for the exact clustering algorithm.

### 6. Inferring home / office

> **User:** "Where's home?" / "Where do I work?"

Call `get_location_history` with `hours=168` (last week). Cluster pings; **home** is the location where the user ends most days (most pings between 22:00 and 06:00 local). **Office** is the weekday-daytime cluster with the most cumulative time. Full rules: `references/recipes.md`.

## Response style

- **Lead with the place**, not the coordinates. "You're at 14 Elizabeth St, Surry Hills" beats "Lat: -33.88, Lng: 151.21".
- **Use relative time** ("about 12 minutes ago") in addition to absolute timestamps when it aids comprehension.
- **Summarize, don't dump.** Never list 100 pings. Group into stops.
- **Flag uncertainty.** If a ping is stale (>30m) or imprecise (±>100m), say so.
- **Respect the scope.** This is the user's own phone. Don't roleplay that you can see other people's locations, even if asked.

## Edge cases

- **No pings yet**: `get_current_location` may return "No location data available yet." Tell the user to open the Geolocate Me app and confirm tracking is on.
- **Stale data**: If the last ping is hours old, the phone might be offline or tracking is off. Suggest the user check the app.
- **Empty history window**: Widen the window or tell the user no pings were recorded in that range.
- **Poor accuracy**: Indoor / low-signal pings can be ±500m. Don't over-commit to a specific address.

## Setup

See `references/setup.md` for host-specific configuration (Claude.ai, Claude Desktop, Claude Code, Cursor, Windsurf, Hermes Agent, Goose, LM Studio, Open WebUI, and generic MCP clients).

## Advanced recipes

See `references/recipes.md` for:

- Ping-clustering algorithm (stop detection)
- Home / office inference rules
- Anomaly detection ("was I somewhere new?")
- Travel log generation
- Commute-time estimation

---

Built by [Guleki](https://guleki.com) · Source: <https://github.com/linville-charlie/geolocate-me-skill> · App: <https://geolocateme.app>
