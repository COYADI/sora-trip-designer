---
name: aosora-trip-itinerary
description: >-
  Build, refine, and publish AoSora travel itinerary HTML sites (bilingual Leaflet +
  CARTO/OSRM maps, Google place-name links) to GitHub Pages at trip.aosora.photography
  (repo COYADI/sora-trip-designer). Use when the user asks for a trip itinerary page,
  aosora trip site, map basemap change, Ring Road / OSRM route, GitHub Pages deploy for
  trips, trip.aosora.photography, or to reuse this skill.
---

# AoSora trip itinerary → GitHub Pages

## Defaults
- Public hub: `https://trip.aosora.photography`
- Host: GitHub Pages from `https://github.com/COYADI/sora-trip-designer` (`main` / root)
- Apex / `www` (`aosora.photography`) stay on Adobe Portfolio — never retarget those DNS records
- Local working tree: `aosora-trip/` (often under the Cursor project). Prefer absolute paths; do **not** call `move_agent_to_root` unless the user asks
- Reply to the user in Traditional Chinese (zh-TW) unless they request another language

## Site structure
```
aosora-trip/
  CNAME                 # trip.aosora.photography
  .nojekyll
  index.html            # hub: brand + trip links (single-line titles)
  DEPLOY.md
  <trip-slug>/          # e.g. iceland-2026/
    index.html
    <slug>-ring-route.js    # window.PRECOMPUTED_RING_ROUTE = [...]
    <slug>-ring-route.json  # same array (fetch fallback)
```

## Itinerary HTML conventions
- **UI**: dark page chrome; Traditional Chinese; `Noto Sans TC` / `Noto Serif TC`
- **Basemap (default)**: Leaflet + **CARTO Voyager** (light, roads visible)
  - URL: `https://{s}.basemaps.cartocdn.com/rastertiles/voyager/{z}/{x}/{y}{r}.png?key=KEY`
  - Attribution: OpenStreetMap + CARTO (required)
  - Raster CARTO tiles need `?key=` (watermark otherwise). Reuse the key already in the trip’s `darkTiles` / tile helper; do not invent a new key. If missing, ask the user or point them to https://carto.com/basemaps/apikey/
  - Prefer Referer lock on the CARTO dashboard to `trip.aosora.photography` (+ localhost key if needed)
  - Alternatives and when to use them: [basemaps.md](basemaps.md)
- **Places `P`**: `zh`, `orig`, `lat`, `lng`, `gmaps` (official Google place name)
- **Links**: `mapsPlaceUrl` / `mapsDirUrl` use `gmaps` + `encodeURIComponent` — not raw coordinates
- **Days**: `places` (markers) + `route` (Google 導航 order) + optional `highlight`
- **Overview route**: precompute driving geometry via OSRM into `PRECOMPUTED_RING_ROUTE`
  - Include **itinerary spurs**, not only pure Ring Road (e.g. Iceland: Borgarnes → Snæfellsnes loop → Borgarnes → Reykjavík)
  - After drawing, `fitBounds` must cover **route ∪ stay markers ∪ highlight markers** so west / peninsula points are never cropped
  - Load `*-ring-route.js` via `<script>` before main logic; keep JSON as fetch fallback
  - Simplify dense OSRM output (e.g. RDP ~40 m) so the JS file stays small
- **UX**: progressive enhancement; try/catch around Leaflet; never leave `.reveal` / content stuck at `opacity: 0`
- **Hub**: brand-first; trip titles stay one line (`white-space: nowrap` on hero/title as needed)

## New trip / edit workflow
1. Edit under `aosora-trip/<trip-slug>/` (UTF-8 only — never UTF-16)
2. Update `P`, `days`, `routeOrder` / overview waypoints for the real drive path
3. Regenerate ring route with OSRM when the path changes (public `router.project-osrm.org` is OK for personal use)
4. Link from hub `index.html`
5. Deploy (below)

## Deploy
1. Commit **only** site files under `aosora-trip/` to `COYADI/sora-trip-designer` `main` (not preview HTML / scratch scripts unless asked)
2. Pages: Deploy from branch `main` / `/ (root)`; custom domain `trip.aosora.photography`; Enforce HTTPS when DNS is ready
3. Root `CNAME` must be exactly `trip.aosora.photography`
4. After push, optional: `gh api repos/COYADI/sora-trip-designer/pages/builds -X POST`
5. Verify: `http://trip.aosora.photography/<slug>/` (HTTPS may lag while cert provisions)

On Windows PowerShell, use full paths if needed:
`C:\Program Files\Git\bin\git.exe`, `C:\Program Files\GitHub CLI\gh.exe`
Avoid bash heredocs; use `git commit --trailer "Co-authored-by: Cursor <cursoragent@cursor.com>" -m "message"`.

## GoDaddy DNS (zh-TW UI)
**我的產品** → 網域 → **DNS** → **新增記錄**：
- **類型**：CNAME · **名稱**：`trip` · **內容值**：`COYADI.github.io`
- Do not change `@` / `www` (Adobe Portfolio)

## Do not
- Retarget apex/`www` to GitHub Pages while Portfolio is live
- Use ambiguous Google search strings; use `gmaps` place names
- Fit overview map to Ring Road only when the itinerary has peninsula / spur days
- Commit unrelated secrets (`.env`, tokens). CARTO basemap keys are client-side by design — restrict by Referer; never paste keys into chat unnecessarily
- Serve site files as UTF-16 (breaks HTML/JS in the browser)

## Quick checks
- [ ] Hub + trip titles one line where intended
- [ ] Overview line includes all major drives (stays + day loops)
- [ ] Overview bounds include Borgarnes / west (or equivalent) highlights
- [ ] Basemap shows roads (Voyager default); CARTO attribution visible
- [ ] Google buttons open named places, not coordinate pins
- [ ] Pages build served from `COYADI.github.io` / custom domain