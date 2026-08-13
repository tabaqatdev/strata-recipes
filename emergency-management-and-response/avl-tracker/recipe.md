# Recipe — Resource & Asset (AVL) Tracker (Emergency)

A reproducible path to a **live resource / asset tracker** on strata-core: plot **units, vehicles, crews and
equipment** from an AVL/GPS feed, answer *"where is everything, what's its status, and which unit is nearest
to this incident?"* — a near-real-time who/where/status operational picture with **nearest-resource-to-incident**
ranking. This is the AVL cluster's lead app in the Emergency programme (`Emergency-Response-Watch-Center.md`),
sibling to the wildfire COP: the COP watches *incidents*, this watches *responders*.

> **Scope (honest).** A **serving/analytics layer over an existing AVL/GPS feed** — not a CAD, not a dispatch
> system of record, not a radio gateway. Positions are **read on a refresh interval** (near-real-time), not a
> streaming socket; strata does not ingest raw NMEA/telemetry — point it at a **published live FeatureServer**
> (Velocity/GeoEvent, Field Maps Location Tracking, or a vendor AVL layer). "Nearest unit" is a
> **distance/`nearest`** rank by default; true **drive-time (Closest Facility)** needs the routing plugin and a
> routing service — offered, not assumed. Reads work on Strata **and** ArcGIS; **no editing** (read-only today).

Run the §5 prompt-script on a fresh strata-core project (or via `/new-app`). ESRI Web Map JSON is the map contract.

---

## 1. Study — how the market frames this

**The question a dispatcher / watch officer asks:** *where is every unit right now, what's its status (available/
enroute/on-scene/out-of-service), and which available unit is closest to this incident?* The pattern is
**AVL (Automatic Vehicle Location)** + **nearest-unit** + a live **status board** — the daily workflow of a 911
center or an EOC resource desk.

**Reference solutions (benchmark + coexist, never copy):**
- **Esri** assembles this from parts, not one product: **ArcGIS Velocity** / legacy **GeoEvent Server** stream
  AVL/GPS into live feature layers; **ArcGIS Field Maps "Location Tracking"** gives personnel/crew breadcrumbs
  (auto-creates *Last Known Locations* / *Tracks* / *Track Lines* with `device_id`, `activity`,
  `battery_percentage`, `speed`, `course`, `location_timestamp`); **Network Analyst "Closest Facility"**
  (`solveClosestFacility`) ranks nearest units by travel time; **ArcGIS Dashboards** surfaces the live picture.
  Real reference architectures: **Manassas Park, VA** (Cradlepoint routers → Event Hubs → Velocity AVL layer,
  refresh every few seconds, position persists 30 min after shutdown) and **Lafayette Parish, LA 911** (GeoEvent
  streaming positions into CAD/dispatcher maps) — the exact "near-real-time via refresh" pattern this app needs.
- **Purpose-built AVL on top of Esri:** **BCS MARVLIS** (an "Impedance Monitor" learns true road speeds from
  historical fleet AVL so nearest-unit accounts for real traffic/coverage/priority, not just proximity);
  **CompassCom CompassTrac / CompassAVL** (tracks LMR radios — Motorola/BK — GPS modems and vehicles with
  geofencing, panic alerts, incident replay). **Track Star** (fleet AVLS rebuilding on ArcGIS). **Blue Raster**
  built the Manassas Park Velocity solution.
- **CAD/AVL incumbents (adjacent, not GIS-native):** Motorola CommandCentral / ASTRO 25, Hexagon I/CAD, Tyler,
  CentralSquare — full dispatch stacks whose AVL/unit-status this app can *consume* rather than replace.

**Our edge:** AI-authored, MIT/open template, runs on **Strata *or* ArcGIS**, **sovereign/on-prem** (unit
positions + PII never touch a cloud LLM), and a **great status board + nearest-unit + KPIs** out of the box —
the operational picture without the CAD price tag.

