# Basemap choices (AoSora trips)

Default for itinerary pages: **CARTO Voyager** (light + roads). Keep page UI dark unless the user asks to restyle.

## Leaflet-ready (raster)

| Style | URL pattern | Roads | Notes |
|-------|-------------|-------|-------|
| **CARTO Voyager** (default) | `.../rastertiles/voyager/{z}/{x}/{y}{r}.png?key=KEY` | Yes | Best light travel look with current CARTO key |
| CARTO Positron | `.../rastertiles/light_all/...png?key=KEY` | Sparse | Ultra-light; route line pops; less road detail |
| CARTO Dark Matter | `.../rastertiles/dark_all/...png?key=KEY` | Soft | Dark; needs key or shows watermark |
| OpenStreetMap | `https://tile.openstreetmap.org/{z}/{x}/{y}.png` | Yes | Busy colors; no key |
| OpenTopoMap | `https://{s}.tile.opentopomap.org/{z}/{x}/{y}.png` | Yes | Terrain/contours; loud for UI |
| OSM HOT | `https://{s}.tile.openstreetmap.fr/hot/{z}/{x}/{y}.png` | Yes | Light humanitarian style |
| CyclOSM | `https://{s}.tile-cyclosm.openstreetmap.fr/cyclosm/{z}/{x}/{y}.png` | Yes | Outdoor/cycling |
| Esri Dark Gray | ArcGIS Canvas Dark Gray Base + Reference | Moderate | No key; previous default |
| Esri Light Gray | ArcGIS Canvas Light Gray Base + Reference | Low | Too sparse for “need roads” |
| Esri Imagery | World Imagery | N/A | Satellite; no road labels |

CARTO free CDN without `?key=` returns **API KEY REQUIRED** tiles (unusable).

## Vector (more detail; more code)

- **OpenFreeMap** + MapLibre: `https://tiles.openfreemap.org/styles/liberty` (light) or `.../dark`
- Use when the user wants denser roads/labels and accepts swapping Leaflet→MapLibre (or a hybrid)

## Decision guide
- Need **roads** + light + minimal change → **Voyager**
- Need **dark** + roads → Dark Matter (key) or OpenFreeMap dark (MapLibre)
- Need **max detail**, OK to change engine → OpenFreeMap
- Never use Google raster tiles inside Leaflet (ToS); Google is for place/dir links only