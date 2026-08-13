# Recipe — Geo Atlas (Atlas open-data hub, reinvented — multi-catalog)

A reproducible path to a **living open-data atlas** on strata-core — **one app, many catalogs**: the
same product serves the **Maryland Geo Atlas** (`?catalog=md`, 189 datasets), the **California Geo
Atlas** (`?catalog=ca`, 2,339 datasets incl. the CAL FIRE and Cal OES hosted orgs), the **Washington DC
Geo Atlas** (`?catalog=dc`, 381 datasets — OCTO/DDOT/DPW/HSEMA/DC Water), and the **Virginia Geo Atlas**
(`?catalog=va`, 563 datasets — VDOT/VGIN/DCR/DEQ/VIMS), each from its own folder under
`app/catalogs/<id>/`. A catalog where **the map is the page**: search or browse the real
datasets, **hover to see any layer live**, pin layers into a
**workbench** that composes a real map, ask **"what data covers my area?"**, open a **dossier** with a
live profile and API playground — and leave with a **working `layers.json`, not a zip file**. Designed
**open-design** from a 2026 study of the portal market (§1): every incumbent is a search box that
occasionally shows a map; this inverts it. Full rationale: `DESIGN-PROPOSAL.md`; the v1 hub-landing recipe
is preserved as `RECIPE-v1-portfolio-hub.md`; v1's DCAT/federation layer is **kept** (§3.3).

> **Scope (honest).** Catalogs reference **live public ArcGIS services**: **md** — Maryland iMap /
> Baltimore / Montgomery (189 datasets; 111 queryable FS, 78 MapServer-only via export-image fallback);
> **ca** — CA Geoportal / Caltrans / DWR / CAL FIRE / Cal OES incl. two hosted AGOL orgs (2,339 datasets;
> 2,291 queryable). Hosted orgs carry no folder taxonomy — CA shelves come from **keyword classification**
> of service names (verify shelf quality per release). Coverage search fans out `returnCountOnly` queries
> — capped at 300 per run on big catalogs (narrow by shelf/search for the rest). Crawl vintage 2026-07-16;
> rerun `build_catalog.py` after re-crawls. Public portal: read-only, no auth, no PII.

---

## 1. Study — how the market frames this (research 2026-07-19)

**Incumbents (benchmark + coexist):** **CKAN**/data.gov (datasets→resources, DataStore previews,
DCAT harvesting — a file cabinet); **Socrata/Tyler** (opendata.maryland.gov — the best *tabular* primer
page: 1,563 datasets, per-column schema, freshness dates; geospatial second-class); **ArcGIS Hub**
(data.imap.maryland.gov — the best *geospatial* dataset page: extent-filtered downloads, API links; but
one gray layer at a time behind a heavy SPA, ISO-jargon categories); **OpenDataSoft** (Explore tabs +
API console — closest to interactive-by-default); **GeoNetwork/CSW** (ISO 19115 card indexes, written
for producers not users).

**The appeal bar (what "alive" looks like):** **Felt** (drag-drop → instant styled shareable map),
**Overture Explorer** (whole catalog renders instantly from PMTiles, click-to-inspect, in-browser
filtered download), **Planetary Computer** (one STAC API drives both the human UI and machine search),
**GEE catalog** (every entry ships an executable snippet), **Source Cooperative** (cloud-native files,
no portal ceremony). Common thread: map first · previews are the actual data · every entry is actionable
in seconds · one artifact serves humans and machines.

**Citable pain (why portals feel dead):** geoportal-discovery study (IJGI 2026): "inefficient search…
inefficient map interactions… inefficient metadata"; spatial-metadata usability (IJGI 2020): standards
serve producers' inventories, not users' choices; Nikiforova & McBride 2021 (41 portals, low usability);
Headd's classic "fancy FTP servers" critique; ODI: "data goes undiscovered; datasets discovered are
unusable; opportunities for connection missed."

