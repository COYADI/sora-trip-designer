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

## Privacy — whole repo (hard rule)
**Scope:** every file under the git root that may be committed or already tracked — hub/trip HTML/JS/JSON, `DEPLOY.md`, `.cursor/skills/**`, comments, scratch files. Not only the trip page.

**Never store or commit the traveler’s personal data:**
- Legal / full names, nicknames used as identity, emails, personal phone numbers
- Booking / reservation / confirmation / PNR / order IDs (hotel, car, restaurant, flight, rail)
- Passport, license, membership IDs, payment method details, card last-4, “paid by …”
- Home address, private URLs, personal calendar invite text, screenshots that embed the above
- Employer-specific internal notes, employee IDs, badge numbers

**OK for a public itinerary (logistics only):**
- Hotel **brand/name**, city, night dates (no confirmation code)
- Car **pick/return shop name + area + clock times** (prefer Google place link; **omit shop phone** unless user asks)
- Restaurant **name + meal time + area**; party size only if useful and user is fine with it
- Airport codes + flight **numbers** / day-of times (remind to re-check schedule)
- Public POIs and tourist addresses
- Workplace: prefer a **district / public landmark building** pin (e.g. 愛宕／MORIタワー). Put a company name on the public site **only if the user explicitly wants it**

**When ingesting booking screenshots / sheets:** extract logistics → **strip all PII and codes** before writing files. Never paste emails or reservation IDs into HTML, JS, MD, or this skill.

### Mandatory scan before every commit / push
From repo root, search tracked + about-to-add files (adjust path). Fail the publish if anything personal matches:

```text
email-like:   @gmail @icloud @yahoo @hotmail @outlook
ids:          預約號碼|訂編|確認番号|confirmation|予約番号|PNR|booking ref
phones:       TEL |Phone |\+886|\+81[0-9\-]{8,}
obvious PII:  passport|駕照|license no|credit card|信用卡
```

Also spot-check for traveler names the user has used in chat. Do **not** commit `*-preview.html`, local notes, or `_*.py` dumps that may contain paste residue.

**Git history:** scrubbing the working tree does not erase old commits. If PII was already pushed, scrub files and commit; warn the user that history may still contain it (history rewrite only if they explicitly ask).

Do not hard-code one person’s private bookings as skill defaults.

## Site structure
```
aosora-trip/
  CNAME                 # trip.aosora.photography
  .nojekyll
  index.html            # hub: brand + trip links (single-line titles)
  DEPLOY.md
  <trip-slug>/          # e.g. iceland-2027/ , japan-2026/
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
  - Include **itinerary spurs** (day loops, pick-up→stay, return-leg stops)
  - `fitBounds` = **route ∪ stay markers ∪ highlight markers**
  - Load `*-ring-route.js` via `<script>` before main logic; JSON as fetch fallback
  - Simplify dense OSRM output (RDP ~40 m)
- **UX**: progressive enhancement; try/catch around Leaflet; never leave `.reveal` stuck at `opacity: 0`
- **Hub**: brand-first; trip titles one line (`white-space: nowrap` where needed); list trips in chronological order (earliest first)
- **Trip pages**: sticky `.topbar` (back + expand/collapse/print/links) at top; `.day-nav` stays under hero like before. On scroll, when day-nav sticks, topbar slides away (`.is-swapped`) so D1–Dn replace the action bar
- If renaming a trip slug, leave a tiny redirect folder at the old path (meta refresh + `location.replace`)

## Ingest: spreadsheet + bookings
1. Sparse sheet = skeleton (dates / 住 / 租車 / 早午晚). Fill recommended POIs; honor user constraints.
2. Booking screenshots override the sheet for stays, airports, car windows, fixed meals, office days — after PII strip.
3. Sheet CSV/gviz may mojibake in terminals; use browser view or UTF-8 file + `Read` for Chinese.
4. Verify tourism links (no stale 404 locales). Fuji **神玉巡拜** (not 「神珠」):
   - `https://fujiyoshida.net/course/index.php?p=course-530`
   - `https://kamitamajunpai.com/kamitamamap-cat/fujisan/`
5. Keep user semantics (e.g. pilgrimage only in Fuji region; “遺珠” = stops on the drive home).
6. Flights: confirm airport codes; late inbound / outbound buffers.
7. Car: shop **names** + clock times; no reservation number; shop phone optional—default omit.
8. Work / dinner: map pins + times; company name on public HTML only if user explicitly wants it.

## New trip / edit workflow
1. Edit `aosora-trip/<trip-slug>/` — **UTF-8 only** (never UTF-16). On Windows, convert UTF-16 dumps before serve/commit.
2. Copy helpers from an existing trip, replace `P` / `days` / stays / copy.
3. Regenerate OSRM when the path changes (`router.project-osrm.org` OK for personal use).
4. Link from hub `index.html`.
5. Local preview: `python -m http.server` from `aosora-trip/`.
6. **Privacy scan (above)** → deploy. Do not commit previews / scratch scripts unless asked.

## Deploy
1. Commit **only** site files (`CNAME`, `.nojekyll`, hub `index.html`, `<slug>/`) plus skill files if the user asked to update the skill — never PII
2. Pages: `main` / `/ (root)`; custom domain `trip.aosora.photography`
3. Root `CNAME` exactly `trip.aosora.photography`
4. After push: `gh api repos/COYADI/sora-trip-designer/pages/builds -X POST` (needs Pages enabled; private repos need a plan that allows Pages)
5. Verify: `https://trip.aosora.photography/<slug>/`

Windows: `C:\Program Files\Git\bin\git.exe`, `C:\Program Files\GitHub CLI\gh.exe`  
`git commit --trailer "Co-authored-by: Cursor <cursoragent@cursor.com>" -m "message"` (no bash heredocs)

## GoDaddy DNS (zh-TW UI)
**我的產品** → 網域 → **DNS** → **新增記錄**：
- **類型**：CNAME · **名稱**：`trip` · **內容值**：`COYADI.github.io`
- Do not change `@` / `www` (Adobe Portfolio)

## Do not
- Retarget apex/`www` to GitHub Pages while Portfolio is live
- Commit **any** traveler PII anywhere in the repo
- Use ambiguous Google search strings; use `gmaps` place names
- Crop overview bounds so spur stays disappear
- Commit secrets (`.env`, tokens). CARTO keys are client-side — Referer-lock
- Serve UTF-16 site files
- Prefer spreadsheet cities over conflicting hotel/airport/car bookings

## Quick checks
- [ ] Whole-repo privacy scan clean (no emails, names, booking codes, personal phones)
- [ ] Hub + trip titles one line where intended
- [ ] Overview includes major drives + loops + return stops; bounds cover stays
- [ ] Voyager basemap + CARTO attribution
- [ ] Google buttons use place names
- [ ] External links not 404
- [ ] Pages live on custom domain when DNS/plan allow
