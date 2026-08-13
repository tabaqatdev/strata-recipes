# Recipe — The Watch Wall (Live Operations Dashboard, reinvented)

A reproducible path to an **unattended EOC watch wall** on strata-core, for California: a **self-rotating
common operating picture** (`views` + `autoPlay`) that cycles Overview → By-Area → Weather/Hydro →
Resources & Lifelines → Grid & Power on its own for a shift, where **every tile shows how fresh its feed
is** (a green→amber→red→gray-dead dot with a cadence label), and where a **high-severity `countChange`
breaks the rotation and snaps the wall to an ESCALATION pane** — the incident zoomed, its card, the
affected lifelines, a red banner — then resumes on the next tick. Designed **open-design** ("watch-wall")
from 2026 research **beyond ESRI** (§1), with **two independent agents that internet-verified the live CA
+ national feeds before any synthesis**. Full rationale: `DESIGN-PROPOSAL.md`; the v1 extent-scoreboard is
preserved as `RECIPE-v1-scoreboard.md`.

> **Scope (honest).** A **near-real-time analytics/serving layer over feeds an agency already publishes** —
> not a CAD/RMS system of record, not a dispatch console, not a mass-notification platform. "Live" is a
> **polling refresh** (default 180s, per-feed cadence shown), **not** a push firehose; there is **no
> per-feature editing** this release (writes need the writable+authenticated ESRI backend 🔶 — read-only
> degradation is the default). It **coexists with** Motorola/Hexagon/Juvare/Genasys command systems as the
> unattended map+metric wall. Feeds and layers are cataloged in **`lodb-catalog-ca.json`**: the
> 2,339-service CA library with **713 tagged to 11 operational roles**, plus **23 external live feeds — 19
> verified-live** (2026-07-20; incl. CA county boundaries + FHSZ hazard zones), 2 proxy-needed (CAISO CSVs,
> FEMA NFHL), 1 key-required (NASA FIRMS, labeled), 1 unverified-egress-blocked (CHP CAD — confirm from a
> browser). A rate-limited or failed
> refresh **keeps last-good data**; only the freshness dot degrades — never a blank wall mid-shift.

Run the §5 prompt-script on a fresh strata-core project (or `/new-app` / `/recipe emergency-management_live-dashboard`).
ESRI Web Map JSON is the map contract throughout; OSM/keyless basemaps by default; everything EPSG:4326.

---

## 1. Study — the market beyond ESRI (research 2026-07-20)

**The question a duty officer asks off the wall:** *what's open, what's escalating, are the lifelines
holding — and is this data actually still live, or did a feed die while I looked away?*

**How the market frames it (benchmark + coexist, never copy).** Everyone sells *situational awareness /
common operating picture (COP)* — but as a **per-seat, cloud, ecosystem-locked platform**:
- **Esri** — the *Emergency Management Operations* **Incident Status Dashboard** (ArcGIS Dashboards,
  Arcade KPIs, ~3-min refresh) + the *Situational Awareness* solution (the live layer) is the reference
  wall use-case we model and exceed.
- **Motorola CommandCentral Aware** / **Hexagon HxGN OnCall + Luciad** — RTCC/COP stacks that fuse
  CAD+video into an *operator-driven* screen; Luciad is the fast live-track renderer competitors respect.
- **Juvare WebEOC** / **Veoci** / **D4H** — EOC systems-of-record (status boards, forms, ICS); WebEOC is
  the dominant, widely-resented incumbent.
- **Everbridge** (critical-event management / mass notification), **Dataminr / Samdesk** (earliest
  detection), **Disaster Tech Charlie** (impact prediction) — alert/detection layers, not walls.
- **Genasys / Zonehaven** (now the de-facto CA evacuation-zone standard), **Perimeter**, **Watch Duty**,
  **PulsePoint** — wildfire/evac specialists we *read from*.
- **Palantir Gotham** — the "everything fuses here" incumbent to beat on cost/openness.
- **Grafana kiosk + playlist** — the **only** product built to run a wall unattended on a timer, and it's
  **GIS-weak** (one Geomap panel). This is the open analog the Watch Wall out-designs on geospatial depth.