**The 12 gaps, ranked** (research §5b): dead previews · no "what covers my area?" · no composition ·
download-and-pray (no profiling) · jargon taxonomy · no freshness/health signals · APIs as links not
playgrounds · static cards · no curation · format friction · no relationships · no feedback loop.
**Maryland concretely:** two portals (Hub + Socrata), two taxonomies, neither composes layers or searches
by coverage — a single fast catalog that does both beats the local incumbents on their home turf.

**Our edge:** the substrate is already live ArcGIS services, so previews, profiling, coverage ranking,
and composition are *queries, not infrastructure* — plus sovereign/MIT on Strata, DCAT federation both
ways (kept from v1), bilingual EN/AR, and the exit artifact is this repo's native contract:
**`layers.json`**.

## 2. UI design spec (front-loaded)

### 2.0 Multi-catalog parameterization (the `?catalog=` contract)
- **One app, N catalogs.** `index.html?catalog=<id>` selects `app/catalogs/<id>/catalog.json`; a header
  switcher swaps catalogs. Everything brand- and geography-specific lives in the catalog's
  **`meta.params`** block — `{id, name, region, tagline, bounds, viewbox (Nominatim), crawl}` — so adding
  a state/city = one config entry in `build_catalog.py` + a source inventory md. No app changes.
- Per-catalog folders can later carry more than `catalog.json` (branding, curated collections, cached
  extents, PMTiles) — the folder IS the catalog package.

### 2.1 Layout (Template: `open-design` — "geo-atlas")
- **Template** `open-design` under the freestyle charter (manifest §10); three §10.3 widget candidates
  (`coverage-search`, `layer-tray`, `service-health`) ship with day-1 fallbacks (§2.6). Anti-collision:
  inverts v1's `portfolio-hub` (map was an embedded row; here the map is the page); nothing shared with
  emergency's `tabbed-workbench` checkbox clearinghouse (floating translucent rail, hover-preview,
  tray, drawer). Harvest candidate name: **`geo-atlas`**.

**Page 1 · ATLAS** (`type:"fixed"` — the product)
```
┌ HEADER: Maryland Geo Atlas · [ search datasets or an address… ] · EN/AR ──┐
│┌ CATALOG RAIL (floating panel, translucent, 340px) ┐                         │
││ shelf chips: Water&Flood 35 · Nature 40 · People…  │   FULL-BLEED MAP       │
││ ▸ Floodplain        ⭓ poly · iMap · [⊕ pin][ℹ]    │  hover card → layer    │
││ ▸ Hospitals         ● point · iMap · [⊕][ℹ]       │  fades in live         │
││ ▸ Traffic Cameras   ● point · iMap · [⊕][ℹ]       │  ⌖ draw-an-area →      │
││ "in view: 23 datasets"                             │  rail re-ranks by      │
│└────────────────────────────────────────────────────┘  actual coverage      │
├──────────────────────────────────────────────────────────────────────────────┤
│ WORKBENCH TRAY: [Floodplain ×][Hospitals ×] → Open as app · Share · Export   │
│ layers.json / GeoJSON                                                        │
└──────────────────────────────────────────────────────────────────────────────┘
   DOSSIER (window, slides over rail): live profile (count · geometry · extent ·
   health · license) → fields → API playground (editable where → live result →
   copyable fetch/Python/layers.json) → downloads → "pairs well with"
```
**Page 2 · HOME** (`type:"scroll"` — first-visit editorial): hero + promise + search; 17 shelf strips
(`flow-row` of live multi-layer preview cards); stats KPIs (189 datasets · 111 live APIs · 4 agencies);
3-step "how it works"; footer feeds. **Page 3 · DEVELOPER**: DCAT-US/RSS/OGC Records, `catalog-md.json`
itself, per-server notes (CORS, `f=geojson` lowercasing, EPSG), playground.
- Responsive: rail → bottom sheet; tray → chip; hover → tap (first tap previews, second pins).