**Standards, specifications & organizations to speak fluently:**
- **NENA** — NG9-1-1 GIS Data Model (**NENA-STA-006** — road centerlines, address points, PSAP boundaries:
  the authoritative address/boundary base), CAD-to-CAD interoperability, and the AVL definition (location
  data transmitted vehicle → consuming system). **EIDO** (Emergency Incident Data Object) — the JSON incident
  object carrying the incident *plus its assigned responders, vehicles and equipment* — is **NENA-STA-021**,
  conveyed by **NENA-STA-024** over WebSockets (**not** STA-006); it is the CAD-interop ingress if you ingest
  live unit status from a customer's CAD.
- **OASIS Emergency Management TC** — **EDXL-DE** (distribution/routing envelope) and **EDXL-RM** (Resource
  Messaging: request/offer/commit for mutual-aid resources) — the standard for cross-jurisdiction *resource
  requests*, distinct from live position but relevant if you later model "requested/committed" status.
- **Project 25 (P25)** — the interoperable Land Mobile Radio suite (with GPS/AVL data-channel provisions),
  originated by APCO and standardized through the **TIA** TR-8 committee; NENA is a separate NG9-1-1 body,
  not a P25 standards author.
- **FirstNet / Band 14** (FirstNet Authority, AT&T-operated, FCC spectrum) — dedicated public-safety broadband
  carrying AVL/location telemetry with priority/preemption.
- **FEMA/DHS** NIMS context; **Cal OES** (mutual-aid regions + a live Fire & Rescue Fleet layer); **CAL FIRE**
  (live aircraft-tracking public view); **NIST PSCR** (P25 conformance). **Esri Web Map JSON** is the schema
  strata renders (`operationalLayers`, `drawingInfo`, `popupInfo`).

## 2. UI design spec (front-loaded)

The tracker is **control-room grade** — this spec is the visual contract; §5 implements it.

### 2.1 Layout (Template `launchpad` — Launchpad)
```
┌ MAP (full-bleed): units by status · live positions ──────────────────────┐
│  ┌ floating FOLLOW card: unit E-14 · fix age 0:32 · ✕ ┐                  │
│  ┌ floating NEAREST-UNIT card (map click → k-nearest) ┐                  │
│                                                                          │
│           ⌂   ✋   📏   ▦ layers   ◐ basemap   ▤ table   ⏱ status        │
│           └───────── floating icon dock (bottom-center) ─────────┘       │
└──────────────────────────────────────────────────────────────────────────┘
```
- **Template** `launchpad` — **Launchpad** from `../APP-TEMPLATE-LIBRARY.md`: scaffold the `AppLayout`
  from the library skeleton and signature wiring; the bullets here only slot this recipe's content into
  the template's regions. Do **not** fall back to `split-dashboard`.
- **Wiring:** `timer` → `refresh` (live positions); `featureSelect` → follow card + `panTo` follow mode; `mapClick` → nearest-unit query card; dock (`controller`) opens every tool as a draggable floating card; stale-beacon check → `message` banner. Zero docked panels — the anti-dashboard.
- **Layer panel required** (`<LayerPanel store={store}>`, docked) so an operator toggles units/crews/incidents/
  facilities/mutual-aid on/off (visibility/opacity/order/zoom-to).
- **Nearest-unit action:** click an incident (or a map point) → the panel ranks the *N* closest **available**
  units by distance (or drive-time if routing wired) and flashes the winner.

### 2.2 Theme (dark, dispatch)
- **Base:** dark control-room (`#0e1116` bg, `#1a1f27` panels, `#e8eef5` text), **tabular numerals** for all
  metrics; big KPI numbers (32–44px). **Status palette (semantic tokens):** `@status.available` `#39d98a`,
  `@status.enroute` `#f5a623`, `@status.onscene` `#4aa3ff`, `@status.outofservice` `#8a8f98`,
  `@status.stale` `#ff5252` (no recent fix). Incidents in `@hazard.active` `#e34a33`.
