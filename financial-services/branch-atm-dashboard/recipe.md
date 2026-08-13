# Recipe — Branch & ATM Network Operations Center (Banking, network-cluster lead)

A reproducible path to a **branch & ATM network operations suite** on strata-core: one app, three pages,
one map spine — **PULSE** (is the estate healthy right now), **DISPATCH** (an ATM died: remote-fix or
send an engineer, and will we beat the SLA clock), **CASH & PLAN** (who runs out of cash when, is today's
CIT run right, where is the network over/under-built). Designed **open-design** from a 2026 competitive
study of the ATM-ops market (§1): the incumbent consoles are strong telemetry with dumb-pin maps — this
app's thesis is that **monitor, dispatch, cash, and planning belong on one map.** Full design rationale:
`DESIGN-PROPOSAL.md`; the v1 single-page dashboard is preserved as `RECIPE-v1-ops-command.md`.

> **Scope (honest).** Performance/fault/cash telemetry (transactions, uptime, module faults, cash levels)
> lives in the bank's own switch/fleet systems (ISO 8583; NCR/DN/Auriga tooling) — **not** public data.
> This app is the **serving + operations layer over that synced feed**; public data supplies locations
> (FDIC BankFind) and context (census tracts, ACS, LMI/CRA). Engineer positions and CIT stops are a small
> operational feed (synthetic in the template). Deploy **on-prem behind the bank's perimeter/SSO**.
> Routing is **keyless public-OSRM** in the template (demo-grade; self-host for production); **isochrones
> need a self-hosted Valhalla/ORS endpoint** — without one the impact ring falls back to a labeled
> straight-line buffer. Dispatch **write-back** (status→dispatched) needs a writable ESRI backend; the app
> degrades to read-only elsewhere.

Run the §5 prompt-script on a fresh strata-core project (or via `/new-app`). ESRI Web Map JSON is the map
contract; the telemetry feed ingests via the Data Store upload flow.

---

## 1. Study — how the market frames this (research 2026-07-19)

**The three questions the personas ask:** ops manager — *is the estate serving customers right now?*;
dispatcher — *who fixes this failure, how fast, ETA vs SLA?*; cash/network planner — *who runs out when,
is the CIT run right, where is coverage wrong?*

**Reference solutions (benchmark + coexist, never copy):**
- **Monitoring consoles:** Auriga **WWS Proactive Monitoring OMNIA** (+ Help Desk, Cash, Insight — a
  multi-console suite), **NCR APTRA Vision / Atleos Vision** (KPI dashboard; street-level **availability
  map** with zoom-aggregation; cash wallboard; "unserved customers" KPI; Intelligent Dispatch), **Diebold
  Nixdorf Vynamic View** (map-form status; **component-level** fault classes; rule engine + escalation),
  **INETCO Insight** (transaction-centric: declines/reversals/timeouts, std-deviation outlier tables,
  drill to raw log), **IR Transact/Prognosis** (payments observability, screen-composable).
- **Cash & CIT:** **NCR OptiCash** (forecast→cost-optimized schedule; −88% emergency deliveries case),
  **Fiserv Integrated Currency Manager**, **Auriga WWS Cash** (the **propose-then-approve** pattern:
  Cash Predictive proposes, Cash Console validates), **Brink's AMS** (ML forecast + live-traffic CIT
  routes — inside the CIT's black box, invisible to the bank).
- **Dispatch:** FLM/SLM split (~2h FLM response, ~98% uptime SLAs); consoles auto-create tickets with
  diagnosis attached; map-based dispatch lives in generic FSM (**Salesforce Field Service**: queue +
  Gantt + technician map) — no ATM-native console unifies it.
- **GIS/planning:** **Esri** branch-network optimization + Dashboards; **CARTO** banking (catchments,
  whitespace, cannibalization); **FFIEC geocoding** + CRA tools (Kadince, GeoDataVision, Maptitude) —
  static, never joined to live status.

