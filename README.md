# Demo 005 — World Cup Site → Tableau Map Embedded in the Venues Page

## What this demo shows
The full data-product arc. It takes the Demo 001 World Cup tracker, rebuilds it as a deployed
React app, publishes the venue data to Tableau Cloud as a governed data source, builds a Tableau
map of all 16 host stadiums, and embeds that map back into the app's **Venues** page — with an
in-app country/venue filter that drives the embedded map. Research → app → governed BI →
interactive embed, end to end.

## Audience
Data/analytics and BI/Tableau practitioners evaluating Cowork + Claude Code for end-to-end
workflows. The "wow" beats: the site going live on Vercel, and clicking a venue filter in the
app and watching the embedded Tableau map react.

## What's carried over / new in prep
- **`prep/index.html`** — the updated Demo 001 tracker (fresh copy from Demo 001); the visual +
  behavior reference for the React rebuild.
- **`prep/worldcup2026.json`** — tournament source of truth (teams, groups, schedule, knockout).
- **`prep/venues.json`** *(new)* — the 16 host venues enriched with precise stadium lat/lon, so
  the Tableau map plots exact points. The original tournament JSON had no coordinates.

## What's new here (the build)
A Vite + React app in `build/` plus its Tableau backing:
- React rebuild of the tracker (Groups / Schedule / Venues / Bracket).
- A Venues page that embeds a Tableau point map via the Embedding API v3.
- A serverless function that mints the embedding JWT (Connected App secret stays server-side).
- A Python tool that publishes `venues.json` to Tableau Cloud as a governed data source.
- An in-app country/venue filter that filters the embedded map.

## How to run / present
Coding demo. Prep (this folder) is done in Cowork; the build, publish, and Tableau workbook run
in Claude Code / locally. See `PRD.md` for scope/success criteria and `HANDOFF.md` for the
Claude Code starting point. The six beats: write spec → build site → deploy (GitHub + Vercel) →
publish data source → build map workbook → wire the filter.

## Important constraint
Tableau Cloud is **not reachable from the Cowork sandbox** (403 tunnel, observed in Demo 003).
The REST publish and live Tableau auth run from Claude Code / a local shell. GitHub + Vercel
steps work from anywhere with network.

## Credentials
`build/secrets.env` (gitignored) is a real copy from Demo 003: Tableau PAT (REST publish) +
Connected App client/secret + JWT username (embedding) + target project. Never paste any value
into chat or the transcript; in production these live in Vercel env vars, not in committed files.

## Run it later
Present with `/run-demo demo-005-world-cup-tableau`.
