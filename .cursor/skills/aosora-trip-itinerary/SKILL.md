---
name: aosora-trip-itinerary
description: >-
  Build, refine, and publish AoSora travel itinerary HTML sites (bilingual Leaflet +
  CARTO/OSRM maps, Google place-name links) to GitHub Pages at trip.aosora.photography
  (repo COYADI/sora-trip-designer). Use when the user asks for a trip itinerary page,
  aosora trip site, import from spreadsheet/bookings, basemap change, Ring Road / OSRM
  route, GitHub Pages deploy, trip.aosora.photography, or to reuse this skill.
---

# AoSora trip itinerary → GitHub Pages

## Defaults
- Public hub: `https://trip.aosora.photography`
- Host: GitHub Pages from `https://github.com/COYADI/sora-trip-designer` (`main` / root)
- Apex / `www` (`aosora.photography`) stay on Adobe Portfolio — never retarget those DNS records
- Local working tree: `aosora-trip/` (often under the Cursor project). Prefer absolute paths; do **not** call `move_agent_to_root` unless the user asks
- Reply to the user in Traditional Chinese (zh-TW) unless they request another language

## Privacy (public site)
**Never publish personal data** on hub or trip pages (or in this skill):
- No full names, emails, phone numbers of travelers
- No booking / reservation / confirmation codes (hotels, cars, restaurants, flights PNR)
- No passport / license / payment details
- OK on the public page: hotel **names**, city nights, **shop locations** + pick/return **clock times**, restaurant **name** + meal time, airport codes + flight **numbers** if the user wants them on the itinerary (times as “check day-of”), public POI addresses
- When the user pastes booking screenshots, extract logistics into the itinerary but **strip codes and PII** before commit/deploy
- Do not hard-code one traveler’s private bookings as skill defaults

## Site structure
```
aosora-trip/
  CNAME                 # trip.aosora.photography
  .nojekyll
  index.html            # hub: brand + trip links (single-line titles)
  DEPLOY.md
  <trip-slug>/          # e.g. iceland-2026/ , japan-2026/
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
  - Alternatives: [basemaps.md](basemaps.md)
- **Places `P`**: `zh`, `orig`, `lat`, `lng`, `gmaps` (official Google place name)
- **Links**: `mapsPlaceUrl` / `mapsDirUrl` use `gmaps` + `encodeURIComponent` — not raw coordinates
- **Days**: `places` (markers) + `route` (Google 導航 order) + optional `highlight`
- **Stays**: prefer **hotel keys** in `stayKeys` / stay pills when bookings exist (not only city centroids)
- **Overview route**: precompute driving via OSRM into `PRECOMPUTED_RING_ROUTE`
  - Include **itinerary spurs** (day loops, pick-up→stay, return-leg stops) — not only the “main highway”
  - `fitBounds` = **route ∪ stay markers ∪ highlight markers**
  - Load `*-ring-route.js` via `<script>` before main logic; JSON as fetch fallback
  - Simplify dense OSRM output (RDP ~40 m)
- **UX**: progressive enhancement; try/catch around Leaflet; never leave `.reveal` stuck at `opacity: 0`
- **Hub**: brand-first; trip titles one line (`white-space: nowrap` where needed)

## Ingest: spreadsheet + bookings (learned workflow)
1. **Sparse sheet is a skeleton** (dates / 住 / 租車 / 早午晚). Fill recommended POIs per day; keep sheet constraints when the user clarifies them.
2. **Booking screenshots override the sheet** for stays, airports, car pick/return, fixed dinners, office days.
3. **Google Sheet export**: shared CSV/gviz may mojibake in consoles; if cells look wrong, use the browser view or write CSV to a UTF-8 file and `Read` it. Prefer the live sheet UI for Chinese text.
4. **Stale tourism URLs**: verify before linking. Example — Fuji **神玉巡拜** (not 「神珠」):
   - Guide: `https://fujiyoshida.net/course/index.php?p=course-530`
   - Map: `https://kamitamajunpai.com/kamitamamap-cat/fujisan/`
   - Old `…/zh_TW/highlight/kamitamajunpai_TC/…` paths often **404**
5. **Semantic labels**: if the user separates “collection activity” vs “return-drive free stops”, do not merge them (e.g. finish a pilgrimage only in its region; “遺珠”-style days = stops **on the drive home**, not a second trip back).
6. **Flights**: confirm airport codes (e.g. Songshan↔Haneda vs Narita). Late inbound → hotel late check-in note; outbound → checkout + transfer buffer.
7. **Car**: record pick shop / return shop / clock times; align day timelines (checkout→pick-up, return deadline before hotel).
8. **Work / dinner days**: office as a `P` + map pin; restaurant with time + area; commute from the night’s hotel.

## New trip / edit workflow
1. Create or edit `aosora-trip/<trip-slug>/` — **UTF-8 only** (never UTF-16). On Windows, if an editor writes UTF-16 (`NUL` bytes), convert with Python before serving.
2. Prefer copying structure/CSS/JS helpers from an existing trip (`iceland-2026` / `japan-2026`), then replace `P` / `days` / stays / copy.
3. Update `routeOrder` for the real path; regenerate OSRM (`router.project-osrm.org` OK for personal use).
4. Link from hub `index.html` (one-line title + short meta chips).
5. Local preview: `python -m http.server` from `aosora-trip/` (MCP browser may not reach localhost).
6. Deploy (below). Do **not** commit `*-preview.html`, compare PNGs, or `_*.py` scratch scripts unless asked.

## Deploy
1. Commit **only** site files (`CNAME`, `.nojekyll`, hub `index.html`, `<slug>/`) to `COYADI/sora-trip-designer` `main`
2. Pages: branch `main` / `/ (root)`; custom domain `trip.aosora.photography`; Enforce HTTPS when DNS ready
3. Root `CNAME` must be exactly `trip.aosora.photography`
4. After push: `gh api repos/COYADI/sora-trip-designer/pages/builds -X POST`
5. Verify: `https://trip.aosora.photography/<slug>/` (cert may lag)

Windows PowerShell: `C:\Program Files\Git\bin\git.exe`, `C:\Program Files\GitHub CLI\gh.exe`  
No bash heredocs;  
`git commit --trailer "Co-authored-by: Cursor <cursoragent@cursor.com>" -m "message"`

## GoDaddy DNS (zh-TW UI)
**我的產品** → 網域 → **DNS** → **新增記錄**：
- **類型**：CNAME · **名稱**：`trip` · **內容值**：`COYADI.github.io`
- Do not change `@` / `www` (Adobe Portfolio)

## Do not
- Retarget apex/`www` to GitHub Pages while Portfolio is live
- Publish PII or booking confirmation codes
- Use ambiguous Google search strings; use `gmaps` place names
- Fit overview map to a simplified trunk route when spurs/stays would be cropped
- Commit secrets (`.env`, tokens). CARTO keys are client-side — Referer-lock; don’t paste keys into chat unnecessarily
- Serve UTF-16 site files
- Treat spreadsheet city labels as final when hotel/airport/car bookings disagree

## Quick checks
- [ ] Hub + trip titles one line where intended
- [ ] No emails / traveler names / reservation codes on public HTML
- [ ] Overview line includes major drives + day loops + return-leg stops
- [ ] Overview bounds include all stay hotels and key highlights
- [ ] Basemap shows roads (Voyager); CARTO attribution visible
- [ ] Google buttons open named places, not coordinate pins
- [ ] External tourism links return 200 (not stale locale paths)
- [ ] Pages served from `COYADI.github.io` / custom domain