**The five market gaps this design occupies:** (1) ops maps are status pins — no coverage/impact
analytics; (2) CIT routes invisible to the bank; (3) dispatch maps require a second product; (4) planning
and operations never share a map; (5) CRA/access-to-cash is a live availability × drive-time question
answered statically. **Our edge:** all four modules on one sovereign map, AI-authored, MIT template, on
Strata *or* ArcGIS — plus the incumbents' proven patterns kept: component fault trees, benchmark overlays,
propose-then-approve cash, wallboard mode.

**Standards & orgs to speak fluently:** ISO 8583; ATMIA best practices (availability conventions);
PCI DSS/PTS (why the feed stays on-prem); FFIEC CRA + geocoding (LMI tract mapping, branch-closing
90-day notice); FDIC BankFind/SoD; NCUA; Census TIGERweb + ACS; ESRI Web Map JSON (`drawingInfo`/
`popupInfo`) as the rendering contract.

## 2. UI design spec (front-loaded)

### 2.1 Layout (Template: `open-design` — "operational day suite")
- **Template** `open-design` under the freestyle charter (`strata/recipes/COMPONENT-MANIFEST.md` §10):
  registry widgets and manifest config only; two §10.3 new-widget candidates (`sla-clock`,
  `route-manifest`) ship with day-1 fallbacks (§2.6). Anti-collision: distinct from every banking sibling
  — no full-width-KPI command board (retired v1), no table-protagonist single page (fraud-aml); here the
  **map carries the decisive information on every page** (status → routes/rings → territories).
- Three `AppPage`s sharing one `layers.json` and one header (`page-nav` tabs: Pulse · Dispatch · Cash &
  Plan; ● LIVE as-of clock; type ▾ Branch/ATM/All; region ▾; EN/AR; wall toggle).

**Page 1 · PULSE** (`type:"fixed"`)
```
┌ HEADER: ● LIVE · type ▾ · region ▾ · Pulse|Dispatch|Cash&Plan · EN/AR · wall ┐
│ ESTATE RAIL (panel, 300px)     │                MAP (hero)                    │
│ accordion: ▸ ATMs (214) 97.4%  │  sites by status (arcade-classed pins,      │
│   fault mix ▍disp ▍comms ▍rdr  │  sized by txn_count) · LMI tracts low-alpha │
│ ▸ Branches (58) 99.1%          │  ┌ floating IMPACT KPI card:                │
│ ▸ regions… availability +      │  │ "pop. within 10-min drive of an OFFLINE  │
│   sparkline per group          │  │  device: 41,300" ⚠ ┐ · legend · basemap  │
│ layer-panel (docked, bottom)   │  └ status-bar: coords · zoom · as-of ┘      │
├────────────────────────────────┴─────────────────────────────────────────────┤
│ DAY STRIP (fixed 220px): ⏵ timeslider (day replay) · txns-vs-benchmark line  │
│ (bands) · EXCEPTION TICKER table (newest faults; row→flash+zoom; ⏩ Dispatch) │
└──────────────────────────────────────────────────────────────────────────────┘
```
**Page 2 · DISPATCH** (`type:"fixed"`, splitter 38/62)
```
┌ INCIDENT QUEUE (38%)               ┃              MAP (62%)                   │
│ table: sev · site · module ·       ┃  ◉ selected offline device (pulsing)    │
│ opened · SLA due (arcade tint:     ┃  ⭘ 10-min isochrone impact ring         │
│ <30min amber, breach red)          ┃  ▬▬ OSRM route engineer→site · ETA 12m  │
│ ─────────────────────────────      ┃  ▲ engineers (status) · alternates ✓/✗  │
│ DETAIL (feature-info): fault tree ·┃  ┌ DISPATCH window (floating lens):     │
│ remote-fix attempted · nearest     ┃  │ impact pop 41,300 · ETA 12m vs SLA   │
│ engineer + ETA · [Dispatch 🔶]     ┃  │ 47m → WILL MEET ✓ · [work order PDF] │
│ [Export work order] · data-actions ┃  └ sketch: area outage → multi-select ┘ │
└────────────────────────────────────┸──────────────────────────────────────────┘
```
**Page 3 · CASH & PLAN** (`type:"fixed"`, column: rail 320px top strip + map)
```
┌ PLAN RAIL: views [ Forecast | Approve ] — REPLENISHMENT table (runout_date   │
│ colored · days_of_cash) · forecast-vs-actual chart (brush) · [Approve plan   │
│ → morning-pack PDF/atlas] · carto cross-filter chips                         │
├──────────────────────────────────────────────────────────────────────────────┤
│ MAP: today's CIT run (route line + numbered stops) · demand hexbins ·        │
│ voronoi service territories · coverage weighted-overlay — plan-lens toggle   │
│ buttons switch the three analysis layers (showHide)                          │
└──────────────────────────────────────────────────────────────────────────────┘
```
- Responsive: every rail/strip stacks below the map (`responsive.small`); ticker/queue become sheets.