### 2.2 Theme (Chesapeake light, luminous)
`mode:"light"`; primary `#0b7285` (Chesapeake teal), secondary `#f59f00`; translucent floating panels
(backdrop-blur) over a muted light basemap; generous radius; WCAG AA. Preview accents fixed per geometry:
polygon teal fill (~40/255), line indigo, point amber — so any three layers compose legibly. EN + AR/RTL
first-class. *Feels like Felt with a government seal — never like CKAN.*

### 2.3 The interactions that beat every portal (the differentiators)
1. **Hover-to-preview** (kills dead previews): card hover → `/query?f=geojson&outSR=4326&
   resultRecordCount=1500&maxAllowableOffset=<zoom-tuned>` onto the shared canvas; MapServer-only →
   export-image fallback + badge.
2. **Compose & leave with a map** (kills no-composition): ⊕ pins into the tray; "Open as app" emits a
   genuine `layers.json` (exportSpec) → opens in `<StrataMap>`; Share = deep-link (pinned ids + extent).
3. **Coverage search** (kills no-spatial-search): geocode or sketch AOI → fan-out
   `returnCountOnly&geometry=` over the 111 FS (≤10 concurrent, cached extents) → rail re-ranks with
   counts ("Floodplain — 1,240 features here").
4. **Dossier, not metadata page** (kills download-and-pray): live count/fields/extent/health ping +
   playground + filtered downloads + pairs-well-with.
5. **Human shelves** (kills jargon): 17 shelves ("Water & Flood", "Getting Around") mapped from ISO
   folders in `catalog-md.json`; ER/BK priority badges from the solution map.

### 2.4 Wiring (§5 authors these as `AppLayout.connections`)
search→filter rail; address-mode search→`zoomTo`+coverage re-rank; card hover→preview showHide; ⊕→tray
`recordsChange`; ℹ→dossier `showHide`+`viewInTable`; shelf chip `categorySelect`→filter; draw
`sketchComplete`→`selectByGeometry` coverage; map `extentChange`→"in view" `showStatistics`; tray
buttons→`export`/`setUrlParam`/`navigate`; playground→`viewInTable`; downloads→`export`.

### 2.5 Capabilities to use (sweep in DESIGN-PROPOSAL §8)
`RestDataSource` over `catalog-md.json` · search widget + `plugin-search` (Nominatim) · `draw` ·
`filter`/`carto`/`query` · `table`/`feature-info` · `kpi` · `add-data` (bring-your-own layer) ·
`share`/`embed`/`setUrlParam` · `@strata/export` (`exportSpec` = the composed map; `exportLayerData`) ·
`@strata/processing` (extent intersect, aggregate) · panels/windows/flow-row/views · theme + i18n ·
`data-management`/Serve for GeoParquet mirrors · **kept from v1:** DCAT-US/RSS/OGC feeds, schema.org
JSON-LD, `llms.txt`. Deliberately not used: timeslider, routing, swipe, story, gauges (rationale in
proposal).

### 2.6 §10.3 New-widget blocks (fallbacks ship day 1)
1. **`coverage-search`** `{catalogSource,maxFanout}` → emits `recordsChange`. *Fallback:* search + draw →
   fan-out counts → table.
2. **`layer-tray`** `{store}` → emits `recordsChange`/`buttonClick`. *Fallback:* `layer-panel` + `share`
   + an `exportSpec` button.
3. **`service-health`** ping badge. *Fallback:* dossier text row computed on open.

## 3. The catalog (source of truth)

**3.1 Content:** one `app/catalogs/<id>/catalog.json` per catalog, generated by **`build_catalog.py`**
(section-aware parser: agency from server/org, FS+MS merged per service, folder-map shelving for md +
**keyword shelving** for ca's folderless hosted orgs, ER/BK priority tags from each inventory's solution
map, and the `meta.params` app-parameter block of §2.0):
- **md** ← `data_sources_md.md`: 189 datasets · 111 queryable · 17 shelves · iMap/Baltimore/Montgomery.
- **ca** ← `data_sources_ca.md`: 2,339 datasets · 2,291 queryable · 15 shelves · Geoportal/Caltrans/DWR/
  CAL FIRE (incl. hosted org, 1,187 services)/Cal OES (hosted, 993)/Conservation. Fire & Burn alone is
  ~1,146 services — search + coverage scoping do the heavy lifting at this size.