- Vehicles as **directional markers** (rotate by `course`/`cog` where the feed carries heading); a **fix-age**
  ramp fades a unit as its last report gets older. Polygon fills (mutual-aid rings) low alpha (~40/255).
- Basemap dark-gray so status colors pop; EN + **AR/RTL** optional (Cairo).

### 2.3 KPI cards (the differentiator)
**Units online** (reporting within the freshness window) · **Available** · **Enroute** · **On-scene** ·
**Out-of-service** · **Avg fix age (min)** · **Low-battery / no-fix count** (exception). Each a big-number card
with label+unit, a **delta vs. last refresh**, and a **status color** by threshold. **Exception banner:**
"N units stale >X min or out-of-service" in red → click filters map+table.

### 2.4 Charts & table
- **Units by status** (bar), **availability gauge** (% available — `RadialGauge`), **fix-age histogram**
  (how fresh is the fleet), **units by agency/type** (bar). A `TimeSeries` (breadcrumb count / online-over-time)
  where a tracks/history layer exists.
- **Unit table:** Unit · Type/Agency · Status · Speed · Heading · Fix age · Battery — sortable, per-column
  filter, **row → zoom + flash**, CSV/GeoJSON export; stale/out-of-service rows tinted.

### 2.5 Capabilities to use (Phases 0–5 — the ops-picture + nearest pattern)
- **Live refresh (no remount).** Keep only the light live AVL layer(s) initial; refresh on an interval; a
  rate-limited/`{error}` response **keeps the last good data** (map/KPIs don't blank). Status filtering is a
  **`definitionExpression`** via `store.setDefinition` (in place), never a remount — matches the strata filter rule.
- **Nearest-unit** — `@strata/processing` **`nearest`** / `withinDistance` (distance rank) for closest available
  unit(s) to a clicked incident; optional **`@strata/plugin-routing`** `fetchIsochrone`/directions for a
  **drive-time** rank (the honest Closest-Facility analogue) when a routing service is available.
- **WIF `connections`** (`@strata/actions` bus) — status chips / agency bar / exception banner → in-place
  `filter` of map + table + KPIs; incident `featureSelect` → the **nearest-unit** action + `zoomTo`; unit
  `rowSelect` → `zoomTo` + `flash` + a docked **`FeatureInfoPanel`** unit card. Author declaratively on
  `AppLayout.connections` (skill `strata-interactivity`).
- **Real charts + deep table** — `ChartPanel` (ECharts/SVG), `RadialGauge`, `CartoPanel` cross-filter widgets;
  `AttributeTablePanel` with server paging + CSV/GeoJSON. A **`DataActionMenu`** (zoom/flash/view-in-table/export)
  on a selected unit.
- **Temporal** — `TimeSlider` over a tracks/breadcrumb layer's `location_timestamp` to scrub crew history.
- **Symbology/popup** genuine ESRI `drawingInfo`/`popupInfo`; status `uniqueValue` renderer + heading rotation;
  popup shows unit id, status, speed, last-report time, battery.
- **Composed export** — a **shift/situation-report PDF** (legend + scalebar + north-arrow + unit roster table),
  `/export image`, and a **share** deep-link for the next watch.
- **"Wall" display variant** — not a shipped one-click toggle: hide the header/chrome and enlarge the KPI
  cards by composing existing `AppLayout` primitives (a `mode:fixed` section + a state-driven class swap on
  a header `button`). See the §7 Harvest note to make this a first-class preset.

## 3. Data sources (Maryland + California)

**The customer's live AVL feed (the core layer):** a published **live FeatureServer** of unit positions —
`{unit_id, status, type, agency, speed, heading, battery, last_report}` — from Velocity/GeoEvent, Field Maps
Location Tracking, or a vendor AVL (MARVLIS/CompassCom). Public repos ship a **synthetic** moving-unit sample;
real feeds stay private. Below are **real live/near-live analogues + nearest-facility targets** (open):

| Role | California | Maryland | National |
|---|---|---|---|
| **Live AVL positions** (the core) | Cal OES **Fire & Rescue Fleet** `services.arcgis.com/BLN4oKB0N1YSgvY8/…/Fire_and_Rescue_Fleet_Layer_08152024/FeatureServer/4` (`ATT_Fleet_Complete_AVL` — frozen Aug-2024 snapshot, seed only); CAL FIRE **Aircraft Tracking** `services1.arcgis.com/jUJYIo9tSA7EHvfZ/…/CAL_FIRE_Aircraft_Tracking_public_view/FeatureServer/0` (real AVL fields: `unitId`,`spd`,`cog`,`batchDateTime`,`altitude`,`tailNumber`,`organizationGroup`,`mission` — live but sparse/stale) | *(no open MD AVL — use synthetic moving-unit sample seeded on MD roads)* | Field Maps Location-Tracking schema (customer-published) |
| **Live air-ops activity** (populated overlay) | CAL FIRE **Helicopter Water Drops** `services1.arcgis.com/jUJYIo9tSA7EHvfZ/…/Helicopter_Water_Drops_(CAL_FIRE_AIRCRAFT_ONLY)/FeatureServer/0` (`unitId`,`dropStartTime`,`incidentName`,`dropDurationSeconds` — **LIVE + dense: 38,480 pts, newest today**; drop events) | — | — |
| **Incidents** (nearest-unit target) | WFIGS current incidents (statewide) | NWS active alerts `api.weather.gov/alerts/active?area=MD` | **WFIGS** `services3.arcgis.com/T4QMspbfLg3qTGWY/…/WFIGS_Incident_Locations_Current/FeatureServer/0`; FEMA OpenFEMA declarations |
| **Stations / facilities** (nearest-facility candidates) | Cal OES Fire/Rescue contacts `_CalOES_Fire_and_Rescue_Contact_Information_Public_View/FeatureServer` | iMap `PublicSafety/MD_Fire/FeatureServer/0` (fire stations) · `MD_EMS` · `MD_Police` · `MD_PublicSafetyAnsweringPoints` (PSAPs) · `Health/MD_Hospitals/FeatureServer/0` | HIFLD fire/EMS/police stations |
| **Mutual-aid regions** (jurisdiction) | Cal OES `CalOES_Fire_Mutual_Aid_Regions_view` · `CalOES_Law_Mutual_Aid_Regions` · `CalOES_Admin_Mutual_Aid_Regional_Boundaries` | *(county boundaries — iMap Boundaries)* | — |
| **Routing context** (drive-time, closures) | Caltrans `CHhighway/CCTV/FeatureServer` (cameras) · QuickMap lane closures/chain-control | iMap `Transportation/MD_TrafficCameras/FeatureServer` · `MD_RoadwayAdministrativeClassification` · MDOT SHA live traffic | OSRM/Valhalla for `@strata/plugin-routing` |

All **EPSG:4326**; iMap + Cal OES + services*.arcgis.com are CORS-enabled. Carry Cal OES / CAL FIRE / MD iMap /
NENA attribution + "no warranty". Reproject any customer feed on the way in.

## 4. Verify each URL first (terminal)
```bash
curl -s "https://services1.arcgis.com/jUJYIo9tSA7EHvfZ/arcgis/rest/services/CAL_FIRE_Aircraft_Tracking_public_view/FeatureServer/0?f=json" \
  | python3 -c "import sys,json;d=json.load(sys.stdin);print('geom:',d['geometryType'],'| fields:',[f['name'] for f in d['fields']][:14])"
```
Note the real field names (they drive symbology/popups/KPIs/rotation): CAL FIRE aircraft `unitId`, `category`,
`batchDateTime` (last-report time — drives **fix age**), `spd` (speed), `cog` (course-over-ground — drives
**marker rotation**), `tailNumber`, `organizationGroup` (agency), `mission`, `altitude`. The Cal OES fleet
service exposes its AVL rows on **sub-layer `/4`** (`ATT_Fleet_Complete_AVL`), not `/0` — confirm the layer id
before `/add-data`. Keep `tile_fields` minimal (position + status) and drive status filtering via
`definitionExpression`, not a remount.

### Verification results (probed 2026-07-21, re-verified 2026-07-22) — the honesty this recipe is built on
Every operational endpoint was hit live (`build_avl_catalog.py` bakes the results into `avl-catalog-ca.json`
`external[]`). Four findings shape the app's data strategy:
- **Cal OES Fleet AVL `/4`** — real AVL **schema** (380 points; `DeviceID`, `Status`, `Make`/`Model`,
  `Position_Speed`, `Position_Direction` = heading, `Position_TimeStamp`, `LastUpdatedTimeStamp`,
  `AssetType_Description`, `Branch`, `HasMDT`, `IsCrashDetected`, `IsSatellite`) **but a frozen snapshot**:
  the re-verify (2026-07-22) found the newest `Position_TimeStamp` is **2024-08-16** — a one-time Aug-2024
  dump (not a 2019–2023 archive as first noted), most fields null, `Status` a numeric vendor code
  (`0`/`2`/`3`), some rows `(0,0)` geometry (filter `Position_Longitude<>0`). It never moves — so it is the
  **seed for the synthetic moving fleet** (real DeviceIDs/plates/asset types/home bases), not a live feed.
- **CAL FIRE Helicopter Water Drops `/0`** — **genuinely LIVE and dense**: 38,480 points, newest timestamped
  **today** (`unitId`, `dropStartTime`, `incidentName`, `dropDurationSeconds`). These are **drop events**, not
  continuous vehicle tracks — but they are the one CAL FIRE/Cal OES layer that is live *and* populated right
  now, so the app wires the most-recent ~1000 (by `dropStartTime`) as a **live air-ops activity overlay** in
  the Layers panel beside the synthetic fleet.
- **CAL FIRE Aircraft Tracking `/0`** — TracPlus positions with real `cog`/`spd`/`batchDateTime`, **live
  during air ops** but the re-verify found only **3 rows, ~2.5 months stale** (newest `batchDateTime`
  2026-05-06): a labeled toggle, never relied on to be populated on demand.
- **No open CA feed is BOTH live-moving AND a dense vehicle *fleet*.** So the app's populated **default is a
  synthetic moving fleet** (clean statuses, heading, ticking `last_report`), with the two real feeds as
  labeled toggles. Live-and-populated **incident** analogues that *do* exist and are wired: **NWS active
  alerts** (`api.weather.gov/alerts/active?area=CA`, ~24 live, CORS-open GeoJSON — send a User-Agent) and
  **WFIGS current incidents** (national; filter `POOState='US-CA'` — 82 live CA points on 2026-07-22).
  Facilities verified live: Cal OES
  **Hospitals**, **CalOESShelters3** (capacity/occupancy), **Fire & Rescue Contacts** (polygon), **Fire
  Mutual-Aid Regions** (polygon).

## Guided wizard — **the prompts that assign the app's defaults**

Claude runs this like an ArcGIS Instant-App wizard: ask each group, **apply the default so "accept all" builds a
complete app**, confirm a one-line summary, then run §5. In Cowork use the multiple-choice tool; in Claude Code, a
short interview (or `/recipe emergency-management_avl-tracker`). Every answer *sets an application default* baked into
`layers.json` / the `AppLayout`.

| # | Wizard question | Options → **default** | The default it assigns |
|---|---|---|---|
| 1 · App | Title & subtitle? | free text → **"Resource & Asset Tracker"** | header title + `strata:notes` subtitle |
| 2 · Region | Which region? | **California** · Maryland | initial extent + which facility/mutual-aid services load (CA: Cal OES fleet/contacts+MARs; MD: iMap stations+hospitals) |
| 3 · AVL feed | Live unit source? | **sample synthetic (moving units)** · CA Cal OES Fleet / CAL FIRE Aircraft · my live FeatureServer URL | the core AVL operational layer + refresh interval + status `uniqueValue` renderer |
| 4 · Refresh | Refresh interval + freshness window? | 15s / 30s / 60s → **30 s**, stale after **5 min** | layer `refresh` + the "units online" / stale-exception threshold |
| 5 · Status field | Status + timestamp fields? | detect → **`status`, `last_report`** (CA aircraft: `mission`/`category`, `batchDateTime`) | the status renderer field, fix-age KPI, `definitionExpression` filter |
| 6 · Nearest | Nearest-unit rank method? | **distance (`nearest`)** · drive-time (routing) | the nearest-unit action (distance by default; wires `@strata/plugin-routing` if drive-time) + facility/incident target layer |
| 7 · KPIs | Headline KPIs? `[multi]` | online · available · enroute · on-scene · out-of-service · avg-fix-age · low-battery → **online + available + enroute + avg-fix-age** | the KPI row (`statistics`/`kpi` + availability `gauge`) |
| 8 · Wall / theme | Wall-display variant + theme/language? | **dark / EN**, wall-display **off** · light · EN+AR (RTL) | `@strata/theme` dark preset + `lang-switch`; wall-display **composed** from `AppLayout` (bigger KPI cards, chrome hidden) — not a built-in toggle |
| 9 · Export | Situation-report PDF + share? | **shift-report PDF + share** · none | wires `/export report` (unit roster) + `/export image` + share deep-link |

**Then:** Claude echoes *"CA · Cal OES fleet AVL · 30 s / stale 5 min · status+last_report · nearest by distance ·
KPIs {online, available, enroute, fix-age} · dark/EN · shift-report PDF"* and, on confirmation, runs §5 — so the
app opens **fully configured**.

## 5. Prompt-script (run in order)
```
A. /new-app — a "Resource & Asset Tracker" `launchpad` template (map ~62% / panels ~38%), dark dispatch theme with
   the status palette (§2.2), plus a composed wall-display variant (bigger KPI cards, header chrome hideable) —
   see §7 harvest note, this is authored via AppLayout, not a built-in toggle. Header with title + a live
   "last refreshed" clock. Bilingual EN/AR optional. Install deps + give me the run command.

B. /add-data the live AVL layer (sample synthetic moving-units, or Cal OES Fire_and_Rescue_Fleet .../FeatureServer/4,
   or CAL FIRE Aircraft .../FeatureServer/0, or my FeatureServer URL) with refresh=30s. Set the initial extent to
   the region. Keep the live layer light in the initial layers.json; add facility/mutual-aid/incident context
   layers to the store AFTER onReady, staggered + default-off. On a rate-limited/{error} refresh, keep the last
   good data (don't blank the map/KPIs). Also add the incident layer (WFIGS / NWS alerts) and station facilities
   (MD_Fire/EMS/Police + MD_Hospitals, or Cal OES contacts) as nearest-unit targets.

C. /symbology units: uniqueValue by {status} with the status palette (available/enroute/on-scene/out-of-service);
   rotate markers by {cog}/{heading} where present; fade by fix age. esriSMSCircle/Triangle only (no esriSMSPath).
   /symbology incidents: @hazard.active. /symbology mutual-aid rings: low-alpha fill. /symbology stations: muted
   neutral facility icons. /popup units: title {unit_id}; show status, speed, heading, last_report (fix age),
   battery, agency. Identify uses these on click (active layer).

D. Nearest-unit (the number): using @strata/processing, /analyze nearest — for a clicked incident, rank the N
   closest AVAILABLE units by distance and flash the winner; list distance to each. Optionally /analyze
   withinDistance for "units within R km of the incident". If the user chose drive-time, wire @strata/plugin-routing
   (isochrone/directions, OSRM/Valhalla) for a travel-time rank — else distance is the honest default.

E. /panel statistics as KPI cards: Units online, Available, Enroute, On-scene, Out-of-service, Avg fix age (min).
   Add an availability RadialGauge (% available). Exception banner: units stale > freshness window OR
   out-of-service OR low-battery — click filters map + table.

F. /panel chart: units by status (bar), availability gauge, fix-age histogram, units by agency/type (bar). Add a
   TimeSeries (online-over-time) if a tracks/history layer exists. /panel table units (Unit, Type/Agency, Status,
   Speed, Heading, Fix age, Battery) — sortable, per-column filter, row→zoom+flash, CSV/GeoJSON; stale rows tinted.

G. WIF: author AppLayout.connections — status chips + agency bar + exception banner → in-place filter of map +
   table + KPIs (setDefinition, no remount); incident featureSelect → the nearest-unit action + zoomTo; unit
   rowSelect → zoomTo + flash + a docked FeatureInfoPanel unit card; a DataActionMenu (zoom/flash/view-in-table/
   export) on a selected unit. A "clear" resets.

H. Controls + export: navigation/scale/geolocate, a Legend + BasemapPanel on canvas, Measure (returns to identify).
   Add a TimeSlider over the tracks layer's location_timestamp (crew history). Ensure the last-refreshed clock ticks
   on each refresh. Compose a wall-display header button (hides chrome, enlarges KPI text via AppLayout — not a
   shipped toggle). Wire /export report (a shift/situation-report PDF: legend/scalebar/north-arrow
   + unit roster table), /export image, and a share deep-link. Dock a LayerPanel (one-line rows) — REQUIRED.
```

## 6. Verify (benchmark to Esri / MARVLIS / CompassCom)
| Check | Pass |
|---|---|
| Live AVL units render by status; refresh ~30 s; "last refreshed" clock ticks | ✅ |
| A rate-limited/error refresh keeps the last good data (map/KPIs don't blank) | ✅ |
| Markers rotate by heading where the feed carries it; fix-age fade works | ✅ |
| KPIs: online, available, enroute, on-scene, out-of-service, avg fix age (gauge) — above the fold | ✅ |
| **Nearest available unit** to a clicked incident ranked by distance + flashed; matches a manual check | ✅ |
| Status chips / agency bar / exception banner cross-filter map + table + KPIs **in place** (no remount) | ✅ |
| Exception banner surfaces stale/no-fix/out-of-service/low-battery units | ✅ |
| Unit table row → zoom + flash; CSV/GeoJSON export; stale rows tinted | ✅ |
| **Shift/situation-report PDF** exports with legend/scalebar + unit roster; share deep-link works | ✅ |
| Layer panel toggles every layer on/off; TimeSlider scrubs crew history | ✅ |
| Runs on Strata **and** ArcGIS; on-prem, no position/PII egress (sovereignty holds) | ✅ (judge) |
| Looks control-room grade vs a Velocity/Dashboards AVL board | ✅ (judge) |

**On-par-or-better:** matches the Velocity/GeoEvent + Dashboards AVL pattern (Manassas Park / Lafayette Parish)
and MARVLIS/CompassCom nearest-unit + status board, and **exceeds on** the AI-authored build, the sovereign/on-prem
posture, cross-widget interactivity, and one-click situation reporting — MIT, on Strata or ArcGIS.

## 7. Harvest (gaps → strata-core)
Log anything missing: a first-class **directional/rotated marker** renderer (rotate by a heading field) and a
**fix-age fade** helper; a **live-refresh "keep last good"** contract + a **"units online / stale"** freshness
KPI; a **nearest-unit widget** (pick incident → ranked list, distance *and* drive-time) as a shipped panel;
a first-class **"wall/kiosk display" `AppLayout` preset** (one-click chrome-hide + KPI-enlarge for a dispatch
screen) — today this recipe composes it from existing primitives (`mode:fixed` + a state-driven class swap),
there is no shipped toggle;
**true drive-time (Closest Facility)** needs a routing service wired end-to-end; a **status/timestamp field
auto-detect**; and **raw-telemetry ingest** (NMEA/streaming) is out of scope — strata consumes a *published* live
layer, not a socket. **EDXL-RM resource request/commit** status (mutual-aid) and **EIDO** CAD unit-status ingest
are future connectors. Editing (unit-status writes) is the auth/editing wave — read-only today.

## 8. Sources
- Esri: [Emergency Management Operations solution](https://www.esri.com/en-us/c/industry/public-safety/emergency-management-operations-solution) ·
  [ArcGIS Velocity](https://www.esri.com/en-us/arcgis/products/arcgis-velocity/overview) ·
  [Field Maps Location Tracking (Tracks)](https://doc.arcgis.com/en/tracker/help/use-tracks.htm) ·
  [Closest Facility service](https://developers.arcgis.com/rest/routing/closestFacility-service-direct/) ·
  [Manassas Park, VA AVL+CAD (Velocity)](https://www.esri.com/en-us/lg/product/stories/law-enforcement-manassas-park-virginia-tracks-vehicles-calls-service-real-time-data-solution) ·
  [Lafayette Parish 911 (GeoEvent)](https://www.esri.com/en-us/lg/industry/public-safety/stories/arcgis-powers-911-situational-awareness-in-lafayette-parish)
- Vendors / benchmark: [BCS MARVLIS](https://www.bcs-gis.com/marvlis.html) · [CompassCom CompassTrac](https://compasscom.com/products/compasstrac-enterprise/) · Blue Raster · Motorola CommandCentral · Hexagon I/CAD.
- Standards / orgs: [NENA-STA-006 (NG9-1-1 GIS)](https://cdn.ymaws.com/www.nena.org/resource/resmgr/standards/nena-sta-006.2-2022_ng9-1-1_.pdf) ·
  [OASIS EDXL-RM](https://www.oasis-open.org/standard/edxlrm/) · [APCO Project 25](https://www.apcointl.org/technology/interoperability/project-25/) ·
  [FirstNet location services](https://firstnet.gov/newsroom/blog/firstnet-enabled-location-services-enhancing-situational-awareness-and-improving) ·
  [EIDO / Exchange (NG911 interop)](https://rapiddeploy.com/blog/exchange-link-the-future-of-ng911-interoperability)
- Data: Cal OES Fire & Rescue Fleet + Mutual-Aid Regions · [CAL FIRE Aircraft Tracking](https://services1.arcgis.com/jUJYIo9tSA7EHvfZ/arcgis/rest/services/CAL_FIRE_Aircraft_Tracking_public_view/FeatureServer) · MD iMap PublicSafety (Fire/EMS/Police/PSAP) + Health/MD_Hospitals · [WFIGS current incidents](https://services3.arcgis.com/T4QMspbfLg3qTGWY/arcgis/rest/services/WFIGS_Incident_Locations_Current/FeatureServer/0) · [NWS active alerts](https://api.weather.gov/alerts/active).
- Internal: `Emergency-Response-Watch-Center.md`, `2-Strata-Core-Plan-and-Spec.md`, `data_sources_{ca,md,national}.md`, `docs/reference/{components,human-language}.md`, sibling recipe `emergency-management_common-operating-picture/RECIPE.md`.

---

## Modernization (parity release)

> Applies the Experience-Builder-parity features (Phases 1–7). See `strata/recipes/MODERNIZATION.md` + `COMPONENT-MANIFEST.md` §8. Cross-cutting for every solution app: a structured **`theme`** (brand/hazard palette → hover/active/focus states + motion), app-shell (`header`/`footer`/`splash`), and `dataSource.{ sourceId }` linking (widgets sharing a source link with **no `connections`**).

- **Live positions:** **`StreamDataSource`** (poll/push) for unit/vehicle locations + **`timer → refresh`**; layer-level `refreshIntervalSeconds` stays as the map-side option.
- **Nearest-unit chain:** **`analysis`** (`withinDistance`/`nearest`) + `sketchComplete → selectByGeometry` from a clicked incident.
- **Layout/theme:** dockable **`panel`** roster linked to the map by `sourceId`; a **`window`** unit card; hazard **`theme`**.