### 2.2 Theme (trading-desk dark, print light)
- Structured `ThemeSpec`: `mode:"dark"`, primary `#22d3ee`, success `#2dd46b`, warning `#f5b83d`, danger
  `#f2545b`; panels `#10161f` on `#0a0f16`; **tabular numerals** everywhere; a **`light`** variant for
  exports/board decks; **wall** mode enlarges the rail stats + ticker. EN + **AR/RTL** optional.
- **Status classing is arcade-driven** (one expression, all surfaces): `offline` danger · `degraded`
  (uptime<95) warning · `active` success · `planned-closure`/`relocated` muted; throughput graduates pin
  size (`classBreaks` on `txn_count`); LMI tracts ~40/255-alpha underlay; `esriSMSCircle` ATMs /
  `esriSMSSquare` branches (never `esriSMSPath`).

### 2.3 The number that is new (the differentiator KPI)
**Cash-access impact:** population inside the 10-minute drive isochrone of each OFFLINE device that has
**no working alternative** inside the same ring — `isochrones()` (or buffer fallback) → `pointsWithin`
tracts → Σ ACS pop → subtract rings containing an `active` alternate (`withinDistance`). Headlined on
Pulse, per-incident on Dispatch, and summed for the CRA/access-to-cash view. No incumbent console has it.
Classic KPIs ride along: availability % (fleet + per group), unserved-customer estimate (txn rate ×
downtime), faults/ATM/month + module mix, remote-fix rate, MTTR, SLA compliance, cash-outs, days-of-cash,
emergency deliveries, forecast accuracy.

### 2.4 Wiring (the demo is the wiring)
- **Shared source:** every widget binds `dataSource:{ sourceId:"sites" }` — rail, pins, ticker, KPIs link
  selection/filter with zero extra wires; `connections` add the cross-page verbs.
- Pulse: selectors → `filter` everything; map `extentChange` → `showStatistics` (rail + impact KPI);
  ticker `rowSelect` → `zoomTo`+`flash`, action button → `navigate` Dispatch; timeslider → time
  `definitionExpression` (day replay, stacks with filters).
- Dispatch (signature loop): queue `rowSelect` → `zoomTo`+`flash` + detail `viewInTable` + routing
  (`nearest` engineer → `route()` → draw + ETA) + isochrone ring + alternates test → dispatch window
  (`showHide`); `[Dispatch]` `buttonClick` → `updateRecord` 🔶 (else `message` toast); `[work order]` →
  `export` PDF; `sketchComplete` → `selectByGeometry` (area outage).
- Cash & Plan: chart `brush` → `filter` plan table + map; plan `rowSelect` → `zoomTo` stop; lens buttons
  → `showHide` hexbin/voronoi/overlay layers; `[Approve]` → `export` morning pack + `message`.

