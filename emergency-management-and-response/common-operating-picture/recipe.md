# Recipe — The Picture (Emergency Common Operating Picture, reinvented)

A reproducible path to an **incident-anchored Common Operating Picture** on strata-core, built on the
full California data inventory: a left **incident rail**, a full-bleed map where the selected incident's
**picture assembles itself** (perimeter, damage, shelters, evacuation zones, closures, cameras — each
with a live **freshness badge**), a right **dossier** carrying a **verified update feed** and a
one-click **SITREP generated from the live state**, an **operational-period scrubber** to replay the
picture, a **catalog drawer** exposing all **2,339 CA services**, and a **public mode** that publishes
the same picture. Designed **open-design** from a 2026 study of the COP market **beyond ESRI** (§1) —
the thesis: *fuse Watch Duty's incident-anchored trust with an EOC's authority, map-native.* Full
rationale: `DESIGN-PROPOSAL.md`; the v1 KPI-board COP is preserved as `RECIPE-v1-ops-command.md`.

> **Scope (honest).** All geodata is **live public California services** cataloged in
> `cop-catalog-ca.json` (from `data_sources_ca.md`, crawl 2026-07-16): 136 core services auto-tagged
> into 13 operational roles + the 2,339-service library + 4 national feeds (FIRMS, WFIGS, NWS CAP,
> USGS — keyless). **Shelter/evacuation feed coverage varies by county** — the assembly must say "no
> feed here" honestly. The **verified-update feed and operational-period snapshots are app-side**
> (synthetic in the template; writable on an ESRI backend via `updateRecord` 🔶, read-only elsewhere).
> Genasys/ALERTCalifornia API licensing unverified — use Cal OES evacuation layers + Caltrans CCTV.
> Notification discipline is a design rule, not a feature flag: publish only on life/property relevance.

---

## 1. Study — the COP market beyond ESRI (research 2026-07-20)