- **dc** ← `data_sources_dc.md` (crawl 2026-07-20): 381 datasets · 82 queryable · OCTO maps2 + em +
  DC Water; folder shelving (DDOT/DPW/HSEMA/OP…) + keywords; the 3,865-service Open Data DC hosted org
  stays a §-noted paging target, not yet cataloged.
- **va** ← `data_sources_va.md` (crawl 2026-07-20): 563 datasets · 414 queryable · VDOT hosted (partial:
  389 of 498 enumerated) + VGIN + VIMS + DCR + DEQ; agency-default + keyword shelving.
Record shape: `{id, title, service, folder, shelf, server, agency,
endpoints{featureServer?,mapServer?}, queryable, preview, tags[emergency-priority|banking-priority]}`.
Rerun the script after each re-crawl (`python3 build_catalog.py [id]`).

**3.2 Enrichment at build time (next iteration):** per-service `?f=json` harvest → description, fields,
extent, maxRecordCount, last-edit → richer records + cached extents for coverage search; nightly health
ping column.

**3.3 Federation (kept from v1):** each record maps to a `CatalogRecord` (`@strata/schema`) → DCAT-US 1.1
(`/api/feed/dcat-us/1.1.json`), RSS, OGC API–Records, per-dataset schema.org JSON-LD, root `llms.txt` —
harvestable by ArcGIS Hub *and* CKAN; only public records federate.

## 4. Verify each URL first (terminal)
```bash
# a queryable FS record (Floodplain — the hover-preview + coverage workhorse):
curl -s "https://mdgeodata.md.gov/imap/rest/services/Hydrology/MD_Floodplain/FeatureServer/0/query?where=1%3D1&returnCountOnly=true&f=json"
# live profile pattern (fields/extent for the dossier):
curl -s "https://mdgeodata.md.gov/imap/rest/services/Health/MD_Hospitals/FeatureServer/0?f=json" | python3 -c "import sys,json;d=json.load(sys.stdin);print(d['name'],d['geometryType'],len(d['fields']))"
# coverage-search unit: count-in-AOI (envelope):
curl -s "https://mdgeodata.md.gov/imap/rest/services/Health/MD_Hospitals/FeatureServer/0/query?geometry=-76.8,39.2,-76.4,39.4&geometryType=esriGeometryEnvelope&inSR=4326&returnCountOnly=true&f=json"
# MapServer-only preview fallback (export image):
curl -s -o /dev/null -w "%{http_code}\n" "https://mdgeodata.md.gov/imap/rest/services/Geoscientific/MD_Soils/MapServer/export?bbox=-77,38.9,-76.5,39.3&bboxSR=4326&size=400,300&format=png&transparent=true&f=image"
```
Gotchas: `f=geojson` lowercases field names; reproject everything `outSR=4326`; throttle coverage
fan-outs; counts drift — the catalog is a vintage, not a promise.

## Guided wizard — the prompts that assign the app's defaults

