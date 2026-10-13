---
name: aosora-trip-itinerary
description: >-
  Build, refine, and publish AoSora travel itinerary HTML sites (bilingual Leaflet +
  CARTO/OSRM maps, Google place-name links) to GitHub Pages at trip.aosora.photography
  (repo COYADI/sora-trip-designer). Use when the user asks for a trip itinerary page,
  aosora trip site, import from spreadsheet/bookings, basemap change, Ring Road / OSRM
  route, GitHub Pages deploy, trip.aosora.photography, budget/saving tips, luggage,
  cash vs card, daily reminder tables, photo golden hour, meals, camera gear, or to
  reuse this skill.
---

# AoSora trip itinerary → GitHub Pages

## Defaults
- Public hub: `https://trip.aosora.photography`
- Host: GitHub Pages from `https://github.com/COYADI/sora-trip-designer` (`main` / root)
- Apex / `www` (`aosora.photography`) stay on Adobe Portfolio — never retarget those DNS records
- Local working tree: `aosora-trip/` (often under the Cursor project). Prefer absolute paths; do **not** call `move_agent_to_root` unless the user asks
- Reply to the user in Traditional Chinese (zh-TW) unless they request another language
- **Gold-standard reference** (full 8-row tip table + all logistics sections): `alps-autumn-2027/`
- For new multi-day / multi-country / photography-leaning trips, **default to the full tip table + logistics sections** below (omit only if the user wants a minimal page)

## Privacy — whole repo (hard rule)
**Scope:** every file under the git root that may be committed or already tracked — hub/trip HTML/JS/JSON, `DEPLOY.md`, `.cursor/skills/**`, comments, scratch files. Not only the trip page.

**Never store or commit the traveler’s personal data:**
- Legal / full names, nicknames used as identity, emails, personal phone numbers
- Booking / reservation / confirmation / PNR / order IDs (hotel, car, restaurant, flight, rail)
- Passport numbers, license numbers, membership IDs, payment method details, card last-4, “paid by …”
- Home address, private URLs, personal calendar invite text, screenshots that embed the above
- Employer-specific internal notes, employee IDs, badge numbers

**OK for a public itinerary (logistics only):**
- Hotel **brand/name**, city, night dates (no confirmation code)
- Car **pick/return shop name + area + clock times** (prefer Google place link; **omit shop phone** unless user asks)
- Restaurant **name + meal time + area**; party size only if useful and user is fine with it
- Airport codes + flight **numbers** / day-of times (remind to re-check schedule)
- Public POIs and tourist addresses
- General **payment advice** (Visa/MC vs cash, pocket CHF/EUR ranges, DCC tip) — never personal card numbers or bank names tied to the traveler
- General **photo/meal/gear advice** (golden hour, supermarket rhythm, lens kit) — no traveler inventory serials or private cloud album URLs
- Workplace: prefer a **district / public landmark building** pin (e.g. 愛宕／MORIタワー). Put a company name on the public site **only if the user explicitly wants it**

**When ingesting booking screenshots / sheets:** extract logistics → **strip all PII and codes** before writing files. Never paste emails or reservation IDs into HTML, JS, MD, or this skill.

### Mandatory scan before every commit / push
From repo root, search tracked + about-to-add files (adjust path). Fail the publish if anything personal matches:

```text
email-like:   @gmail @icloud @yahoo @hotmail @outlook
ids:          預約號碼|訂編|確認番號|confirmation|予約番号|PNR|booking ref
phones:       TEL |Phone |\+886|\+81[0-9\-]{8,}
obvious PII:  passport no|護照號碼|駕照號碼|license no|card last|末四碼
```

Do **not** fail the scan merely because the page says 「信用卡」「刷卡」「腳架」「鏡頭」in general logistics copy.

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
  <trip-slug>/          # e.g. iceland-2027/ , alps-autumn-2027/
    index.html
    <slug>-ring-route.js    # window.PRECOMPUTED_RING_ROUTE = [...]
    <slug>-ring-route.json  # same array (fetch fallback)
```

## Itinerary HTML conventions
- **UI**: dark page chrome; Traditional Chinese; `Noto Sans TC` / `Noto Serif TC`
- **Copy language**: Taiwan zh-TW only (台灣用語). Prefer 齒軌列車（山區小火車）、纜車、取租車、腳踏車道、行動電源、零錢、導覽 — never PRC simplified leftovers (厕所／适合／防晒／街灯／老镇／预留／面包／脚架／爲) or mojibake (e.g. 羆車、零鈞、圖驗). Keep natural TW words: 設備、項目、日歸、菜單、背包、台灣. After any automated edit, spot-check Traditional Chinese.
- **Basemap (default)**: Leaflet + **CARTO Voyager** (light, roads visible)
  - URL: `https://{s}.basemaps.cartocdn.com/rastertiles/voyager/{z}/{x}/{y}{r}.png?key=KEY`
  - Attribution: OpenStreetMap + CARTO (required)
  - Raster CARTO tiles need `?key=` (watermark otherwise). Reuse the key already in the trip’s `darkTiles` / tile helper; do not invent a new key. If missing, ask the user or point them to https://carto.com/basemaps/apikey/
  - Prefer Referer lock on the CARTO dashboard to `trip.aosora.photography` (+ localhost key if needed)
  - Alternatives: [basemaps.md](basemaps.md)
