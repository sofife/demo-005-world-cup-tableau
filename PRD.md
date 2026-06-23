# PRD: World Cup Site → Tableau Map Embedded in the Venues Page

## Problem
We have a static World Cup tracker (`prep/index.html`) and venue data, but no deployed app and
no governed BI layer. We want a real, deployed web app whose **Venues** page shows an embedded,
interactive Tableau map of all 16 host stadiums — driven by a governed Tableau Cloud data
source — and an in-app filter (country → specific venue) that filters that embedded map.

## What's being built
A Vite + React web app (rebuilt from the Demo 001 tracker) plus the Tableau assets that back
its Venues page:
1. A React app in `build/` that reproduces the tracker's pages (Groups, Schedule, Venues,
   Bracket) reading from `worldcup2026.json` / `venues.json`.
2. A **Venues page** that embeds a Tableau map viz via the Tableau Embedding API v3.
3. A serverless function (Vercel) that mints a short-lived embedding JWT from the Connected App
   credentials so the secret never reaches the browser.
4. A Python publish tool that shapes `prep/venues.json` into a Tableau extract and publishes it
   to Tableau Cloud as a governed data source via the REST API (PAT auth, Overwrite mode).
5. An in-app **filter control** (country, then specific venue) that filters the embedded map.

The Tableau **workbook** (the map viz itself) is authored against the published data source as
part of the demo (beat 5); the app embeds that workbook's view.

## Scope
**In scope**
- React rebuild of the four tracker pages; visual parity with `prep/index.html` (dark theme).
- Venues page with an embedded Tableau point map of all 16 stadiums.
- Serverless JWT endpoint for embedding auth.
- Python publish of `venues.json` → Tableau Cloud data source (`World Cup 2026 Venues`).
- In-app country + venue filter that drives the embedded viz (Embedding API filter calls).
- Deploy to Vercel from a GitHub repo.

**Out of scope**
- Re-researching or live-scraping scores (the JSON/venue files are the inputs).
- Authoring the Tableau workbook programmatically (it's built in Tableau during the demo;
  the app only embeds the resulting view).
- Auth flows beyond the seeded PAT (publish) and Connected App (embedding).
- Mobile-specific layouts beyond responsive basics.

## Inputs
- `prep/index.html` — the updated Demo 001 tracker (visual + behavior reference for the rebuild).
- `prep/worldcup2026.json` — tournament source of truth (teams, groups, schedule, knockout).
- `prep/venues.json` — 16 venues with `stadium, city, country, capacity, lat, lon` (map source).
- `build/secrets.env` (gitignored) — `TABLEAU_SITE_URL`, `TABLEAU_PAT_NAME`, `TABLEAU_PAT_SECRET`
  (REST publish); `TABLEAU_CONNECTED_APP_CLIENT_ID/SECRET_ID/SECRET_VALUE`, `TABLEAU_JWT_USERNAME`
  (embedding); `TABLEAU_PROJECT` (publish target).

## Output
- A deployed Vercel app (React) with Groups / Schedule / Venues / Bracket pages.
- `build/` React project + serverless JWT function.
- `build/publish_venues.py` (or similar) + `build/venues.hyper` extract.
- A published `World Cup 2026 Venues` data source on the Tableau Cloud developer site.
- A Tableau workbook with a point map of the 16 venues (built in the demo, embedded in the app).

## Success criteria
- The deployed site loads at a public Vercel URL; all four pages render with parity to the tracker.
- The Venues page shows the embedded Tableau map with all 16 stadiums plotted at their lat/lon.
- The published data source appears on Tableau Cloud and is queryable; venue count = 16.
- Selecting a country (USA/Mexico/Canada) filters the embedded map to that country's venues;
  selecting a specific venue filters to that single point.
- The Connected App secret never appears in client-side code or network payloads (JWT minted
  server-side only).

## Open questions to resolve in build
- **Embed object:** embed a single map *view* and filter it, vs. embed a dashboard. Default to a
  single worksheet/view for simplicity.
- **Filter mechanism:** Embedding API `applyFilterAsync` on a field (e.g. `country`, `stadium`)
  vs. a parameter. Default to categorical filters on `country` and `stadium`.
- **Where the publish runs:** Tableau Cloud is unreachable from the Cowork sandbox; run the
  publish from Claude Code / a local shell (see HANDOFF).