| # | Wizard question | Options → **default** | Assigns |
|---|---|---|---|
| 1 · App | Title & tagline? | free → **"Maryland Geo Atlas — see it before you download it"** | header + hero |
| 2 · Catalogs | Which catalogs? `[multi]` | **md (189) + ca (2,339)** · one only · my own inventory md | the `app/catalogs/<id>/` set + header switcher |
| 3 · Landing | Default landing? | **Home (first visit) / Atlas (returning)** · Atlas always | page order + remember flag |
| 4 · Shelves | Taxonomy? | **17 human shelves** · raw agency folders | rail chips + Home strips |
| 5 · Coverage | Coverage search? | **on (10-concurrent fan-out, cached extents)** · off | draw + search wiring |
| 6 · Preview caps | Hover-preview budget? | **1500 features + offset generalization** · 500 (slow nets) | query params |
| 7 · MS-only | MapServer-only handling? | **image fallback + badge** · hide them · Serve mirror list | 78 datasets' behavior |
| 8 · Exit | Tray exports? `[multi]` | **layers.json + share-link + GeoJSON** · CSV · GeoParquet (Serve) | tray buttons |
| 9 · Feeds | Federation? | **DCAT-US + RSS + OGC Records + llms.txt** · off | Developer page |
| 10 · Theme | Theme + language? | **Chesapeake light / EN+AR** · dark · EN | ThemeSpec + lang-switch |

**Then:** echo *"189 datasets · human shelves · coverage on · 1500-cap previews · image-fallback ·
layers.json+share+GeoJSON · full feeds · light EN+AR"* → confirm → §5.

## 5. Prompt-script (run in order)
```
A. python3 build_catalog.py — (re)generate app/catalogs/{md,ca}/catalog.json; confirm md 189/111/17 and
   ca 2339/2291/15 counts. The app reads ?catalog=<id> (default md) and brands itself from meta.params.

B. /new-app — "Maryland Geo Atlas", Template: open-design per RECIPE §2.1: pages atlas (fixed),
   home (scroll), developer; Chesapeake light ThemeSpec (§2.2) + AR/RTL; header search (dataset +
   address modes). Register catalog-md.json as a RestDataSource "catalog".

C. ATLAS page: full-bleed map section; floating translucent catalog rail (panel): shelf chips
   (categorySelect→filter), dataset cards bound to "catalog" (title · geometry glyph · agency · ⊕ · ℹ ·
   priority badges); "in view: N" chip (extentChange→showStatistics).

D. Hover-preview engine: card hover → fetch the record's FS /query (f=geojson, outSR=4326,
   resultRecordCount≤1500, maxAllowableOffset by zoom) → styled by geometry accent (§2.2) → fade in;
   hover-out removes unless pinned. MapServer-only → /export image overlay + "image preview" badge.

E. Workbench tray (layer-tray fallback: layer-panel + buttons): ⊕ pins the layer; tray shows chips ×;
   [Open as app] → exportSpec composes a genuine layers.json and opens it in <StrataMap>; [Share] →
   setUrlParam deep-link (ids+extent); [Export] → merged GeoJSON / per-layer CSV.

F. Coverage search (coverage-search fallback): address geocode (plugin-search) or draw AOI
   (sketchComplete) → fan out returnCountOnly&geometry over queryable FS (≤10 concurrent, cached
   extents prefilter) → re-rank rail with per-dataset counts; extent-test MS-only entries.

G. Dossier (window over the rail): live profile (?f=json + count + health ping), field table,
   API playground (where/outFields editor → viewInTable + copyable fetch/Python/layers.json snippet),
   downloads (exportLayerData GeoJSON/CSV; GeoParquet when Serve-mirrored), "pairs well with"
   (same-shelf + extent-overlap heuristic).

H. HOME page: hero + search; 17 shelf strips (flow-row of cards w/ live mini-previews, lazy-loaded);
   stats KPIs; 3-step how-it-works; footer feeds. DEVELOPER page: DCAT-US/RSS/OGC Records feeds,
   llms.txt, catalog JSON, per-server notes, playground.

I. Federation (kept from v1): render records → CatalogRecords → the feeds; per-dataset JSON-LD;
   only-public rule. Bilingual QA; a11y pass (rail keyboard nav, prefers-reduced-motion).

J. Verify §6; log gaps to §7.
```

