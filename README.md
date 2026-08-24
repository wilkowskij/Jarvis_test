# Jarvis — personal dashboard

Live: https://claude.ai/code/artifact/323f876d-297b-4d92-a367-767509724bb0

A single-page animated dashboard published as a claude.ai Artifact. It declares
the `mcp` and `artifact` runtime capabilities, so it isn't just a static
snapshot:

- **Important email** and **today's calendar** are read live from your
  connected Gmail and Google Calendar via `claude.use('mcp')`, refreshed on
  an interval and on demand (the ⟳ button).
- **Action items** combine auto-derived signals (RSVPs pending, unread
  threads waiting on a reply, meetings needing prep) with your own checklist,
  which persists across visits via `claude.use('artifact')`.
- **Markets** and **headlines** can't be fetched live from inside the page
  (no market-data connector, sandboxed egress) — they're baked into
  `data/market.json` and refreshed by a scheduled weekday task, growing one
  real close per instrument per run.

## Files

| Path | Role |
|---|---|
| `index.html` | The whole dashboard — markup, CSS, JS, embedded data. Published as-is. |
| `data/market.json` | Source of truth for the accumulating market series + headlines. |

## How the two writers coexist

`index.html` embeds two `<script type="application/json">` blocks:
`market-data` (owned by the scheduled refresh) and `user-state` (owned by
the page itself — your checklist, dismissed auto-items, email filter).
The page never serializes its live, interacted-with DOM back to itself;
it clones the document, empties the rendered `#app-root` container, updates
just its own data block, and republishes that. The refresh task does the
same in reverse: it reads the currently published page, keeps your
`user-state` block untouched, rewrites `market-data`, and republishes.
Neither writer ever clobbers the other's half.

## Re-publishing by hand

Open a Claude Code session against this repo and ask it to republish
`index.html` to the URL above (pass the URL explicitly so it updates in
place rather than creating a new artifact).
