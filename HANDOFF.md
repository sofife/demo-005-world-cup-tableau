# HANDOFF — for Claude Code

## STATUS (prep done in Cowork, 2026-06-22)
Prep is scaffolded. The build is **not** written yet — that's your job. This demo turns the
Demo 001 World Cup tracker into a deployed React app whose **Venues** page embeds an
interactive Tableau map, backed by a governed Tableau Cloud data source, with an in-app
country/venue filter driving the map.

- `prep/index.html` — the updated Demo 001 tracker (single-file, dark theme, four pages:
  Groups, Schedule, Venues, Bracket). **This is the visual + behavior reference** for the
  React rebuild. Match its look and data behavior.
- `prep/worldcup2026.json` — tournament source of truth (teams, 12 groups, full schedule with
  scores, knockout structure). Unplayed matches have `null` scores.
- `prep/venues.json` — the 16 host venues enriched with `stadium, city, country, capacity,
  lat, lon`. **This is the map data source.** (The original tournament JSON had no coordinates.)
- `build/secrets.env` (gitignored) — real creds copied from Demo 003:
  - REST publish: `TABLEAU_SITE_URL` (pod + site content URL), `TABLEAU_PAT_NAME`,
    `TABLEAU_PAT_SECRET`, `TABLEAU_PROJECT`.
  - Embedding: `TABLEAU_CONNECTED_APP_CLIENT_ID`, `TABLEAU_CONNECTED_APP_SECRET_ID`,
    `TABLEAU_CONNECTED_APP_SECRET_VALUE`, `TABLEAU_JWT_USERNAME`.
  - **Never echo any value.** Never ship the Connected App secret to the browser.
- **Tableau Cloud is NOT reachable from the Cowork sandbox** (403 tunnel, confirmed Demo 003).
  Run the REST publish and any live Tableau auth from a local shell, not from Cowork.

## Goal
Build, in `build/`: (1) a Vite + React app that reproduces the tracker and embeds a Tableau
map on the Venues page, (2) a serverless JWT endpoint for embedding auth, (3) a Python tool
that publishes `venues.json` to Tableau Cloud, and (4) an in-app country/venue filter that
filters the embedded map. Deploy to Vercel via GitHub. See `PRD.md` for scope/success criteria
and `DEMO.md` for the live framing.

## Stack (locked)
- **Frontend:** Vite + React (JS or TS — your call; keep it light). Tailwind optional; the
  reference uses hand-rolled CSS variables — port the dark theme either way.
- **Data:** bundle `worldcup2026.json` and `venues.json` as static assets the app reads.
- **Embedding:** Tableau Embedding API v3 web component (`<tableau-viz>`), loaded from Tableau's
  embedding JS. Auth via a JWT minted server-side.
- **JWT endpoint:** a Vercel serverless function (`/api/tableau-jwt` or similar) that signs a
  JWT from the Connected App creds (`CLIENT_ID`, `SECRET_ID`, `SECRET_VALUE`, scopes for embed,
  `sub = TABLEAU_JWT_USERNAME`). Secret stays server-side; expose only the short-lived token.
- **Publish:** Python — `tableauserverclient` (PAT auth) + `pantab`/Hyper API for the extract.

## Where to start
1. Read `PRD.md` (scope, inputs/outputs, success criteria) and skim `prep/index.html`.
2. Scaffold the Vite + React app in `build/`; port the four pages and the dark theme from the
   reference. Wire Groups/Schedule/Bracket to `worldcup2026.json`, Venues to `venues.json`.
3. Write `build/publish_venues.py`: load `venues.json` → tidy table → `venues.hyper` → sign in
   with PAT → publish Overwrite as `World Cup 2026 Venues` to `TABLEAU_PROJECT`. Print the URL.
   Run it from a shell that can reach Tableau Cloud (not Cowork).
4. In Tableau, author a workbook on the published data source: a symbol/point map using
   `lat`/`lon` (mark = circle, geographic). Publish the workbook; note the view's embed path.
5. Build the **Venues page embed**: `<tableau-viz>` pointing at the published view, token from
   the serverless JWT endpoint.
6. Add the **filter**: a control listing countries (USA/Mexico/Canada), each expanding to its
   venues; on click, call `applyFilterAsync` on `country` / `stadium` against the embedded viz.
7. Push to GitHub, deploy on Vercel, set the Tableau env vars in Vercel project settings, verify
   the live URL.

## Key decisions / constraints
- **JWT minted server-side only** — the Connected App secret must never appear in client bundles
  or network payloads. Put Tableau env vars in Vercel, not in committed files.
- **Map marks:** use the venue `lat`/`lon` for exact stadium placement (don't rely on Tableau's
  city geocoding). Mark type = filled/symbol point; tooltip = stadium, city, capacity.
- **Filter fields:** filter on `country` (3 values) and `stadium`/`venue_id` (16 values). Keep
  the in-app control and the embedded viz filters consistent.
- **All venues are North America** (USA/Mexico/Canada) — the "world map" auto-fits to the points.
- **Publish mode Overwrite** so re-runs replace the data source cleanly.
- Keep `secrets.env`, `*.hyper`, `node_modules/`, build output, and `__pycache__/` out of git
  (see `.gitignore`). Add a `.env.example` documenting required vars without values.

## Watch for
- **Sandbox can't reach Tableau Cloud** — do the publish + workbook from local/Claude Code.
- **CORS / embed domain allow-list** — the Vercel domain may need to be allowed for embedding on
  the Tableau site. Check Tableau Cloud → Settings → Connected Apps / embedding allow-list.
- **JWT scopes/expiry** — short expiry; include the right embedding scopes or the viz won't load.
- **Data parity** — venue count must be 16; spot-check stadium names against `prep/index.html`.
- **`null` scores** in the tournament JSON — handle gracefully in the Schedule/Groups pages.
