# Demo: World Cup Site → Tableau Map Embedded in the Venues Page

> **How to run this:** Cowork opens this file at the start of the demo, reads the brief
> back to the audience, then waits. Work begins only after the presenter says "go."

## Audience
Data/analytics and BI/Tableau practitioners, plus anyone evaluating Cowork + Claude Code
for end-to-end "data product" workflows. They care about the full arc: ship a real web app,
publish governed data to Tableau Cloud, and stitch an interactive Tableau viz back into the
app so the two stay in sync.

## What this demo shows
The longest arc in the demo series. We start from the Demo 001 World Cup tracker and turn it
into a deployed web app, then make it *data-driven by Tableau*: the venue data is published
to Tableau Cloud as a governed data source, a map workbook plots every host stadium, and that
map is embedded live into the app's **Venues** page. A filter in the app (by country, then by
specific venue) drives the embedded Tableau map in real time. It connects research → app →
governed BI → interactive embed in one continuous story.

## The scenario
The 2026 World Cup is hosted across 16 stadiums in the USA, Mexico, and Canada. We already
have the tournament as structured data (`prep/worldcup2026.json`) and a tracker UI
(`prep/index.html`), and we've enriched the venues with precise stadium coordinates
(`prep/venues.json`). We want a real, deployed site where the Venues page shows an embedded
Tableau map of every stadium — and where clicking a country or a specific venue in the app
filters that map.

## The plan
Once given the go-ahead, this demo runs in six beats:
1. **Write the build spec.** Produce `PRD.md` and `HANDOFF.md` so Claude Code knows exactly
   what site to build (already scaffolded here — this beat is reviewing/finalizing them).
2. **Claude Code builds the site.** A Vite + React rebuild of the tracker in `build/`, with a
   Venues page wired for an embedded Tableau viz and a country/venue filter, plus a small
   serverless function to mint the Tableau embedding JWT.
3. **Deploy.** Push to GitHub and deploy on Vercel; show the live URL.
4. **Publish the data source.** Shape `prep/venues.json` into a Tableau extract and publish it
   to Tableau Cloud as a governed data source via the REST API (PAT auth).
5. **Build the map workbook.** Create a Tableau workbook with a symbol/point map showing each
   venue as a point on the map, connected to the published data source.
6. **Wire the filter.** Add an in-app filter — country, then specific venue — that, when
   clicked, filters the embedded Tableau map via the Embedding API.

## What "done" looks like
- A live Vercel URL serving the React app, with a working **Venues** page.
- A governed `World Cup 2026 Venues` data source on the Tableau Cloud developer site.
- A Tableau workbook with a point map of all 16 stadiums, embedded in the Venues page.
- An in-app country/venue filter that visibly filters the embedded map on click.

## Pace / emphasis
Live screen share — favor visible momentum across the six beats. The big "wow" moments are
(a) the deployed site going live on Vercel and (b) clicking a venue filter in the app and
watching the embedded Tableau map react. Keep narration flowing during the build/publish
steps; show the live result after each beat rather than going silent.

---
## Presenter notes (do NOT read aloud)
- **Carried from Demo 001:** `prep/worldcup2026.json` (tournament source of truth) and
  `prep/index.html` (the updated tracker — this is the "fresh copy from 001" base for the
  rebuild). **New in prep:** `prep/venues.json` — the 16 venues enriched with precise
  stadium lat/lon (the source JSON had no coordinates).
- **Stack decision (locked):** Vite + React rebuild of the tracker. JWT for embedding is
  minted by a serverless function (Vercel) so the Connected App secret never ships to the
  browser.
- **"World map" is really North America** — all 16 venues are USA/Mexico/Canada. The map
  auto-fits to the points. Country filter = USA / Mexico / Canada.
- **Secrets:** `build/secrets.env` (gitignored) is a real copy from Demo 003 — PAT (REST
  publish) + Connected App client/secret + JWT username (embedding). Never paste any value
  into chat or the transcript.
- **Known constraint (from Demo 003): Tableau Cloud is NOT reachable from the Cowork sandbox
  (403 tunnel).** The REST publish (beat 4) and any live Tableau auth run from Claude Code /
  a local shell, not from Cowork. Vercel + GitHub steps are fine from anywhere with network.
- **Sequencing risk:** beats 4–5 (publish + workbook) must exist before beat 6 (the embed)
  can show anything real. If publishing stalls live, fall back to a pre-published data source
  / pre-built workbook and narrate the embed against that.
- **Embedding tech:** Tableau Embedding API v3 (`<tableau-viz>` web component) + a JWT from the
  Connected App; the in-app filter calls `applyFilterAsync` (or sets a viz filter) on click.
- This is a long demo — consider check-pointing after beat 3 (site live) and beat 5 (map
  built) so each half can stand alone if time runs short.