## 6. Verify (benchmark to the §1 market)
| Check | Pass |
|---|---|
| `?catalog=md` and `?catalog=ca` both boot, re-branded (name, bounds, geocoder, dev page) from meta.params | ☐ |
| Hover a card → the real layer renders on the canvas in <1.5 s, styled and labeled | ☐ |
| Pin 3 layers → tray composes them; [Open as app] yields a working layers.json; share-link restores it | ☐ |
| Draw an AOI → rail re-ranks with real counts; "in view" chip tracks the extent | ☐ |
| Dossier shows live count/fields/extent/health; playground query round-trips; snippet paste-works | ☐ |
| 17 human shelves; zero ISO jargon user-facing; ER/BK badges present | ☐ |
| MapServer-only entries preview as images with an honest badge (78 of 189) | ☐ |
| DCAT-US validates; CKAN/Hub can harvest it; JSON-LD + llms.txt present (v1 parity) | ☐ |
| EN/AR RTL mirrors fully; AA contrast on the translucent rail | ☐ |
| Beats the §1 bar: no incumbent has hover-preview + composition + coverage search + playground on one canvas | ☐ (judge) |

## 6.5 Note — Strata Serve path (same pattern as branch-atm §6.5)
Mirror the priority datasets (ER/BK-tagged + top shelves) to GeoParquet via DuckDB → publish on Serve →
instant previews under our SLA, GeoParquet downloads become real, MS-only gaps close, and the atlas keeps
working if iMap throttles. The catalog record gains a `mirror` endpoint field; DC GIS/Overture variants
of `build_catalog_*.py` make this portable to any city/state.

## 7. Harvest (gaps → strata-core)
- **`coverage-search`**, **`layer-tray`**, **`service-health`** widgets (shipped as fallbacks here) —
  §10 promotion candidates; the whole silhouette is a **`geo-atlas`** template candidate after reuse.
- **DCAT feed generator + JSON-LD helper** (carried from v1 — still unbuilt in core).
- **PMTiles pre-render pipeline** for instant previews of heavy layers (Overture pattern).
- A **catalog-crawler skill** (`services?f=json` recursion → data_sources md + catalog JSON) so any
  ArcGIS server becomes an atlas in minutes.

## 8. Sources
- Incumbents: CKAN data-viewer/DataStore/ckanext-dcat docs · data.gov Catalog API · Socrata Primer +
  catalog docs · ArcGIS Hub explore/download/federation docs · OpenDataSoft Explore API v2.1 + console ·
  GeoNetwork ISO 19139 docs · MD iMap training PDF (geodata.md.gov) · opendata.maryland.gov catalog API
  (live-verified 2026-07: 3,641 assets / 1,563 datasets).
- The appeal bar: Overture Explorer blog (PMTiles+GERS) · Felt · Atlas.co · Planetary Computer STAC
  docs + Element 84 analysis · Source Cooperative (Radiant Earth) · GEE catalog pattern.
- Critiques: IJGI 2026 geoportal-discovery study · IJGI 2020 spatial-metadata usability ·
  Nikiforova & McBride 2021 (Telematics & Informatics) · Máchová et al. 2018 · Headd "I Hate Open Data
  Portals" (civic.io) · ODI data-portals report · Neumaier et al. (ACM JDIQ).
- Data: `.private/data_sources/data_sources_md.md` (2026-07-16 crawl) → `catalog-md.json`
  (`build_catalog_md.py`); Maryland iMap / Baltimore / Montgomery REST endpoints (§4-verified).
- Internal: `DESIGN-PROPOSAL.md` · `RECIPE-v1-portfolio-hub.md` (kept: DCAT/feeds/JSON-LD design) ·
  `strata/recipes/COMPONENT-MANIFEST.md` §10 · `strata/docs/guide/app-design.md`.

---

## Modernization (parity release)
> Native to this design: `RestDataSource` catalog binding · floating `panel`/`window`/`flow-row` shells ·
> `connections`-first interactivity (hover/pin/coverage loops) · structured light theme + AR/RTL ·
> `exportSpec` as the product's exit artifact · `setUrlParam` deep-links.