- **Places `P`**: `zh`, `orig`, `lat`, `lng`, `gmaps` (official Google place name)
- **Links**: `mapsPlaceUrl` / `mapsDirUrl` use `gmaps` + `encodeURIComponent` — not raw coordinates. Driving trips: `travelmode=driving`; rail/city-heavy: `transit` when appropriate.
- **Days**: `places` + `route` + optional `highlight` + **all daily tip fields** (below)
- **Stays**: prefer **hotel keys** in `stayKeys` / stay pills when bookings exist (not only city centroids)
- **Overview route**: precompute driving via OSRM into `PRECOMPUTED_RING_ROUTE`
  - Include **itinerary spurs** (day loops, pick-up→stay, return-leg stops)
  - `fitBounds` = **route ∪ stay markers ∪ highlight markers**
  - Load `*-ring-route.js` via `<script>` before main logic; JSON as fetch fallback
  - Simplify dense OSRM output (RDP ~40 m)
- **UX**: progressive enhancement; try/catch around Leaflet; never leave `.reveal` stuck at `opacity: 0`
- **Hub**: brand-first; trip cards use `data-start` / `data-end` (YYYY-MM-DD); JS buckets into **ongoing → future → past** (hide empty sections). Ongoing/future: earliest start first; past: most recent end first
- **Trip pages**: sticky `.topbar` (back + expand/collapse/print + section jump links) at top; `.day-nav` under hero. On scroll, when day-nav sticks, topbar slides away (`.is-swapped`)
- Recommended **topbar jump links** (when sections exist): `#reminders` 提醒 · `#photo-meals-gear` 拍照／餐／設備 · `#money` 現金／刷卡 · `#budget` 省錢攻略 · `#luggage` 行李
- If renaming a trip slug, leave a tiny redirect folder at the old path (meta refresh + `location.replace`)

## Daily tip table (required for multi-day trips)
Each `days[]` object **must** include these short strings (use `"—"` only when truly N/A).

| Field | Column | What to write |
|-------|--------|----------------|
| `save` | 省錢 | Passes vs singles, one-way car fees, stay-location tradeoffs |
| `bag` | 行李／寄放 | Suitcase OK? Hotel hold? Daypack-only mountain day? |
| `book` | 預約／票務 | Tickets/slots to buy or confirm that day |
| `pay` | 付款 | Card vs cash; ATM; local currency; rental deposit notes |
| `photo` | 拍照 | Golden hour / best light; reflection lakes; skip on transfer days |
| `meals` | 三餐 | 早／午／晚 plan (supermarket, hut, early kitchen close) |
| `gear` | 設備 | Lenses, tripod, filters, batteries, shoes; phone-only OK |
| `watch` | 當日提醒 | Weather, last lift, rules, safety, small logistics |

CSS (keep with other styles):

```css
.day-tip-table { margin-top: 1rem; overflow-x: auto; }
.day-tip-table table.route th {
  width: 7.2rem; white-space: nowrap; color: var(--ice); vertical-align: top;
}
.day-tip-table table.route td { color: var(--text-dim); font-size: .88rem; line-height: 1.55; }
```

Renderer (keep in sync with `alps-autumn-2027`):

```javascript
const tipTable = `<div class="day-tip-table card" style="padding:.35rem .6rem;margin-top:1rem"><table class="route"><tbody>
  <tr><th>省錢</th><td>${d.save || "—"}</td></tr>
  <tr><th>行李／寄放</th><td>${d.bag || "—"}</td></tr>
  <tr><th>預約／票務</th><td>${d.book || "—"}</td></tr>
  <tr><th>付款</th><td>${d.pay || "—"}</td></tr>
  <tr><th>拍照</th><td>${d.photo || "—"}</td></tr>
  <tr><th>三餐</th><td>${d.meals || "—"}</td></tr>
  <tr><th>設備</th><td>${d.gear || "—"}</td></tr>
  <tr><th>當日提醒</th><td>${d.watch || "—"}</td></tr>
</tbody></table></div>`;
```

**Placement (gold standard):** tip table goes at the **bottom of `.day-body`**, after map + map-actions — **not** under the stay/drive/daylight stats.

Day-body order:

1. `.stat-row` (住宿／移動／日照參考)
2. `.timeline`
3. notes
4. `.poi-list`
5. `.day-map`
6. `.map-actions`
7. `${tipTable}` ← last