**The consumer benchmark — Watch Duty:** 7.2M yearly users (8M+ in one week during the Jan-2025 LA
fires; #1 on the App Store after the county's false evacuation WEA); map-first, **incident-anchored**
(incident page = summary + geometry + reverse-chron **human-verified update feed**); ~300 volunteer
dispatchers/firefighters monitoring radio/cameras/satellites; notifies **only on life/property threat**
under a published code of conduct; free, no login, five languages. CAL FIRE's "not an official source"
caution is itself the market gap: officials fragment across CAL FIRE/sheriff-Facebook/Nixle/WEA.

**The operational benchmarks:** **Genasys Protect/Zonehaven** — pre-planned evacuation zones with a
status lifecycle (Advisory → Warning → Order → Lifted / Shelter-in-Place), agency console + public
"Know Your Zone" map. **Perimeter** — one authoring surface, two audiences (agency + public).
**TAK/ATAK** — every unit a PLI dot on a shared picture; geofence triggers; event-based sync.
**EOC platforms** (WebEOC boards/Significant Events/SITREP cycle, Everbridge asset-impact map, Hexagon
Smart Advisor, D4H status boards, Noggin, Veoci): every "COP" is **a dashboard of boards with a map
widget** — truth in tables, the map a viewport. **Public viewers** worth stealing from: FIRMS (age-
encoded hotspots + time slider), QuickMap (1-min CHP feed, per-layer cadence, CCTV/CMS), ALERTCalifornia
(1,000+ cameras, AI detections beating 911 >50% of the time), PurpleAir, PowerOutage.us (timelapse),
NWS CAP (severity/certainty/urgency), InciWeb (page-of-record), MapAction (standard map products),
Sahana (shelters as registries), Ushahidi (verification queue).

**The 12 gaps this design occupies:** map-as-widget · no editorial narrative tier · notification blast
vs discipline · freshness/provenance opacity · no single incident anchor · evac lifecycle bolted-on ·
login walls (no public path) · cameras/sensors absent · form-era UX · SITREP as duplicate typing · no
temporal replay · no PLI. **Our edge:** boards become views of map features; freshness is a pixel;
the SITREP is generated; the public view is the same picture filtered; and the whole 2,339-service CA
inventory is one drawer away — sovereign, MIT, on Strata or ArcGIS.

**Standards & orgs:** ICS operational periods & SITREP/IAP conventions; CAP v1.2 (NWS alerts); NIFC/
WFIGS refresh model; FIRIS (Cal OES IR aircraft → Intterra); DINS damage standards; Cursor-on-Target
(TAK) as the event-sync reference; WEA lessons (Jan-2025 LA false alert).

## 2. UI design spec (front-loaded)

### 2.1 Layout (Template: `open-design` — "incident-anchor")
- **Template** `open-design` under the freestyle charter (manifest §10); three §10.3 widget candidates
  (`incident-feed`, `freshness-badge`, `period-scrubber`) ship with day-1 fallbacks (§2.6).
  Anti-collision: v1 `ops-command` retired (no KPI-strip protagonist here); distinct from every
  emergency sibling (scoreboard, launchpad, media-pager, zone-lookup, tabbed-workbench, time-player,
  chart-board, ranked-list, compare-swipe, scroll-story, sidebar-explorer). Harvest candidate:
  **`incident-anchor`** (reusable for outages/spills/security incidents).

```
┌ HEADER: ● LIVE · hazard chips (fire/flood/quake) · op-period ▾ · wall · PUBLIC/EOC ┐
│ INCIDENT RAIL (300px)   │        FULL-BLEED MAP           │ DOSSIER (380px)        │
│ ▸ Creek Fire      ⬤2m   │  perimeters (FIRIS, age-tinted) │ CREEK FIRE · 4,210 ac  │
│   4,210 ac · 15% ▂▄     │  hotspots (FIRMS, age-fade) ·   │ ▍15% contained gauge   │
│ ▸ Ridge Fire      ⬤7m   │  evac zones (lifecycle color) · │ ── PICTURE (auto) ──   │
│   890 ac · 60%          │  shelters ⌂ · closures ✕ ·      │ perimeter ⬤2m · DINS   │
│ ▸ River Flood     ⬤31m  │  cameras ▣ · quakes ○           │ ⬤14m · shelters 3 ⬤9m  │
│ sort: threat ▾          │  selected incident spotlit;     │ · closures 2 · cams 4  │
│ ── CATALOG DRAWER ──    │  rest of state dimmed           │ ── VERIFIED FEED ──    │
│ + 2,339 CA layers…      │                                 │ 14:22 ✎ zone E12→ORDER │
│                         │                                 │ 13:58 ✎ perim +340 ac  │
│                         │                                 │ [＋ post] [SITREP PDF]  │
├─────────────────────────┴─────────────────────────────────┴────────────────────────┤
│ OP-PERIOD SCRUBBER: ◄ P-3 · P-2 · P-1 · ▶ NOW — replay perimeters/zones/feed       │
└────────────────────────────────────────────────────────────────────────────────────┘
```
- **Pages:** `picture` (EOC, fixed) + `public` (same layout, chrome stripped, EOC-only layers filtered,
  default on phones). Responsive: rail → bottom cards; dossier → full-screen incident sheet.

### 2.2 Theme (hazard dark; freshness is a token)
Dark ops base `#0d1117`; map ≥70% of pixels (never a KPI board). Hazard palette: active `#e34a33`,
perimeter `#991e14`, FIRMS hotspots age-faded; **evac lifecycle** advisory `#f5b83d` / warning
`#fd8d3c` / order `#e03131` / lifted `#2b8a3e` (arcade-classed, shared by fills and chips).
**Freshness token everywhere:** dot green <5 m · amber <30 m · red stale, with "updated Xm ago" text —
per feed, per dossier row, per rail card. Tabular numerals; wall mode; EN + AR/RTL; public mode flips
to a light, calm publication theme.

### 2.3 The interactions that beat both markets (differentiators)
1. **Incident-anchored assembly** (kills no-anchor + map-as-widget): rail `rowSelect` → `zoomTo` +
   spotlight (dim the rest) + the dossier fans out the 13 roles **within the incident radius**
   (`withinDistance`/`pointsWithin` over role-tagged services) — perimeter, DINS, shelters, zones,
   closures, cameras, hospitals — each row with count + freshness badge + "no feed here" honesty.
2. **Verified update feed** (kills no-editorial-tier): an attributed, timestamped editorial layer above
   the raw log; posting gated (🔶 `updateRecord` on ESRI backend; local+toast elsewhere); the
   notify-on-threat discipline written into the recipe.
3. **Generated SITREP** (kills duplicate typing): one click renders the briefing PDF from live state —
   incident map (legend/scalebar/north-arrow), zone statuses, feed excerpts, period diff. MapAction-
   style map-pack export per incident.
4. **Operational-period replay** (kills no-replay): the scrubber steps P-3…NOW; `views` mapStates apply
   period filters to perimeters/zones; feed scopes to the period — after-action continuity built in.
5. **The drawer** (unique to us): all **2,339 CA services** (shelved, searchable) pinnable into the
   picture — the COP and the Geo Atlas share one catalog spine.

### 2.4 Wiring (authored as `AppLayout.connections` — §5 implements)
rail `rowSelect` → map `zoomTo`+`flash` + dossier `viewInTable` + role fan-out; hazard chips
`categorySelect` → `filter` rail+map; scrubber `viewChange` → mapState apply; map `featureSelect` →
dossier context; drawer `rowSelect` → `showHide` pin; `timer` → `refresh` freshness; post `buttonClick`
→ `updateRecord` 🔶/`message`; SITREP `buttonClick` → `export`; PUBLIC/EOC `buttonClick` → `navigate`;
draw `sketchComplete` → `selectByGeometry` (spot request); shelter row → routing `nearest`+route.

### 2.5 Capabilities to use (full sweep in DESIGN-PROPOSAL §8)
`RestDataSource(cop-catalog-ca.json)` + per-role sources · processing (`withinDistance`, `pointsWithin`,
`buffer`, `hotspot`, `aggregate`) · plugin-routing (nearest shelter + route) · timeslider plugin (FIRMS
age / period) · search plugin · export (SITREP PDF, map pack, spec) · arcade (lifecycle + freshness
classing) · `views`+`mapState` (scrubber) · panels/windows · theme+i18n · timer→refresh ·
feature-arcgis/auth 🔶 (editorial writes). Deliberately not used: swipe/story (after-action's job),
weighted-overlay (mitigation's job), reporter (public submissions = future Ushahidi tier).

### 2.6 §10.3 New-widget blocks (fallbacks ship day 1)
1. **`incident-feed`** `{incidentId, canPost}` → emits `rowSelect`. *Fallback:* newest-first `table`
   over an updates layer + `feature-info`; posting via `updateRecord` 🔶 or local store.
2. **`freshness-badge`** — *Fallback:* `text` chip + `timer → refresh` + arcade thresholds.
3. **`period-scrubber`** — *Fallback:* `views` (nav:"slides") whose `mapState`s encode period
   `definitionExpression`s.

## 3. The data catalog (source of truth)

**`cop-catalog-ca.json`**, generated by **`build_cop_catalog.py`** (reuses the Geo Atlas CA parser —
one inventory, two products): **library 2,339** services (shelved, searchable — the drawer) of which
**136 are core, auto-tagged into 13 roles**: perimeters (20, incl. yearly archives feeding the
scrubber) · damage/DINS (57) · fire-hazard/FHSZ (25) · evacuation (7) · mutual-aid (6) · boundaries (4)
· contacts (4) · shelters (3) · hospitals (3) · roads (3) · flood (2) · cameras (1) · seismic (1); plus
**4 national feeds** (FIRMS hotspots, WFIGS perimeters ~5-min, NWS CAP `api.weather.gov`, USGS quakes —
all keyless GeoJSON/REST). Rerun after re-crawls: `python3 build_cop_catalog.py`. Role rules are regex
on service names — extend `ROLES` as feeds evolve.

## 4. Verify each URL first (terminal)
```bash
# live FIRIS/NIFC perimeters (the rail's source) — count + newest:
curl -s "https://services1.arcgis.com/jUJYIo9tSA7EHvfZ/arcgis/rest/services/CA_Perimeters_NIFC_FIRIS_public_view/FeatureServer/0/query?where=1%3D1&returnCountOnly=true&f=json"
# Cal OES statewide evacuation layer (lifecycle field names drive the arcade classing — inspect!):
curl -s "https://services.arcgis.com/BLN4oKB0N1YSgvY8/arcgis/rest/services/Combined_Statewide_Evacuation_Layer_view/FeatureServer/0?f=json" | python3 -c "import sys,json;d=json.load(sys.stdin);print(d.get('name'),[f['name'] for f in d.get('fields',[])][:12])"
# NWS CAP alerts for CA (keyless; severity/certainty/urgency):
curl -s "https://api.weather.gov/alerts/active?area=CA" -H "accept: application/geo+json" | head -c 300
# USGS quakes day feed:
curl -s "https://earthquake.usgs.gov/earthquakes/feed/v1.0/summary/all_day.geojson" | head -c 200
```
Gotchas: hosted AGOL `f=geojson` field-name casing varies; always `outSR=4326`; FIRIS/WFIGS layer ids
drift — verify at build; per-fire DINS services are event-scoped (empty between fires — the freshness
badge, not an error, tells that story).

## Guided wizard — the prompts that assign the app's defaults

| # | Wizard question | Options → **default** | Assigns |
|---|---|---|---|
| 1 · App | Title? | free → **"The Picture — California COP"** | header |
| 2 · Hazards | Hazard set? `[multi]` | **fire + flood + quake** · fire only | chips + role subset |
| 3 · Anchor | Incident source? | **FIRIS/NIFC public view + WFIGS** · my incident layer | rail binding |
| 4 · Assembly | Picture radius? | **15 km** · 8 · 30 | role fan-out distance |
| 5 · Periods | Operational periods? | **12 h ICS, 4 kept** · 24 h | scrubber snapshots |
| 6 · Feed | Verified-update store? | **local (template)** · my ESRI updates layer 🔶 | feed backend + posting |
| 7 · Public | Public mode? | **on (default on mobile)** · off | the `public` page |
| 8 · Drawer | Catalog drawer? | **full 2,339 library** · core-only | drawer source |
| 9 · Outputs | Exports? `[multi]` | **SITREP PDF + map pack + spec** | export wiring |
| 10 · Theme | Mode/lang/wall? | **hazard-dark / EN+AR / wall** · light | ThemeSpec |

**Then:** echo *"CA · fire+flood+quake · FIRIS+WFIGS · 15 km assembly · 12 h periods · local feed ·
public on · full drawer · all exports · dark EN+AR"* → confirm → §5.

## 5. Prompt-script (run in order)
```
A. python3 build_cop_catalog.py — (re)generate cop-catalog-ca.json; confirm 2339/136/13 counts.

B. /new-app — "The Picture — California COP", Template: open-design per §2.1: pages picture + public;
   hazard-dark ThemeSpec (§2.2) with the evac-lifecycle + freshness arcade tokens; header (LIVE clock,
   hazard chips, op-period ▾, wall, PUBLIC/EOC). Register cop-catalog-ca.json as RestDataSource.

C. The map spine: perimeters (FIRIS public view, age-tinted stroke) + FIRMS hotspots (age-faded dots) +
   evac zones (lifecycle fills) + shelters/closures/cameras/hospitals/quake feeds from the role-tagged
   core; NWS CAP polygons on demand. Per-layer freshness metadata captured at fetch (§2.6 fallback).

D. INCIDENT RAIL: cards bound to the perimeter/incident source — name, acreage, containment chip
   (gauge-in-card), spark, freshness dot; threat sort (uncontained × acreage × proximity-to-people via
   pointsWithin tracts); rowSelect → zoomTo + spotlight (dim non-selected via opacity filter) + dossier.

E. ASSEMBLY: on incident select, fan out the 13 roles withinDistance(radius Q4) → dossier PICTURE rows
   (count + freshness + "no feed for this county" when a role returns nothing); nearest open shelter
   gets a route (plugin-routing) drawn on demand.

F. DOSSIER: incident header (acres/containment/updated); PICTURE section (E); VERIFIED FEED
   (incident-feed fallback: newest-first table + post button — updateRecord 🔶 else local+toast; the
   notify-discipline note rendered in the composer); [Generate SITREP] → export PDF (map + zone table +
   feed excerpt + period diff); map-pack export per incident.

G. OP-PERIOD SCRUBBER: views(nav:"slides") P-3…NOW; each view's mapState applies period
   definitionExpressions to perimeters (yearly/archive services) + zones + scopes the feed; wall mode
   enlarges rail cards.

H. CATALOG DRAWER: window over the rail listing the 2,339-service library (search + shelf chips);
   rowSelect pins/unpins into the picture (showHide) — shared spine with the Geo Atlas.

I. PUBLIC page: same layout, stripped chrome, EOC-only layers filtered, light calm theme, zone search
   ("know your zone" hand-off to the evac recipe's app where deployed); default on small screens.

J. Verify §6; log gaps to §7.
```

## 6. Verify (benchmark to Watch Duty + the EOC field, not just Esri)
| Check | Pass |
|---|---|
| Select an incident → picture assembles <3 s: roles, counts, freshness badges; absent feeds say so | ☐ |
| Freshness honest everywhere: dots + "Xm ago" per feed/card/row; stale turns red, never hides | ☐ |
| Verified feed: attributed, timestamped, newest-first; posting gated; discipline note visible | ☐ |
| SITREP PDF generated from live state (map + zones + feed + period diff) — zero re-typing | ☐ |
| Scrubber replays perimeters/zones/feed across ≥3 periods | ☐ |
| Evac lifecycle colors (advisory/warning/order/lifted) consistent between fills, chips, and legend | ☐ |
| Drawer: search 2,339 services, pin any into the picture; shared catalog spine with the Geo Atlas | ☐ |
| Public mode: same picture filtered, light theme, mobile default; no login anywhere | ☐ |
| National feeds live: FIRMS age-fade, NWS CAP severity fields, USGS quakes | ☐ |
| Writes only on ESRI backend; read-only path still demos (local feed + toast) | ☐ |
| Beats the bar: incident anchor + freshness pixel + generated SITREP + replay + 2,339-layer drawer exist together in NO incumbent (Watch Duty, WebEOC, Genasys, D4H) | ☐ (judge) |

## 7. Harvest (gaps → strata-core)
- **`incident-feed`**, **`freshness-badge`**, **`period-scrubber`** widgets (shipped as fallbacks) —
  §10 promotion candidates; the silhouette is an **`incident-anchor`** template candidate (outages,
  spills, security).
- A **role-tagged catalog** convention (`role:` field) worth promoting into the Geo Atlas catalog spine.
- **Feed-freshness capture** at the DataSource layer (last-modified/ping) — belongs in
  `@strata/data-source`.
- **PLI ingestion** (TAK/CoT bridge) and an **Ushahidi-tier public-report inbox** — future waves.
- Snapshot/diff store for period replay (today: mapState filters; production wants persisted snapshots).

## 8. Sources
- **Consumer/ops products:** Watch Duty (watchduty.org how-it-works/blog; Wikipedia; AP/KPBS/CBS/NBC
  Jan-2025 LA coverage) · Genasys Protect/Zonehaven (genasys.com; San Mateo/Napa/Lake county pages) ·
  Perimeter (perimeterplatform.com; GovTech) · TAK/ATAK (civtak.org; Hackaday deep-dive).
- **EOC platforms:** Juvare WebEOC standard-boards docs · Everbridge Visual Command Center docs ·
  Hexagon HxGN OnCall Smart Advisor · D4H status boards · Noggin · Veoci COP pages.
- **Public viewers/feeds:** NASA FIRMS (Earthdata) · NIFC/WFIGS open data (~5-min perimeters) · InciWeb ·
  CAL FIRE DINS on data.ca.gov · Cal OES FIRIS program page · ALERTCalifornia (UCSD; Route Fifty 2026) ·
  Caltrans QuickMap FAQ (1-min CHP feed) · api.weather.gov CAP docs · USGS GeoJSON feeds · PurpleAir ·
  PowerOutage.us · Windy.
- **Open-source/humanitarian:** Sahana Eden · Ushahidi · MapAction product catalogue.
- **Data:** `.private/data_sources/data_sources_ca.md` (2026-07-16 crawl) → `cop-catalog-ca.json`
  (`build_cop_catalog.py`, reusing `../geodata-management_atlas-open-data-hub/build_catalog.py`); `data_sources_national.md`.
- **Internal:** `DESIGN-PROPOSAL.md` · `RECIPE-v1-ops-command.md` (retired KPI board) ·
  `STRATA-CORE-ISSUES.md` · `strata/recipes/COMPONENT-MANIFEST.md` §10 · `strata/docs/guide/app-design.md`.

---

## Modernization (parity release)
> Native to this design: `RestDataSource` catalog spine · `views`+`mapState` period replay ·
> `timer → refresh` freshness · arcade lifecycle/freshness classing shared by symbology and chips ·
> panels/windows shells · export-from-state SITREP · two-page public publication path.
