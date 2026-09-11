# DraftIQ

A personal fantasy football draft assistant: live recommendations, tiers,
scarcity/urgency alerts, targets/avoids, and draft-board tracking, run
entirely on your own machine.

## Setup

1. Install Python 3.10+.
2. From this folder, install dependencies:
   ```
   pip install -r requirements.txt
   ```

## Running it

```
python app.py
```

Then open **http://127.0.0.1:5000** in your browser. Stop the server with
`Ctrl+C`.

By default it serves on port 5000. To use a different port:

```
PORT=8000 python app.py
```

## Loading your rankings

`uploads/sample_players.csv` is a **format template only** — placeholder
names ("QB Example 1", etc.), not real players or real rankings. It exists
so the app has something to show you immediately and so you can see the
exact column layout expected.

Before you draft, replace it with a real rankings export (from
FantasyPros, ESPN, your own spreadsheet, etc.) using the **Setup** tab's
upload box. Required columns:

| column | notes |
|---|---|
| `name` (or `player`/`player_name`) | player's name |
| `position` (or `pos`) | QB/RB/WR/TE/DEF/K, etc. |
| `rank` | overall rank — required, rows without one are dropped |
| `adp` | average draft position — optional but recommended |
| `team` | optional |

### Or sync from Fantasy Football Calculator instead

The **Setup** tab also has a "Sync from Fantasy Football Calculator"
option that pulls live ADP rankings directly — no CSV needed. Pick a
scoring format (Standard / Half PPR / Full PPR) and your team count,
then hit **Sync Rankings**.

A player's **rank** is always their position in that sorted list (1st,
2nd, 3rd...) — never the raw ADP number, which is a fractional average
(e.g. 24.4) and isn't the same thing.

A few things worth knowing:

- **Cached once a day.** Fantasy Football Calculator's own ADP data only
  updates once a day, and they ask API users not to poll more often than
  that. DraftIQ respects this automatically — a sync within 24 hours of
  the last one for that scoring format/team-size just reuses what's
  already cached, unless you check **Force Refresh**.
- **A few days of history are kept** (per scoring format + team size)
  specifically so ADP movement can be tracked over time, not just the
  latest snapshot.
- **ADP Movers**, shown on the on-clock (Draft Room) screen, flags
  players whose ADP has shifted meaningfully since the oldest snapshot
  still in that history — rising (being drafted earlier than before) or
  falling (being drafted later). This needs at least two days of synced
  data to show anything; right after your first sync it'll be empty,
  which is expected.
- This integration was written against Fantasy Football Calculator's
  published API docs and sample data, but hasn't been exercised against
  a live call in the environment it was built in (no outbound network
  access there). If the sync ever fails outright rather than just
  returning stale-looking data, that's the first thing to check —
  compare the actual JSON shape at
  `https://fantasyfootballcalculator.com/api/v1/adp/ppr?teams=12`
  against what `ffc_source.py`'s `_normalize()` function expects.

## Notes on this being a single-user tool

This app keeps one draft's worth of state in memory (no accounts, no
database). It's built for one person running it locally for their own
draft — not for hosting somewhere multiple people would use it
simultaneously. If you want to share it with your league or turn it into
something multiple people use at once, that needs a different
architecture (per-user sessions, a real database, etc.) — ask if you want
help scoping that out.

## Data

Nothing here leaves your machine — no accounts, no analytics, no external
calls. Your uploaded CSV and draft progress live only in this folder and
in the running process's memory.