### 2.5 Capabilities to use (the full-capacity sweep lives in DESIGN-PROPOSAL.md §8)
`@strata/plugin-routing` (OSRM route, `nearest`, **isochrones**) · `@strata/processing`
(`withinDistance`, `pointsWithin`, `aggregate`, `hexbinDensity`, `voronoi`, `weightedOverlay`, `buffer`
fallback) · `plugin-timeslider` (day replay) · `plugin-statusbar` · `plugin-search` (site/address geocode)
· `@strata/export` (work-order + morning-pack PDF, image, CSV/GeoJSON, spec) · arcade (status/SLA
classing) · DataSource `sourceId` linking · `setDefinition` in-place filtering · structured theme + i18n ·
`data-management` publish of the merged sites+feed layer. Deliberately not used: swipe/compare, story/
exhibit, insets, 3D, reporter (rationale in the proposal).

### 2.6 §10.3 New-widget blocks (escape hatch, with day-1 fallbacks)
1. **`sla-clock`** — live mm:ss countdown chip to SLA breach. Props `{dueField, warnMins}`; emits —;
   honors `refresh`. **Fallback:** queue column `sla_due_ts` + arcade tint + `timer → refresh`.
2. **`route-manifest`** — ordered stop list synced to a `RouteResult`. Props `{stopsLayerId}`; emits
   `rowSelect`; honors `flash`. **Fallback:** `table` over the `cit-run` stops + the routing plugin panel.

## 3. Data sources (Maryland + California)