**The four empty intersections (the design brief).** After-action reports and reviews converge on one
complaint — *"too many tabs, I can't tell what's stale, it costs a fortune per seat, it's in someone
else's cloud, and it never tells me when something changed."* Hence: **(a) a true unattended wall mode**,
**(b) per-feed freshness made visible**, **(c) an anomaly-interrupt that breaks its own rotation**, **(d)
a sovereign/on-prem posture**. The Watch Wall occupies all four; no incumbent holds them together.

**Our edge:** AI-authored, MIT/open, runs on **Strata *or* ArcGIS**, control-room-grade KPI/chart quality,
cross-widget interactivity, **the freshness + anomaly-interrupt + unattended-rotation trio**, and
**sovereign/on-prem** so incident data never touches a cloud LLM.

**Standards & orgs to speak fluently:** **NIMS/ICS** (the EOC supports ICS, it doesn't command it);
**FEMA Community Lifelines — now 8** (Water Systems split out 2023: Safety & Security · Food, Hydration,
Shelter · Health & Medical · Energy · Communications · Transportation · Hazardous Materials · Water
Systems; green/yellow/red/gray); **CAP v1.2** (OASIS/IPAWS alert format — we consume it from
`api.weather.gov`) + the **EDXL** suite (DE/HAVE/RM/SitRep); **CISA 16 sectors** (PPD-21 / **NSM-22
2024**); **NFPA 1660** (2024 — consolidates the old 1600); **COP** doctrine; **ESRI Web Map JSON / ArcGIS
REST**. Orgs: FEMA · CISA · IAEM · OASIS · NWS/NOAA · USGS · NIFC/WFIGS · Cal OES · Caltrans · CAISO.

## 2. UI design spec (front-loaded)

Control-room grade, legible at 15 ft, stable unattended for a shift. This is the visual contract; §5 builds it.

### 2.1 Layout (Template: `open-design` — "watch-wall")
- Charter/manifest §10; three §10.3 candidates (`freshness-dot`, `anomaly-interrupt`, `feed-ticker`) each
  with a day-1 fallback (§2.6). **Anti-collision:** v1 `scoreboard` retired (no extent-recompute
  protagonist / human-present assumption); distinct from `ops-command` (cop — an *operator-driven* command
  board vs this **unattended, time-rotated, freshness-gated, anomaly-interrupting** wall) and from
  `time-player` (flood — a human scrubs; here time drives rotation autonomously) and every other emergency
  sibling. **Harvest candidate: `watch-wall`.**

```
┌ HEADER: ● LIVE · California ▾ · ⟳ AUTO-ROTATE ▶ pane 3/5 · shift 14:03 · [Manual] [Brief] ─────────┐
│  ┌───────────────────── ROTATING STAGE  (views · nav:slides · autoPlay 20s · loop · fade) ─────────┐ │
│  │ 1 OVERVIEW   full-bleed map (incidents·evac·outages) + KPI strip + FEMA lifeline stoplight row   │ │
│  │ 2 BY AREA    county bars  ⇄  map  ⇄  incident table (row → zoom + flash)                         │ │
│  │ 3 WX / HYDRO NWS alert TICKER · river-gauge hydrograph (TimeSeries) · quakes · AQI/smoke         │ │
│  │ 4 RESOURCES  8 FEMA lifeline gauges · resources committed · shelters open (+capacity/ADA)        │ │
│  │ 5 GRID/POWER Cal OES outage map · CAISO fuel-mix · impacted-customers KPI                        │ │
│  │ ⚠ ESCALATION (interrupt-only) the escalating incident zoomed + card + affected lifelines         │ │
│  └────────────────────────────────────────────────────────────────────────────────────────────────┘ │
├────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ FEED BAR (footer): FIRIS ⬤2m · WFIGS ⬤4m · NWS ⬤live · quakes ⬤1m · outages ⬤11m · gauges ⬤32m ·   │
│   AQI ⬤48m · CCTV ⬤live · CAISO ⬤5m       ← per-feed freshness (green→amber→red→gray-dead + cadence) │
└────────────────────────────────────────────────────────────────────────────────────────────────────┘
```
- Pages: `wall` (fixed; header + `views` root + footer feed bar) + `brief` (print/export view — the analyst
  page where `draw`/`analysis` live, off the wall). A **`LayerPanel` is REQUIRED** (docked, one-line rows +
  ⋯) so a watch officer can curate. Phone: the stage is a swipeable single pane; footer → one freshness
  chip; auto-rotate defaults **off**.

### 2.2 Theme (dark watch-center; **light/dark switch**)
Structured `Theme`, `mode:"dark"`, `@strata/theme` **`hazard`** base: `colors.primary` slate `#3b82c4`,
`danger` `#e03131` (escalation banner + high-sev), `warning` `#f59f00` (stale feed / evac Warning),
`success` `#2b8a3e` (contained / lifeline green); **`fonts.scale:"spacious"` + tabular numerals** so KPIs
read at 15 ft on 4K. Freshness ramp green→amber→red→gray with a cadence label; the escalation view flips
the stage to a `--strata-critical` frame; lifeline stoplight green/amber/red/gray. Keyless **CARTO Dark**
basemap so red incident data pops; **polygon fills low alpha (~40/255)** so evac zones, perimeters, outage
areas and floodplain stack. A **`theme-switch` (light/dark)** flips the UI *and* the basemap
(CARTO Dark ↔ **Positron** light) and persists to `localStorage`; all scrims (KPI strip, pane title) are
themed via `--scrim-*` tokens so nothing stays dark in light mode. EN + AR/RTL (Cairo mirror). *Character:
an airport departures board for disasters — calm and self-running until it isn't.*

### 2.2b Map context, legend & identify (enrichment)
- **County boundaries** (CA GIO `Boundaries/CA_Counties`, `POLYGON_NM`) render as always-on thin lines
  beneath every layer — the geographic frame a duty officer reads by. Toggleable from the legend.
- **On-map legend** (canvas surface, bottom-left on the two map-primary panes): per-pane symbology swatches
  + **layer toggles** (County boundaries, Fire Hazard Zones) + an "◉ click any feature to identify" hint.
- **Click-to-identify**: a single map click runs `queryRenderedFeatures` (a forgiving 7-px hit box) across
  every hazard + context layer → a themed popup (incident, perimeter, evac zone, outage, alert, gauge, AQI,
  shelter, quake, county). If the FHSZ overlay is on, the same click also runs the MapServer `/identify`
  and appends the hazard class. A **hover over the stage softly pauses rotation** (walk-up to inspect).
- **Fire Hazard Severity Zones (FHSZ)** — an optional CAL FIRE FRAP hazard overlay (MapServer raster via
  `/export?bbox={bbox-epsg-3857}`), toggled from the legend; click-identify returns `HAZ_CLASS`
  (Moderate/High/Very High) + responsibility area (SRA/LRA/FRA).

### 2.3 The differentiators
1. **Unattended self-rotation** (`views` + `autoPlay:{intervalMs,loop}`) — the wall cycles the whole COP
   hands-free for a shift; dwell per pane is configurable. (Grafana has this; nobody has it *with GIS depth*.)
2. **Per-feed freshness** — every tile's dot ramps green→amber→red→**gray (dead)** off the source's
   **last-successful** poll, with the cadence printed. Answers "is this quiet or is it broken?"
3. **Anomaly-interrupt** — a high-severity `countChange` (M5+ quake, warning upgrade, lifeline→red,
   high-sev incident count↑) **halts rotation, snaps to the escalation pane + red banner**, resumes next
   tick. Attention management no incumbent ships.
4. **Sovereign/on-prem** — runs local; incident data never touches a cloud LLM; MIT; Strata *or* ArcGIS.
5. **Generated shift-briefing PDF** — the COP as of now (KPIs, lifeline colors, active alerts, feed
   vintages), one click, no re-typing.

### 2.4 Wiring (→ `AppLayout.connections`, implemented in §5)
stage `viewChange` → feed-bar `message` (pane indicator); `timer`(180s) → live sources `refresh`; hi-sev
source `countChange` → stage `navigate{viewId:escalation}` + global `message` banner; area chart
`categorySelect` → map + table `filter`; incident-table `rowSelect` → map `zoomTo`+`flash` + feature-info;
alert ticker `rowSelect` → map `zoomTo`; Manual `buttonClick` → stage `navigate` (pause); [Brief]
`buttonClick` → `export`; overview map `extentChange` → in-view KPI `showStatistics`.

### 2.5 Capabilities (sweep in DESIGN-PROPOSAL §8)
`views`(autoPlay — signature) · `kpi`/`gauge`(live `stat`: KPIs + 8 lifeline gauges + %-contained radial)
· `chart`(by-area bar/by-status donut/trend line) · `carto` · `table`(server-paged, CSV/GeoJSON) ·
`list`(ticker) · `feature-info`(**click-to-identify popup across all hazard/context layers + FHSZ
`/identify`**) · `legend`(**on-canvas, per-pane, County/FHSZ toggles**) · **county-boundary + FHSZ hazard
context layers** · `layer-panel`(REQUIRED) · processing `pointsWithin`(incidents×county) +
`aggregate` + `withinDistance` + `hexbinDensity` · `date-filter`/timeslider(new-in-last-hour + trend) ·
DataSource `StreamDataSource`(poll, every feed) + `RestDataSource`(catalog) + `StatisticsDataSource` +
`sourceId` linking · export/report(shift PDF + county atlas) · app-shell + structured theme +
**`theme-switch`(light/dark + basemap swap, persisted)** + motion + hover-pause +
i18n · feature-arcgis/auth 🔶 (read-only degradation default). **Deliberately unused:** swipe/scroll-story/
media-pager/weighted-overlay (other siblings' jobs); interactive `draw`/`analysis` kept on `brief`, off
the wall; `reporter`/writes (needs ESRI backend 🔶).

### 2.6 §10.3 New-widget blocks (fallbacks ship day 1)
1. **`freshness-dot`** → *fallback* `text` chip + `timer → refresh` + arcade age-tint (CI feed-bar pattern).
2. **`anomaly-interrupt`** → *fallback* `countChange` on a hi-sev filtered source → `navigate{viewId}` +
   `message`; `escalation` is the last view, `autoPlay` resumes next tick (also visits it ~1×/loop showing
   "no active escalation" — honestly noted).
3. **`feed-ticker`** → *fallback* `list`/`table` bound to the NWS-alerts source, sorted `effective` desc,
   arcade severity tint (a real ticker minus the auto-scroll).

## 3. The data catalog (source of truth)
**`lodb-catalog-ca.json`** via **`build_lodb_catalog.py`** (reuses the Geo Atlas CA parser): library
**2,339**; **713 role-tagged** — incidents 471 (inflated by CAL FIRE's per-fire historical perimeter
snapshots — the *live* incident feeds are the external block) · damage 86 · hydro 62 · transportation 36 ·
health 15 · resources 11 · alerts 7 · evacuations 7 · power 7 · shelters 6 · lifelines 5; **external** =
the **23 internet-verified live feeds** with per-feed `status` + `cadence` + `gotcha` (19 verified-live / 2
proxy-needed / 1 key-required / 1 unverified-egress-blocked) — incl. the enrichment layers **CA county
boundaries** (`ext:ca-counties`, CA GIO, `POLYGON_NM`) and **Fire Hazard Severity Zones** (`ext:fhsz`, CAL
FIRE FRAP MapServer — raster `/export` + click `/identify` → `HAZ_CLASS`). Extend the `ROLES` regexes and
`EXTERNAL` list as feeds evolve; rerun after re-crawls. `python3 build_lodb_catalog.py`.

## 4. Verify each URL first (terminal)
```bash
# Cal OES power outages — LAYER /1 = Areas (poly); encode %28View%29; ~15-min:
curl -s "https://services.arcgis.com/BLN4oKB0N1YSgvY8/arcgis/rest/services/Power_Outages_%28View%29/FeatureServer/1?f=json" \
  | python3 -c "import sys,json;d=json.load(sys.stdin);print(d['geometryType'],[f['name'] for f in d['fields']][:8])"
# CAL FIRE FIRIS+WFIGS combo — LAYER /0; wkid 4269 → outSR=4326:
curl -s "https://services1.arcgis.com/jUJYIo9tSA7EHvfZ/arcgis/rest/services/CA_Perimeters_NIFC_FIRIS_public_view/FeatureServer/0?f=json" \
  | python3 -c "import sys,json;d=json.load(sys.stdin);print(d['name'],d['geometryType'],d['extent']['spatialReference'])"
# WFIGS incident locations — filter POOState='US-CA':
curl -s "https://services3.arcgis.com/T4QMspbfLg3qTGWY/arcgis/rest/services/WFIGS_Incident_Locations_Current/FeatureServer/0/query?where=POOState%3D%27US-CA%27&returnCountOnly=true&f=json"
# NWS active alerts (CAP; keyless; set a User-Agent in prod):
curl -s -H "User-Agent: strata-core/1.0 (ops@example.gov)" "https://api.weather.gov/alerts/active?area=CA" \
  | python3 -c "import sys,json;d=json.load(sys.stdin);print('alerts',len(d['features']))"
# USGS quakes (the anomaly-interrupt source; keyless GeoJSON):
curl -s "https://earthquake.usgs.gov/earthquakes/feed/v1.0/summary/all_day.geojson" \
  | python3 -c "import sys,json;d=json.load(sys.stdin);print('quakes',len(d['features']))"
# EPA AirNow (keyless ArcGIS FS — use THIS not airnowapi.org): filter StateName='California'
curl -s "https://services.arcgis.com/cJ9YHowT8TU7DUyn/arcgis/rest/services/Air%20Now%20Current%20Monitor%20Data%20Public/FeatureServer/0/query?where=StateName%3D%27California%27&returnCountOnly=true&f=json"
```
**Gotchas (from the 2026-07-20 verify):** field names differ across sibling feeds — evac status is
`zone_status` (Zonehaven roll-up, richer) vs `STATUS` (CalOESHosted); shelters use `CDSS_Active_Shelter_
Sites_View` (234 rows, `ada_accessible_site`/`overall_capacity`/`census`) over sparse `CalOESShelters3` (7
rows, but pet fields). **wkid varies:** 4269 (FIRIS/WFIGS), 4759 (NWS gauges), 4326 (CCTV), 3857 (Cal OES)
→ reproject all to 4326. **Proxy only:** CAISO CSVs + FEMA NFHL (`STRATA-CORE-ISSUES.md` proxy note).
**MapServer `f=geojson` lowercases field names** — lowercase in client renderer `field` refs. Cal OES
outage URL needs `%28View%29`. CHP CAD `sa.xml` is keyless per docs but was datacenter-IP-blocked from the
verify sandbox — confirm from a browser before wiring; ship as a labeled placeholder until then.

## Guided wizard

| # | Question | Options → **default** | Assigns |
|---|---|---|---|
| 1 | Title & region? | → **"Live Operations Dashboard — California"** | header + initial extent + feed set |
| 2 | Panes to rotate? `[multi]` | overview · by-area · wx/hydro · resources+lifelines · grid/power → **all 5** | the `views` + which sources load |
| 3 | Rotation cadence? | 10 / **20** / 30 s · loop on/off → **20s · loop** | `autoPlay.intervalMs` + pane indicator |
| 4 | Refresh cadence? | 1 / **3** / 5 min | each feed's `refreshIntervalSeconds` + freshness thresholds |
| 5 | Anomaly triggers? `[multi]` | M5+ quake · warning upgrade · lifeline→red · hi-sev incident↑ → **all** | the `countChange`→`navigate` interrupt(s) + banner |
| 6 | Live feeds? `[multi]` | NWS alerts · river gauges · quakes · outages · AQI/smoke · CCTV · CAISO → **alerts + gauges + quakes + outages + AQI** | extra live layers + freshness dots + hydrograph |
| 7 | Lifelines row? | **yes (8 FEMA gauges, 2023 doctrine)** · legacy 7 · no | the lifeline stoplight/gauges + mapping |
| 8 | Wall mode? | **on (chromeless, big type)** · off | the 4K wall variant (composed per §2.1, not a built-in flag) |
| 9 | Theme, language, export? | **watch-dark / EN / shift-briefing PDF** · EN+AR(RTL) · +atlas | `hazard` ThemeSpec + `lang-switch` + `/export` |

**Then:** Claude echoes *"CA · all 5 panes rotating 20s · refresh 3-min · interrupts {M5+, warning-upgrade,
lifeline-red, hi-sev↑} · feeds {alerts, gauges, quakes, outages, AQI} · 8 lifelines · wall on · watch-dark
/ EN + briefing PDF"* and, on confirmation, runs §5 — the app opens fully configured.

## 5. Prompt-script (run in order)
```
A. python3 build_lodb_catalog.py — confirm library 2339 · role-tagged 713 · external 23 (19 verified-live).
   Register the catalog as a RestDataSource and each live feed as a StreamDataSource(poll, per-feed cadence).

B. /new-app — "Live Operations Dashboard — California", open-design per §2.1: pages `wall` + `brief`;
   watch-dark hazard ThemeSpec (§2.2, fonts.scale spacious + tabular numerals); header (● LIVE, region ▾,
   ⟳ auto-rotate toggle + pane indicator, shift clock, Manual, Brief). Root of `wall` = a `views` node
   (nav:slides, autoPlay:{intervalMs:20000, loop:true}, animate:fade) with 5 rotating views + a 6th
   `escalation` view; footer = a row of freshness-dot chips (fallback text). If wall mode, author the
   chromeless big-type variant (§2.1). Install deps + give the run command.

C. /add-data the live feeds (from §3 external, per region): incidents = FIRIS+WFIGS combo /0 (wkid 4269) +
   WFIGS Incident Locations /0 (POOState='US-CA') + CAL FIRE Incidents GeoJSON API; evac = Combined
   Statewide (Zonehaven, zone_status); shelters = CDSS Active Shelter Sites; power = Cal OES Power Outages
   /1 + PSPS; alerts = NWS active alerts (CAP) + NWS WWA /1 + GDACS; hydro = NWS river gauges /0 (wkid
   4759) + USGS NWIS streamflow + USGS quakes; AQI = EPA AirNow FS (StateName='CA' — 2-letter, not
   'California'); traffic = Caltrans CCTV + CWWP2 LCS. Plus **context: CA county boundaries** (CA GIO
   `Boundaries/CA_Counties`, always-on thin lines beneath everything) and an optional **FHSZ hazard overlay**
   (CAL FIRE FRAP `Fire_Severity_Zones` MapServer raster via `/export?bbox={bbox-epsg-3857}`, default-off).
   Set refresh 180s. Route CAISO CSVs + FEMA NFHL through the dev-proxy (non-CORS). LOAD STRATEGY: light
   live layers initial; heavy context (floodplain, historical perimeters) after onReady, staggered +
   default-off; map.resize()+fitBounds in onReady (STRATA-CORE-ISSUES #5/#6/#7). Reproject all to 4326.

D. /symbology (genuine ESRI drawingInfo): incidents unique-value by type/status (esriSMSCircle),
   high-sev @danger; evac zones unique-value by zone_status (Warning→Order ramp), low fill alpha; shelters
   sized by capacity; outage areas by ImpactedCustomers; NWS alert polygons by severity; AQI points by
   PM25_AQI_LABEL ramp.

E. /popup per layer (verified fields): incidents {IncidentName} → type/status/county/PercentContained/
   reported; shelters → site_status/overall_capacity/census/ada_accessible_site; evac → community/
   zone_status/est_population; NWS alerts → event/severity/expires; outages → UtilityCompany/
   ImpactedCustomers/EstimatedRestoreDate; gauges → location/status/observed.

F. /analyze (@strata/processing): pointsWithin incidents × county → the By-Area counts; aggregate
   incidents by type/status → chart/KPI group-bys; withinDistance resources & shelters near active
   incidents → "resources committed" + facilities-at-risk; hexbinDensity for incident density (overview).

G. /panel KPIs (kpi/gauge, live stat, §2.3): Active incidents (+Δ 1h) · New in last hour · Evacuations
   active · Shelters open (+% capacity) · Road/lane closures · Resources committed · Impacted customers ·
   Avg %contained (radial gauge, threshold colors). Lifeline stoplight = 8 FEMA gauges. /panel chart:
   by-AREA bar (map-linked), by-STATUS donut, TREND line; a TimeSeries hydrograph for the river gauge; a
   feed-ticker (fallback list) of NWS alerts. /panel table incidents (Name/Type/County/Status/Reported/
   %Contained) — sortable, per-column filter, row→zoom+flash, CSV/GeoJSON; high-sev rows tinted.

H. WIF (author AppLayout.connections, §2.4): stage viewChange → feed-bar message; timer(180s) → refresh
   (freshness reset); hi-sev source countChange → stage navigate{viewId:escalation} + message banner
   (the anomaly-interrupt, fallback per §2.6.2); area chart categorySelect → map + table filter;
   incident-table rowSelect → zoomTo+flash + feature-info; alert ticker rowSelect → zoomTo; Manual
   buttonClick → navigate (pause); overview map extentChange → in-view KPI showStatistics; **map mapClick →
   click-to-identify popup (queryRenderedFeatures, 7-px box, across all hazard/context layers + FHSZ
   `/identify`); stage hover → soft-pause rotation.** Controls: an on-canvas **Legend** (per-pane swatches +
   County/FHSZ toggles + identify hint) + BasemapPanel + Measure; a **`theme-switch` (light/dark)** that
   swaps the UI + basemap (CARTO Dark↔Positron, persisted); a docked LayerPanel (REQUIRED); bookmarks.
   Wire /export report (shift-briefing PDF: KPIs, lifeline colors, active alerts, feed vintages), a
   per-county atlas, and a share deep-link. Keep last-good data on a failed refresh; degrade only the dot.

I. Verify §6; log gaps to §7.
```

## 6. Verify (benchmark to Esri Incident Status Dashboard + Grafana kiosk)
| Check | Pass |
|---|---|
| Wall auto-rotates all 5 panes hands-free (autoPlay), dwell configurable, loops a full shift unattended | ☐ |
| Every tile shows a per-feed freshness dot (green→amber→red→gray-dead) + cadence, off last-**successful** poll | ☐ |
| Anomaly-interrupt: a hi-sev countChange halts rotation, snaps to escalation + red banner, resumes next tick | ☐ |
| KPIs above the fold: active, new-1h, evac, shelters (+capacity), closures, resources, impacted-customers, %contained gauge | ☐ |
| FEMA lifeline stoplight (8 gauges) reads green/amber/red/gray | ☐ |
| By-Area chart ⇄ map ⇄ table cross-filter in place (no remount); table row → zoom+flash; CSV/GeoJSON | ☐ |
| Live feeds render + refresh: FIRIS/WFIGS incidents, Cal OES outages /1, NWS alerts, quakes, river gauges, AirNow, CCTV | ☐ |
| County boundaries render as always-on context; toggle on/off from the legend | ☐ |
| Light/dark switch flips UI **and** basemap (CARTO Dark↔Positron), all scrims adapt, persists across reload | ☐ |
| On-map legend shows per-pane swatches + County/FHSZ toggles + identify hint | ☐ |
| Click-to-identify: clicking a hazard/context feature opens a themed popup (county, incident, evac, outage, alert, gauge…) | ☐ |
| FHSZ overlay toggles on; click-identify returns HAZ_CLASS (Moderate/High/Very High) + SRA/LRA/FRA | ☐ |
| Hover over the stage pauses rotation; leaving resumes (walk-up to inspect) | ☐ |
| Verified gotchas honored in the running app: wkid 4269/4759/4326→4326, zone_status vs STATUS, %28View%29, AirNow StateName='CA', proxy for CAISO/NFHL | ☐ |
| A rate-limited / failed refresh keeps last-good data (KPIs/map/charts never blank); only the dot degrades | ☐ |
| Shift-briefing PDF from live state (KPIs, lifeline colors, alerts, feed vintages); per-county atlas; share deep-link | ☐ |
| Wall variant (chromeless, big type) runs at 4K legible at 15 ft; phone = swipeable single pane, rotate off | ☐ |
| Runs on Strata **and** ArcGIS; on-prem, no data egress; key-gated/unverified feeds shown with status, not faked | ☐ (judge) |
| Beats the bar: unattended rotation + per-feed freshness + anomaly-interrupt + sovereignty together in NO incumbent | ☐ (judge) |

**On-par-or-better:** matches Esri's Incident Status Dashboard (counts by type/area/status, trend, KPI
cards, ~3-min refresh) and Grafana's kiosk rotation, and **exceeds on** GIS depth + the freshness/
anomaly-interrupt/unattended trio, the FEMA 8-lifeline stoplight, the sovereign/on-prem posture, and the
one-click shift-briefing PDF — MIT, on Strata or ArcGIS.

