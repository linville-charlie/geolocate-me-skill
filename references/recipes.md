# Recipes

Detailed behavioral patterns for using the Geolocate Me tools well. Load this file when the user asks anything that requires more than a simple lookup.

## Stop detection (ping clustering)

A **stop** is a period where the user stayed roughly put. Algorithm:

1. Call `get_location_history` with an appropriate time window.
2. Pings are returned newest first — reverse to chronological order.
3. Walk the list. A ping starts a **new stop** if it is more than ~75 meters from the current stop's centroid.
4. A stop ends when the next ping is far away OR there's a gap >5 minutes with no close-enough ping.
5. Ignore stops shorter than 3 minutes (they're likely traffic stops, not actual visits).

Pseudocode:

```
sort pings ascending by timestamp
stops = []
current = null
for p in pings:
  if current is null or distance(p, current.centroid) > 75m:
    if current and current.duration > 3min:
      stops.append(current)
    current = new stop starting at p
  else:
    current.add(p)
if current and current.duration > 3min:
  stops.append(current)
```

Use Haversine distance or, for rough clustering, just compare lat/lng rounded to 4 decimal places (~11m precision).

## Dwell time at a specific place

> User: "How long was I at the office today?"

1. Call `get_location_history` with `from=start_of_local_today`.
2. Run stop detection.
3. Find stops whose reverse-geocoded address matches "office" (use known home/office addresses if already inferred; otherwise ask which one).
4. Report: `sum(end - start)` across matching stops.

If the user hasn't told you where the office is, either:
- Ask ("which address is the office?") — best when the user is a new user
- Use the office-inference recipe below to guess — then confirm

## Home / office inference

Only worth doing if the user has at least 3 days of history.

1. Call `get_location_history` with `hours=168` (a week).
2. Run stop detection. You'll get a list of stops, each with a centroid and total duration across days.
3. **Home** is typically the stop with the most cumulative time between 22:00 and 07:00 local. It's often where stops end each day.
4. **Office** is the stop with the most cumulative time between 09:00 and 17:00 on weekdays, that is *not* home.
5. Confirm with the user before relying on these inferences.

## Day reconstruction

> User: "Reconstruct my day"

1. Call `get_location_history` with `from=start_of_local_today`.
2. Run stop detection.
3. Present as a timeline:

```
Today:
- 07:00–08:40  Home (123 Example St)
- 08:40–09:15  In transit
- 09:15–12:30  Office (456 Main St)
- 12:30–13:15  Lunch — 12 Example Ave
- 13:15–17:45  Office
- 17:45–18:20  In transit
- 18:20–now    Home
```

Gaps between stops are labelled **"In transit"**. Compute transit duration from the gap between the previous stop's last ping and the next stop's first ping.

## Travel-log generation

> User: "Draft a travel log for last week."

1. Call `get_location_history` with `hours=168`.
2. Run stop detection with a larger threshold (e.g. 200m, since travel-relevant stops are coarser).
3. Group consecutive stops in the same city/neighborhood.
4. Write a narrative: "Monday morning started at home, then a 9:30am meeting in the CBD. Afternoon in Surry Hills. Dinner in Redfern."

Reverse-geocoded addresses from the tool include suburb/neighborhood — use that rather than full street addresses for readability.

## Commute-time estimation

> User: "When should I leave for the office?"

1. Call `get_location_history` with `hours=168`.
2. Filter to trips that start near home and end near office (or vice versa).
3. Compute duration for each. The median is a reasonable estimate; include the 90th percentile if the user wants a safety buffer.

## Anomaly detection

> User: "Have I been anywhere unusual recently?"

1. Call `get_location_history` with `hours=720` (30 days).
2. Run stop detection.
3. Build a set of "familiar" stops: any location with ≥3 distinct visits in the past 30 days.
4. Any stop in the recent window (e.g. last 7 days) that is **not** in the familiar set is a candidate anomaly.
5. Present: "You spent 2 hours in Bondi on Saturday — you haven't been there in the last 30 days."

## Time-at-location heatmap (textual)

> User: "Where do I spend most of my time?"

1. Call `get_location_history` with `hours=168`.
2. Run stop detection.
3. Bucket by reverse-geocoded suburb/neighborhood.
4. Sum durations per bucket, rank descending.

Present as a simple list:

```
This week:
  Home (Surry Hills) — 58h
  Office (CBD)       — 41h
  Gym (Waterloo)     — 5h
  Other              — 12h
```

## Response-formatting rules

Across all recipes:

- Use the **user's local time**, not UTC. Infer from the server-returned timestamps + user's apparent timezone if needed.
- Round times to 5-minute granularity for presentation ("09:15" rather than "09:12:47").
- Show **suburb or neighborhood** (from reverse-geocoded address) unless the user asks for the precise street address.
- When summarizing, suppress stops under 3 minutes.
- Prefer addresses. Output raw lat/lng only when the user asks for them or analysis requires them.

## Rate-limit awareness

- `get_location_history` returns up to 1000 pings per call. A week at 30s intervals is ~20,000 pings — you'll get truncated results.
- For long ranges, prefer fewer but larger calls (one `hours=168` call beats seven `hours=24` calls).
- Don't call `get_current_location` repeatedly for the same question — call once, reuse the result.