**Operational feed (synthetic in the template; the bank's in production):** `network-sites` —
`{site_id, name, type(branch|atm), status(active|degraded|offline|planned-closure|relocated), module_fault
(dispenser|card_reader|printer|depositor|comms|none), address, county, lat, lon, txn_count, uptime_pct,
cash_level, days_of_cash, runout_date, sla_due_ts, opened_ts, remote_fix_attempted, revenue, footfall}`;
`field-engineers` — `{eng_id, name, status(available|en-route|on-site), lng, lat, skills}`; `cit-run` —
`{stop_no, site_id, planned_eta, denomination_load}`. Locations seeded from **FDIC BankFind** (real
points) and joined to the synthetic columns.

**REAL DC estate (verified 2026-07-19 — the demo app's default; all keyless, CORS-open, `f=geojson`):**
| Layer | Endpoint | Fields | Count |
|---|---|---|---|
| **ATMs (real!)** | `maps2.dcgis.dc.gov/dcgis/rest/services/DCGIS_DATA/Business_Goods_and_Service_WebMercator/MapServer/31` | `NAME` (operator), `ADDRESS`, `WARD` | 381 |
| **Bank branches** | same service, layer `/0` | `NAME`, `ADDRESS`, `WARD`, `LATITUDE`, `LONGITUDE` | 182 |
| **Neighborhood clusters** | `…/DCGIS_DATA/Administrative_Other_Boundaries_WebMercator/MapServer/17` | `NAME`, `NBH_NAMES` | 46 |
| **Tract population (ACS)** | `…/OP/ACS_Demographic_Characteristics/MapServer/54` | `GEOID`, `NAME`, **`DP05_0001E`** (total pop) | 206 |

Gotchas: MapServer `f=geojson` **lowercases field names**; native SR is EPSG:26985 — always pass
`outSR=4326`; also on that service: `/37` credit unions, `/1` check-cashing. **Census Data API now
requires a (free) key on every request** — the ACS layer above carries the same totals keyless, so prefer
it. **OSM Overpass** (`amenity=atm|bank`; tags `operator/brand/name/opening_hours/fee/cash_in`) works from
real browsers (CORS `*`) but throttles bots — use as enrichment, not primary. **Overture Maps** is the
publish-path option: places categories `atms`/`banks`, divisions down to `neighborhood` (sparse
`population` column), GeoParquet on S3/Azure (`release/…/theme=places/type=place/*`) → DuckDB → Strata
Serve; population surfaces: **Kontur Population** (H3 r8 hexes, HDX, CC-BY) or **WorldPop** rasters.

**Fallback + MD/CA variants (v1 sources):**
| Need | Maryland | California | National |
|---|---|---|---|
| Branch/ATM seed | FDIC BankFind `filters=STALP:MD` | `STALP:CA` | FDIC API · NCUA |
| Deposits proxy | FDIC SoD MD | FDIC SoD CA | FDIC `/api/sod` |
| Tracts + LMI/pop | iMap `MD_CensusStatisticalBoundaries/FeatureServer/2` + `MD_AmericanCommunitySurvey` | TIGERweb + ACS | TIGERweb · ACS API |
| Counties | iMap `MD_PoliticalBoundaries` | `CA_Counties` | — |
| Competitors | FDIC BankFind (other banks) | idem | idem |
| CRA/HMDA overlay | FFIEC CRA · CFPB HMDA | idem | idem |

All EPSG:4326; FDIC/Census/ACS/FFIEC + iMap are CORS-open; seed once, **publish** the merged layer to
Strata Serve for the live view. Carry attributions. **Routing endpoints:** OSRM
`router.project-osrm.org` (demo; self-host for production); isochrones: self-hosted Valhalla `/isochrone`
or OpenRouteService (keyed) — wizard Q7.

## 4. Verify each URL first (terminal)
```bash
# FDIC branch locations (real points; LATITUDE/LONGITUDE/SERVTYPE_DESC…). Note the 301 → api.fdic.gov: use -L.
curl -sL "https://banks.data.fdic.gov/api/locations?filters=STALP:MD&fields=NAME,ADDRESS,CITY,STALP,LATITUDE,LONGITUDE,SERVTYPE_DESC&limit=3&format=json" | python3 -m json.tool | head -40
# MD census tracts: layer /2 is tracts (2020-vintage field names GEOID20/TRACTCE20 — check the suffix).
curl -s "https://mdgeodata.md.gov/imap/rest/services/Boundaries/MD_CensusStatisticalBoundaries/FeatureServer/2?f=json" \
  | python3 -c "import sys,json;d=json.load(sys.stdin);print(d.get('name'),[f['name'] for f in d.get('fields',[])][:10])"
# OSRM demo route (keyless; demo-grade — expect rate limits):
curl -s "https://router.project-osrm.org/route/v1/driving/-76.61,39.29;-76.49,38.98?overview=false" | head -c 300
# REAL DC ATMs (381) + tract population (the demo app's default estate):
curl -s "https://maps2.dcgis.dc.gov/dcgis/rest/services/DCGIS_DATA/Business_Goods_and_Service_WebMercator/MapServer/31/query?where=1%3D1&outFields=NAME,ADDRESS,WARD&outSR=4326&f=geojson&resultRecordCount=2" | head -c 400
curl -s "https://maps2.dcgis.dc.gov/dcgis/rest/services/OP/ACS_Demographic_Characteristics/MapServer/54/query?where=1%3D1&outFields=GEOID,DP05_0001E&outSR=4326&f=geojson&resultRecordCount=1" | head -c 300
```
`f=geojson` lowercases MapServer field names (client renderers lowercase; server WHERE canonical case).
The telemetry/engineer/CIT fields are **yours** — synthetic in the template, the bank's feed in production.

## Guided wizard — the prompts that assign the app's defaults

| # | Wizard question | Options → **default** | The default it assigns |
|---|---|---|---|
| 1 · App | Title & as-of? | free → **"Branch & ATM Network Operations Center"**, live clock | header + `strata:notes.asOf` |
| 2 · Region | State? | **Maryland** · California | extent + context layers |
| 3 · Pages | Which pages? `[multi]` | **Pulse + Dispatch + Cash&Plan** · subset | the `AppPage` set + page-nav |
| 4 · Network | Location source? | **FDIC-seeded + synthetic feed** · my FeatureServer · Data-Store upload | `network-sites` layer + renderers |
| 5 · SLA | FLM SLA minutes? | **120** · 60 · 240 | `sla_due_ts` synth + clock thresholds + verdicts |
| 6 · Impact | Impact ring? | **10-min drive (isochrone)** · 15-min · straight-line 3 km | the cash-access-impact computation + labeling |
| 7 · Routing | Endpoints? | **public OSRM + no isochrone (buffer fallback)** · self-hosted OSRM+Valhalla URLs | provider config + honesty labels |
| 8 · Engineers | Engineer feed? | **synthetic (6)** · my layer URL | `field-engineers` layer |
| 9 · Cash | CIT run + forecast? | **synthetic day-plan** · my feed | `cit-run` + forecast chart data |
| 10 · Overlays | Context? `[multi]` | **LMI tracts + counties** · competitors · HMDA | underlay set + CRA tagging |
| 11 · Output | Reports? | **work-order PDF + morning pack + CSV** · none | `/export` wiring |
| 12 · Theme | Mode/lang/wall? | **dark / EN / wall-capable** · light · EN+AR | ThemeSpec + lang-switch + wall |

**Then:** echo *"MD · 3 pages · FDIC+synthetic · SLA 120m · 10-min impact (buffer-fallback) · synthetic
engineers+CIT · LMI+counties · all reports · dark/EN/wall"* → confirm → run §5.

## 5. Prompt-script (run in order)
```
A. /new-app — "Branch & ATM Network Operations Center", Template: open-design per RECIPE §2.1: three fixed
   pages (pulse, dispatch, cash-plan) sharing one header (page-nav tabs, LIVE clock, type ▾, region ▾,
   EN/AR, wall). Dark trading-desk ThemeSpec (§2.2) + light print variant. Install deps + run command.

B. Data: seed FDIC BankFind points for the state; join the synthetic telemetry columns (§3 schema); add
   field-engineers + cit-run synthetic layers; /add-data MD iMap tracts (/2) + counties, low-alpha.
   Publish the merged network-sites layer; register one DataSource sourceId:"sites" shared app-wide.

C. Symbology/popups (genuine ESRI JSON): uniqueValue on {status} via the arcade classing expression
   (§2.2), classBreaks size on {txn_count}; circles=ATMs, squares=branches; popups: name, type, status,
   module_fault, uptime %, cash days, SLA due; closure_date flag on LMI branches (90-day-notice rule).

D. PULSE: left panel = accordion estate rail (per-group kpi availability + sparkline + stacked-bar fault
   mix, bound to sourceId sites) + docked layer-panel; map hero + floating impact-KPI card + legend +
   basemap + status-bar; bottom day strip = timeslider plugin (day replay) + txns-vs-benchmark line
   (bands) + exception ticker table. Wire per §2.4 (selectors→filter, extentChange→showStatistics,
   ticker rowSelect→zoomTo+flash, ticker action→navigate dispatch).

E. IMPACT ANALYSIS (@strata/processing + plugin-routing): for each OFFLINE site compute the 10-min
   isochrone (isochrones(); buffer 3km fallback labeled "straight-line") → pointsWithin tracts → Σ ACS
   pop → subtract rings with an active alternate (withinDistance). Store as impact layers; feed the
   Pulse KPI + Dispatch rings + a CRA/access view (aggregate by tract).

F. DISPATCH: splitter 38/62. Left: incident queue table (open incidents, SLA-sorted, arcade tint,
   sla_due_ts column + timer→refresh — the sla-clock fallback) + feature-info detail (fault tree,
   remote-fix, nearest-engineer ETA) + [Dispatch] + [Export work order] + data-actions. Right: map with
   engineers layer; queue rowSelect → zoomTo+flash + detail + routing (nearest engineer → OSRM route,
   draw + ETA) + impact ring + alternates ✓/✗ + dispatch window lens (impact pop, ETA-vs-SLA verdict).
   Dispatch button → updateRecord on ESRI backend, message toast otherwise; draw widget: sketchComplete →
   selectByGeometry for area outages. Work order → exportPDF (route map + fault + verdict).

G. CASH & PLAN: plan rail views [Forecast|Approve]: replenishment table (runout_date colored,
   days_of_cash) + forecast-vs-actual chart (brush→filter) + carto chips + [Approve → morning pack].
   Map: cit-run stops + OSRM route line (route-manifest fallback = stops table + routing panel); lens
   buttons showHide {hexbinDensity demand, voronoi territories, weightedOverlay coverage}. Plan
   rowSelect → zoomTo stop.

H. Cross-cutting: search plugin (site/address lookup) in the header; measure/coordinates on maps (revert
   to identify); statusbar plugin; i18n EN+AR if chosen; wall toggle enlarges rail + ticker.

I. Exports: /export report (morning pack: map + legend/scalebar/north-arrow + KPI + plan tables),
   work-order PDF, /export image, layer CSV/GeoJSON, shareable spec deep-link.

J. Verify §6; log gaps to §7.
```

## 6. Verify (benchmark to the §1 market, not just Esri)
| Check | Pass |
|---|---|
| Three pages, one map spine; page-nav + shared header; wall mode readable at 3 m | ☐ |
| Pulse: rail groups show availability/fault mix; extent change recomputes; ticker→flash/zoom/navigate | ☐ |
| Day replay: timeslider moves pins + benchmark curve together (stacks with type/region filters) | ☐ |
| **Impact KPI truthful:** isochrone (or labeled buffer) → tract pop → alternates subtracted | ☐ |
| Dispatch loop <10 s: queue click → route + ring + ETA-vs-SLA verdict lens, no instructions needed | ☐ |
| SLA tinting live (timer refresh); breach rows red; verdict flips when ETA > SLA | ☐ |
| Area-outage sketch selects multiple sites and sums impact | ☐ |
| Work-order PDF: route map + fault detail + verdict; morning pack: map + KPI + plan tables | ☐ |
| Cash page: brush scopes plan+map; CIT route visible with numbered stops; lens toggle swaps analyses | ☐ |
| Write-back only on ESRI backend; read-only path still demos (toast + local state) | ☐ |
| FDIC points real; synthetic feeds labeled; OSRM demo-grade labeled; no keyed default anywhere | ☐ |
| Beats the §1 bar: impact ring + bank-visible CIT route + SLA map-dispatch exist in NO incumbent console | ☐ (judge) |

## 6.5 Note — next step: serve the estate via Strata Serve (agreed, not yet built)

The demo app currently fetches DC GIS live per load. The production move is to **publish the merged
estate once** and let the app read our own FeatureServer (faster, offline-safe, sovereign):

1. **Extract + merge → GeoParquet** (DuckDB): pull the four DC GIS layers (ATMs `/31`, banks `/0`,
   clusters `/17`, ACS tracts `/54`), join the synthetic/bank telemetry columns onto sites, write
   `network_sites.parquet`, `neighborhoods.parquet`, `tracts_pop.parquet` (EPSG:4326;
   `ST_Read` the geojson query URLs or cache to disk first).
2. **Publish** per the repo's publish model: one `[[duckdb.datasources]]` block per layer (`service:
   "banking"`, `folder: "network-ops"`, `source_wkid=4326`; omit `object_id_field` — string `site_id` ⇒
   synthesized OID) + a metadata bundle each (`metadata.toml` w/ `displayField` + aliases +
   small `tile_fields`, `drawingInfo.json` = the §2.2 status renderer, `popupInfo.json`); restart
   `wt-server`; URLs become `…/rest/services/network-ops/banking/FeatureServer/{0,1,2}`.
3. **Repoint the app**: swap the four DC GIS URLs for the Serve endpoints (keep DC GIS as the refresh
   pipeline, e.g. a nightly re-extract). The fallback chain becomes Serve → DC GIS → synthetic.
4. **Enrichment at publish time** (cheaper than in-browser): join ACS income fields (same service `/54`,
   DP03 fields) to tag LMI tracts for the CRA lens; optionally join Overpass attributes
   (`opening_hours`, `fee`, `cash_in`) by proximity match; optionally source from **Overture**
   (places `atms`/`banks`, divisions `neighborhood`) via the S3 GeoParquet release instead of DC GIS —
   same DuckDB step, portable to any city; population via **Kontur** H3 hexes where ACS doesn't exist.
5. **Future app features unlocked** (deferred by decision, 2026-07-19): operator/brand filter,
   LMI/CRA overlay lens, Overpass attribute enrichment.

## 7. Harvest (gaps → strata-core)
- An **isochrone map widget/panel** (today: plugin function + hand-wired layer) and a first-class
  **`sla-clock`** + **`route-manifest`** (shipped here as fallbacks) — promote per manifest §10.1.
- A **FDIC BankFind seed helper** and **join-feed-to-points helper** (carried over from v1).
- **README staleness:** `plugin-routing` README omits shipped isochrones; `processing` README lists 8 of
  20 ops — code is truth; update per REFERENCE-DOCS matrix.
- A **benchmark-band chart preset** (today vs reference period) and an **availability heat-calendar**
  widget (market gap nobody ships — cheap differentiator).
- Multi-stop route optimization (CIT ordering) — OSRM `route()` takes waypoints but no TSP; note for a
  routing-plugin phase.

## 8. Sources
- **Primary product docs:** NCR APTRA Vision Buyer's Guide (hts-hightechsystems.com PDF) · DN Vynamic View
  Availability Manager product card (renome-smart.com PDF) · NCR APTRA OptiCash datasheet
  (technomedia-ltd.com PDF) · Auriga WWS Cash Management + Proactive Monitoring OMNIA (aurigaspa.com,
  atmmarketplace.com) · INETCO ATM Analytics UI walkthrough (inetco.com blog) · IR Transact (ir.com) ·
  KAL Kalignite (kal.com) · Brink's AMS forecasting/CIT (brinks-ams.com) · Fiserv Integrated Currency
  Manager (fiserv.com) · Salesforce Field Service Dispatcher Console (Trailhead) · ATMeye.iQ · Splunk
  Lantern ATM monitoring.
- **GIS/compliance:** Esri network optimization + banking hub · CARTO banking/site-selection · FFIEC
  geocoding/CRA · Kadince CRA guide · Maptitude CRA maps · FDIC branch-closing procedures.
- **Data:** FDIC BankFind Suite API · FDIC SoD · NCUA · Census TIGERweb + ACS · Maryland iMap · CA
  geoportal · OSRM (`router.project-osrm.org`) · Valhalla/ORS isochrones.
- **Internal:** `DESIGN-PROPOSAL.md` (this design's 11-part rationale) · `RECIPE-v1-ops-command.md` (the
  retired single-page dashboard) · `Banking-Financial-Services.md` (#1/#2/#3/#6) ·
  `data_sources_{md,ca,national}.md` · `strata/recipes/COMPONENT-MANIFEST.md` (§10 charter) ·
  `strata/docs/guide/app-design.md`.

---

## Modernization (parity release)
> Native to this design rather than retrofitted: structured **`theme`** (dark ops + light print),
> app-shell (`header`/`footer`, wall mode), **`dataSource.sourceId`** linking (the "sites" source drives
> rail+map+ticker+KPIs with zero wires), `timer → refresh` for the live feel, `splitter`/`panel`/`window`/
> `views` layout nodes, and arcade-driven classing shared by symbology and row tinting.