Day section-sub copy should mention all eight columns, e.g. 「每日表格：省錢、行李、預約、付款、拍照、三餐、設備、當日提醒。」

`#reminders` intro should say the daily table already has **八欄** (not 四欄).

## Overview logistics sections (long / multi-country / photo trips)
Add dedicated sections with stable `#` ids + topbar links. Gold-standard page order (`alps-autumn-2027`):

1. Map / stays / hub moves / overnight (as usual)
2. **`#reminders`** — once-per-trip checklist (docs, plugs Type C/J/F, apps, insurance, 112) vs what lives in the daily eight-row table
3. **逐日行程** (`#days` / days root) — each day ends with the tip table
4. **`#photo-meals-gear`** — trip-wide photo timing, kit list, meal rhythm, apps (PhotoPills / Sun Surveyor / MeteoSwiss)
5. **`#money`** — card-first; pocket cash ranges; DCC = local currency; rental needs credit card
6. **`#budget`** — passes, transport, attractions, lodging; biggest bills first
7. Notes / flexibility (optional)
8. **`#luggage`** — segment table suitcase vs daypack; hotel hold > lockers > forward shipping
9. Packing / daily check (optional)

Shorter city trips may omit some overview sections, but still fill daily tip fields when useful.

### Money defaults
- Visa/Mastercard + contactless for hotels, rail, major lifts, supermarkets, city transit
- Pocket cash for toilets, stalls, some meters, remote huts only
- Quote **pocket** ranges (not full trip budget); ATM over airport exchange
- CHF in Switzerland (not EUR); EUR in Eurozone
- Flag one-way international car returns as a top cost lever

### Luggage defaults
- Medium suitcase + daypack; never drag suitcase on hike/lift days
- Car-free towns (e.g. Zermatt): hotel electric station pickup → note in `bag` + `book`
- Multi-night base holds main luggage; car trunk ≠ safe locker

### Photo / meals / gear defaults
- **Photo**: sunrise for lakes/ridges; afternoon side-light for plateaus/passes; soft light for city architecture; no forced golden hour on transfer days; default **no drone**
- **Meals**: supermarket breakfast on hike days; hut or packed lunch; mountain kitchens often close early (~21:00); Sunday closures → restock Saturday
- **Gear**: one body + phone backup; wide + standard zoom covers most; optional 70–200; CPL for water; light tripod for dawn lakes; spare batteries (cold drains fast); say **行動電源** (not 充電寶)
- Per-day `photo` / `meals` / `gear` must be **place-specific**, not only generic boilerplate

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
9. After skeleton days exist, **fill all eight tip fields** — do not leave them empty on long trips.

## New trip / edit workflow
1. Edit `aosora-trip/<trip-slug>/` — **UTF-8 only** (never UTF-16). On Windows, Cursor Write may emit UTF-16: convert with Python `Path.write_text(..., encoding="utf-8")` or decode UTF-16 LE then rewrite. Prefer generating Chinese via Python `\uXXXX` in ASCII-only build scripts, then delete scratch `_*.py`.
2. Copy from `alps-autumn-2027` (tip tables + logistics) or `oktoberfest-2026` / `iceland-2027` (map chrome); replace `P` / `days` / stays / copy.
3. Regenerate OSRM when the path changes (`router.project-osrm.org` OK for personal use).
4. Link from hub `index.html`.
5. Local preview: `python -m http.server` from `aosora-trip/` (cache-bust query if JS data changed).
6. **Privacy scan** → deploy when user wants publish. Do not commit previews / scratch scripts unless asked.
7. Greenfield (“無中生有”) trips: invent sensible dates/season rationale, city-level stays (“飯店名待補”), and **full eight tip rows** + logistics sections; ask before commit/push if user rules require it.

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
- Ship multi-day trips without the eight daily tip fields (especially when user asked for budget / luggage / payment / photo / meals planning)
- Put the tip table under `.stat-row` / before `.timeline` (it belongs at the **bottom** of `.day-body`)
- Leave broken Chinese (simplified mix-ins, 羆車-style garble) in the public page

## Quick checks
- [ ] Whole-repo privacy scan clean (no emails, names, booking codes, personal phones/card numbers)
- [ ] Hub + trip titles one line where intended
- [ ] Overview includes major drives + loops + return stops; bounds cover stays
- [ ] Voyager basemap + CARTO attribution
- [ ] Google buttons use place names
- [ ] External links not 404
- [ ] Daily tip table filled: `save` `bag` `book` `pay` `photo` `meals` `gear` `watch`
- [ ] Tip table rendered **last** in `.day-body` (after `.map-actions`)
- [ ] Logistics sections when relevant: `#reminders` (before days) · `#photo-meals-gear` `#money` `#budget` `#luggage` (after days)
- [ ] Topbar jump links match existing section ids
- [ ] zh-TW copy spot-checked (台灣用語; no mojibake / simplified leftovers)
- [ ] Pages live on custom domain when DNS/plan allow