## 7. Harvest (gaps → strata-core)
Log to `STRATA-CORE-ISSUES.md`: a first-class **`freshness-dot`** widget (last-successful-poll age ramp) and
an **`anomaly-interrupt`** widget (threshold → forced viewChange + autoPlay pause/resume with a real
dwell) — both shipped here as fallbacks, both belong in the core; a **`feed-ticker`** (auto-scroll alert
strip); a **CAP-alert ingest** adapter (`api.weather.gov/alerts` → styled layer); a **CSV-feed
DataSource** (CAISO fuel-mix) via the proxy; a **push/websocket** refresh (today it's polling); a real
**`wall`/kiosk layout preset** (today hand-composed per §2.1); a **wall auto-rotate that pauses on manual
touch and resumes on idle**. **Editing** (log/update incidents, resource-status writes) needs the
writable+authenticated ESRI backend 🔶 — read-only this release; Strata editing planned. The catalog spine
now serves four products (Atlas, COP, Lifelines, Watch Wall) — promote the role-tagging convention into the
shared builder. Per the factory loop, each recipe hardens the core.

## 8. Sources
- **Esri (benchmark):** Emergency Management Operations solution · Incident Status Dashboard · Situational
  Awareness solution · "Dashboards for emergency response" blog.
- **Market beyond ESRI:** Motorola CommandCentral Aware · Hexagon HxGN OnCall / Luciad · Juvare WebEOC ·
  Veoci · D4H · Everbridge CEM · Dataminr First Alert · Samdesk · Disaster Tech Charlie · Genasys/Zonehaven
  · Perimeter · Watch Duty · PulsePoint · Palantir Gotham · IBM Environmental Intelligence · **Grafana
  kiosk/playlist** (the unattended-wall analog).
- **Standards / orgs:** FEMA Community Lifelines (8, 2023 Water Systems) · NIMS/ICS · OASIS CAP v1.2 + EDXL
  · CISA 16 sectors (PPD-21 / NSM-22 2024) · NFPA 1660 (2024) · FEMA IPAWS/COP · IAEM.
- **Verified live feeds (2026-07-20, two agents):** Cal OES Power Outages `Power_Outages_(View)/1` (15-min)
  · CAL FIRE FIRIS+WFIGS combo `/0` (wkid 4269) · WFIGS Incident Locations/Perimeters Current · CAL FIRE
  Incidents GeoJSON API (`incidents.fire.ca.gov`) · Cal OES Combined Statewide Evacuation (Zonehaven) ·
  CDSS Active Shelter Sites · NWS active alerts (CAP) + WWA `/1` · USGS quakes + NWIS streamflow · NWS
  river gauges `/0` (wkid 4759) · EPA AirNow FS (keyless, `StateName='CA'`) · Caltrans CCTV + CWWP2 LCS ·
  CAISO fuel-mix CSV (proxy) · OpenFEMA · GDACS · **CA county boundaries** (CA GIO `Boundaries/CA_Counties`,
  `POLYGON_NM`) · **FHSZ** (CAL FIRE FRAP `Fire_Severity_Zones` MapServer — raster `/export` + `/identify`).
  Flagged: FEMA NFHL (proxy), NASA FIRMS (free key), CHP CAD `sa.xml`
  (egress-blocked — confirm from browser), PowerOutage.us/ShakeAlert (permitted/paid).
- **Data:** `data_sources_ca.md` (2026-07-16) → `lodb-catalog-ca.json` (`build_lodb_catalog.py`).
- **Internal:** `DESIGN-PROPOSAL.md` · `RECIPE-v1-scoreboard.md` · manifest §10 · app-design guide · sibling
  recipes `emergency-management_common-operating-picture`, `emergency-management_critical-infrastructure`.

---

## Modernization (parity release)
> Native to this design: **`views` autoPlay** (the self-rotating wall) · **`StreamDataSource(poll)`** per
> live feed + `timer → refresh` freshness loop · **`countChange → navigate`** anomaly-interrupt ·
> `StatisticsDataSource` live KPIs/gauges + `sourceId` linking (no wiring) · structured **`hazard`**
> ThemeSpec + `fonts.scale:"spacious"` for distance · **`theme-switch` (light/dark + basemap swap,
> persisted)** · **on-canvas `legend` with layer toggles** · **`mapClick`→click-to-identify** across all
> hazard/context layers (+ MapServer `/identify` for FHSZ) · **county-boundary + FHSZ hazard context
> layers** · hover-to-pause rotation · app-shell (header/footer/splash) · export-from-state shift-briefing
> PDF · last-good-data-on-failure as a first-class behavior.
