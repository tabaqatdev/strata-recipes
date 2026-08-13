# Recipe — Hazard & Impact Zone Analyzer (Emergency, Wave-A flagship)

A reproducible path to a **hazard & impact-zone analyzer** on strata-core: overlay a **flood / fire / storm /
hazmat** hazard zone on **population, critical facilities, and infrastructure**, then answer *"who and what is
inside the zone?"* — draw or import the impacted area, **buffer/intersect + summarize-within**, and surface
the exposed people, hospitals, schools, responders, roads, and structures as KPIs, charts, and a table. This
is the Emergency cluster's impact-assessment lead app (`Emergency-Response-Watch-Center.md`, reused by the
Mitigation-Planning and Target-Hazard apps). It mirrors Esri's canonical *"define impacted area → understand
impact"* workflow — the same buffer/intersect + summarize-within pattern, AI-authored and on **Strata *or*
ArcGIS**.

> **Scope (honest).** An **analytics/serving layer** over open hazard + facility data — not a loss-estimation
> engine (no Hazus casualty/debris/dollar-loss modeling), not authoritative damage assessment. Exposure is
> **"inside the zone" (buffer/intersect/within)**, not drive-time and not a probabilistic risk score. Hazard
> geometry is **drawn, imported, or read from a live service**; there is **no editing/write-back** (read-only
> today). The conversational **"Ask the map"** layer is **off** in this release — the impact answer is
> deterministic (analyze + summarize), not an LLM query.

Run the §5 prompt-script on a fresh strata-core project (or via `/new-app`). ESRI Web Map JSON is the map
contract throughout; **everything is EPSG:4326**.

---

## 1. Study — how the market frames this

**The question an EOC / emergency manager asks:** *a hazard zone just appeared (or I drew one) — how many
people are inside, which hospitals/schools/fire stations are exposed, which roads and structures are hit, and
can I evidence it for a mitigation plan?* Under **44 CFR Part 201**, a hazard-mitigation plan's risk
assessment must **identify hazards and the assets exposed to them** — so "who/what is inside the zone" is a
compliance deliverable, not a nice-to-have.

**Reference solutions (benchmark + coexist, never copy):**
- **Esri — Emergency Management Operations** (the incumbent): draw/import an **Impacted Area**, then the
  *"Understand Impact"* tab runs a GeoEnrichment/Network-Analysis infographic — a **People** tab (population/
  demographics inside) and an **Infrastructure** tab (Critical Infrastructure, Shelters, Road Closures, USA
  Structures inside). Same pattern in **Flood Impact Analysis**, **Target Hazard Analysis**, and **Hazard
  Mitigation Planning**.
- **FEMA RAPT** (Resilience Analysis and Planning Tool, built on Esri) — the national reference: 100+ layers
  in three groups — **Infrastructure** (HIFLD), **Hazards** (NWS radar/WWA, stream gauges, wildfires, flood/
  seismic/sea-level-rise), and **Community Resilience Indicators** (ACS). A good layer taxonomy to mirror.
- **FEMA Hazus 7.0** — the standard loss-estimation methodology (flood/hurricane/earthquake/tsunami). We cite
  it as the analytical backbone but **stay short of it**: this app finds *what's exposed*, not dollar loss.
- **Adjacent vendors:** **Genasys Protect** (Esri-native evacuation/mass-notification — zone→population feeds
  it), **One Concern** (AI asset-failure resilience), **EagleView/Pictometry** (post-event imagery).

**Our edge:** AI-authored, MIT/open, runs on **Strata *or* ArcGIS**, sovereign/on-prem, and the impact answer
is a **live cross-widget analysis** (draw a zone → KPIs/charts/table update in place) rather than a static
infographic export.

**Standards, specifications & organizations to speak fluently:**
- **FEMA NFHL** — authoritative US flood zones (`FLD_ZONE`, SFHA, floodway, BFE); 100-yr / 500-yr.
- **CISA 16 Critical Infrastructure Sectors** — the taxonomy Esri's Critical Infrastructure layer is built on;
  use it for facility categorization + symbology. Governed today by **NSM-22** (National Security Memorandum
  on Critical Infrastructure Security and Resilience, signed 2024-04-30), which **superseded PPD-21**; the
  16-sector taxonomy itself is unchanged, only the underlying directive.
- **HIFLD** schema for critical facilities; **Census ACS 5-yr** for population/demographic exposure.
- **44 CFR Part 201** (hazard-mitigation planning), **NIMS/ICS** (incident framework), **FEMA Hazus** (loss
  methodology — referenced, not implemented), **OpenFEMA** disaster-declarations schema.
- Orgs: **FEMA · CISA · NOAA/NWS · USGS · NIFC · Cal OES · CAL FIRE · CGS · MD iMap · MDE · Census**.

## 2. UI design spec (front-loaded)

### 2.1 Layout (Template `chart-board` — Chart Board)
```
┌ MAP (~60%) ────────────────────────┬─ CHART RAIL (brushable) ───────────┐
│ hazard zone + sketch/buffer ·       │ population exposed (histogram)     │
│ critical facilities by CISA sector  │ exposure by sector (bars · brush)  │
│ filter chips: [sector ×][zone ×]    │ hospitals / road km (donut)        │
├─────────────────────────────────────┴────────────────────────────────────┤
│ KPIs recompute on brush & extent: pop 184k · hospitals 6 · roads 42 km   │
└──────────────────────────────────────────────────────────────────────────┘
```
- **Template** `chart-board` — **Chart Board** from `../APP-TEMPLATE-LIBRARY.md`: scaffold the `AppLayout`
  from the library skeleton and signature wiring; the bullets here only slot this recipe's content into
  the template's regions. Do **not** fall back to `split-dashboard`.
- **Wiring:** `sketchComplete` → `selectByGeometry` scopes the analysis; chart `brush`/`categorySelect` → `filter` map + sibling charts (stacking); map `extentChange` → recompute. Brushable distributions are the protagonist.
- **Hazard toggle** (flood/fire/storm/hazmat) switches the active hazard overlay + which KPIs headline;
  **Sketch → buffer** lets an operator draw a zone and set a buffer radius; **swipe** compares "all assets"
  vs "assets ∩ zone".
- **LayerPanel required** (dock it in the right column) so an operator toggles every hazard/facility/
  population layer on/off (visibility/opacity/order/zoom-to).

### 2.2 Theme (dark, hazard)
- **Base:** dark EOC (`#0e1116` bg, `#1a1f27` panels, `#e8eef5` text), high contrast, **tabular numerals**.
- **Hazard palette (semantic tokens):** flood `@flood.zone` blues (100-yr deep, 500-yr light) at low alpha;
  fire severity `#ffeda0→#feb24c→#f03b20`; storm-surge purples; hazmat plume amber; facilities by **CISA
  sector** (categorical); vulnerable tracts a sequential red ramp. **Polygon fills low alpha (~40/255)** so
  hazard + boundary layers stack readably over a dark-gray basemap.
- Bilingual **EN + AR/RTL** optional (Cairo mirror).

### 2.3 KPI cards (the differentiator)
**Population in zone** (sum ACS pop of intersecting tracts) · **Hospitals in zone** · **Schools / Shelters in
zone** · **Fire / EMS / Police in zone** · **Road km in zone** · **Structures in zone** · plus a
**weighted-vulnerability** composite (pop × SVI). Each a big-number card with a label+unit, a **status color**
by threshold, and a delta vs. the previous zone. Exception banner: *"N hospitals and M very-high-risk tracts
inside the zone"* in red → click filters map + table.

### 2.4 Charts & table
- **Exposure by facility type** (bar), **population by tract** (bar, map-linked), **facilities by CISA sector**
  (stacked), **road km by class** (bar). A chart click cross-filters map + table + KPIs.
- **Exposure table:** Asset · Type · CISA sector · Distance-to-hazard · County · In-zone? — sortable,
  per-column filter, **row → zoom + flash**, CSV/GeoJSON export; in-zone rows tinted.

### 2.5 Capabilities to use (Phases 0–5)
- **The impact analysis** — `@strata/processing`: `/analyze buffer` the drawn/imported hazard, then
  `pointsWithin`/`intersect` facilities × zone and roads × zone (**clip**) → the "in zone" counts + km;
  `aggregate` ACS tracts intersecting the zone for **population in zone**; `weightedOverlay` for a pop×SVI
  vulnerability surface. This is the **buffer/intersect + summarize-within** pattern Esri runs.
- **WIF `connections`** — hazard toggle / sector bar / tract bar → in-place `filter` of map + table + KPIs
  (`setDefinition`, no remount); table `rowSelect` → `zoomTo` + a docked `feature-info` asset card; a
  `recordsChange` from the analyze output re-drives the KPI statistics.
- **Real charts + deep table** (ECharts/SVG; server-paged table with CSV/GeoJSON export).
- **Composed export** — an **impact-assessment PDF** (legend + scalebar + north-arrow + exposure tables) for a
  44-CFR-201 mitigation packet, a **per-facility-type report**, a per-county **atlas**, and a **share**
  deep-link for the EOC.
- **Symbology/popup** genuine ESRI `drawingInfo`/`popupInfo`; facilities by CISA sector; flood by `FLD_ZONE`;
  media popup with the asset card + distance-to-hazard.
- Bus sinks the map (`bus`+`store`): selections/flashes from the table/charts light up on the canvas.

## 3. Data sources (Maryland + California)

**The hazard zone (the input):** drawn (Sketch → buffer), imported (GeoJSON/upload), or read live from a
hazard service below. **Everything reprojected to EPSG:4326.**

| Role | California | Maryland | National |
|---|---|---|---|
| **Flood zone** | Cal OES `100_Year_Floodplain/FeatureServer/4` · `500_Year_Floodplain/FeatureServer` (`services.arcgis.com/BLN4oKB0N1YSgvY8`) | iMap `Hydrology/MD_Floodplain/FeatureServer/1` (Effective FEMA Floodplain) | **FEMA NFHL** `hazards.fema.gov/arcgis/rest/services/public/NFHL/MapServer/28` *(non-CORS → proxy)* |
| **Fire** | CAL FIRE `CA_Perimeters_NIFC_FIRIS_public_view/FeatureServer/0` (`services1.arcgis.com/jUJYIo9tSA7EHvfZ`); FHSZ `services.gis.ca.gov/.../Environment/Fire_Severity_Zones/MapServer/0` | — | **NIFC/WFIGS** `services3.arcgis.com/T4QMspbfLg3qTGWY/.../WFIGS_Interagency_Perimeters_Current/FeatureServer/0` |
| **Storm / surge** | NWS WWA (below) | iMap `Weather/MD_StormSurge/MapServer` · `Weather/MD_SeaLevelRiseVulnerability/MapServer` | **NWS WWA** `mapservices.weather.noaa.gov/eventdriven/rest/services/WWA/watch_warn_adv/MapServer`; **NHC** surge `nhc.noaa.gov/gis/` |
| **Earthquake / seismic** | CGS `gis.conservation.ca.gov/server/rest/services` (`CGS_Earthquake_Hazard_Zones`) | (low seismicity) | **USGS** `earthquake.usgs.gov/earthquakes/feed/v1.0/summary/all_day.geojson` |
| **Population / vulnerability** | Census TIGERweb + ACS 2022 | iMap `Demographics/MD_CensusData/FeatureServer` · `PublicSafety/MD_VeryHighRiskCensusTracts/FeatureServer` | **Census ACS** `api.census.gov/data/2022/acs/acs5` |
| **Hospitals** | Cal OES `Hospitals/FeatureServer/0` (`services.arcgis.com/BLN4oKB0N1YSgvY8`); Caltrans `CHaviation/Hospital_Heliports/FeatureServer` (heliports) | iMap `Health/MD_Hospitals/FeatureServer/0` · `Health/MD_LongTermCareAssistedLiving/FeatureServer` | HIFLD |
| **Fire/EMS/Police** | Cal OES `_CalOES_Fire_and_Rescue_Contact_Information_Public_View` | iMap `PublicSafety/MD_Fire` · `MD_EMS` · `MD_Police` · `MD_PublicSafetyAnsweringPoints/FeatureServer` | HIFLD |
| **Schools / shelters** | Cal OES `CalOESShelters3/FeatureServer` (shelters); Cal OES `CaliforniaPublicSchools/FeatureServer/0` (schools, `services.arcgis.com/BLN4oKB0N1YSgvY8`) | iMap `Education/MD_EducationFacilities/FeatureServer` · `Education/MD_Libraries` | — |
| **Roads / closures** | Caltrans `CHhighway/SHN_Lines/FeatureServer`; QuickMap closures | iMap `Transportation/MD_RoadwayAdministrativeClassification/FeatureServer` | — |
| **Evacuation zones** | Cal OES `Combined_Statewide_Evacuation_Layer_view/FeatureServer` | — | — |

MD iMap + CA hosted-org / Caltrans are **CORS-enabled** (fetch direct). **FEMA NFHL and some NOAA/USGS
MapServers are non-CORS** → route through the Vite dev-proxy / server-side (see `docs/help/cors-and-proxy.md`).
Carry FEMA/CAL FIRE/Cal OES/CGS/NWS/Census attribution + "no warranty".

## 4. Verify each URL first (terminal)
```bash
curl -s "https://mdgeodata.md.gov/imap/rest/services/Health/MD_Hospitals/FeatureServer/0?f=json" \
  | python3 -c "import sys,json;d=json.load(sys.stdin);print(d.get('name'),'|',d.get('geometryType'),'|',[f['name'] for f in d.get('fields',[])][:10])"
```
Note the real field names driving symbology/joins/KPIs (verified 2026-07-16, `curl -s -o /dev/null -w
"%{http_code}"` on each `?f=json`): Cal OES flood is layer **`/4`** (`FLD_ZONE`/`ZONE_SUBTY`, HTTP 200); MD
floodplain is layer **`/1`** ("Effective FEMA Floodplain"); MD hospitals **`/0`** (HTTP 200, fields
`County`/`Facility_Name`/`Facility_Address`/`License_Capacity`); Cal OES `Hospitals/FeatureServer/0` (HTTP 200,
fields `NAME`/`FTYPE`/`FCODE`); Cal OES `CaliforniaPublicSchools/FeatureServer/0` (HTTP 200, fields
`PlaceName`/`Place_addr`/`Type`); CAL FIRE FIRIS perimeters carry `incident_name`/`area_acres`/
`poly_DateCurrent`. **MapServer `f=geojson` lowercases field names** (FEMA NFHL, FHSZ, NWS WWA are
MapServers) — use lowercase in client-side renderer `field` refs; server-side WHERE keeps canonical case (the
style compiler now coalesces case). USGS quakes: `mag`/`place`/`time`. **FEMA NFHL** (`hazards.fema.gov`)
sits behind a WAF that resets connections from many non-browser/datacenter clients — a direct `curl` can come
back empty even though the service is up; always route it through the server-side proxy (never assume a local
`curl` failure means the endpoint is dead) and keep last-good data on a failed refresh.

## Guided wizard — **the prompts that assign the app's defaults**

Claude runs this like an ArcGIS Instant-App wizard: ask each group, **apply the default so "accept all" builds
a complete app**, confirm a one-line summary, then run §5. In Cowork use the multiple-choice tool; in Claude
Code, a short interview (or `/recipe emergency-management_hazard-impact-analyzer`). Every answer *sets an application
default* baked into `layers.json` / the `AppLayout`.

| # | Wizard question | Options → **default** | The default it assigns |
|---|---|---|---|
| 1 · App | Title & subtitle? | free text → **"Hazard & Impact Zone Analyzer"** | header title + `strata:notes` |
| 2 · Region | Which state? | **California** · Maryland | initial extent + which facility/hazard services load (CA: Cal OES/CAL FIRE/Caltrans/CGS; MD: iMap Health/PublicSafety/Hydrology) |
| 3 · Hazard | Primary hazard? | **flood** · fire · storm/surge · earthquake · hazmat | the default hazard overlay + hazard chip + which KPIs headline; active hazard = **flood** |
| 4 · Zone source | Where's the zone from? | **draw it (Sketch → buffer)** · import GeoJSON/upload · read a live service | the zone input control + whether a live hazard layer loads; default buffer radius **1 km** |
| 5 · Exposure targets `[multi]` | What to count inside? | population · hospitals · schools/shelters · fire/EMS/police · roads · structures → **population + hospitals + schools + fire/EMS/police** | which facility/population layers load + which KPI cards render + the exposure table columns |
| 6 · Population | Population source? | **ACS tracts (sum pop of intersecting tracts)** · dot-density estimate | the `aggregate`/summarize-within op + the "population in zone" KPI + vulnerability composite |
| 7 · Facility taxonomy | Categorize facilities how? | **CISA 16 sectors** · plain type | the facilities `uniqueValue` renderer field + the "by sector" stacked chart |
| 8 · Report | Impact report? | **impact-assessment PDF (44 CFR 201 packet)** · none | wires `/export report` + per-county atlas |
| 9 · Theme | Theme + language? | **hazard-dark / EN** · light · EN+AR (RTL) | `@strata/theme` `hazard` preset + `lang-switch` |

**Then:** Claude echoes *"CA · flood · draw-zone 1 km · {population + hospitals + schools + fire/EMS/police} ·
ACS tracts · CISA sectors · impact PDF · hazard-dark/EN"* and, on confirmation, runs §5 — so the app opens
**fully configured**.

## 5. Prompt-script (run in order)
```
A. /new-app — a "Hazard & Impact Zone Analyzer" `chart-board` template (map ~62% / panels ~38%), plus a "wall" mode.
   Dark hazard theme (@strata/theme hazard preset). Header with title + a hazard chip (flood/fire/storm/
   hazmat) + a live clock. Bilingual EN/AR optional. Install deps + give me the run command.

B. /add-data the hazard + context for the chosen region (from §3): the flood zone (CA Cal OES 100/500-yr, or
   MD MD_Floodplain/1, or FEMA NFHL via the dev-proxy), plus the facility layers chosen in wizard Q5 (hospitals,
   schools/shelters, fire/EMS/police) and the ACS population tracts. Set the initial extent to the state. Keep
   the light layers initial; add heavy context (roads, FHSZ) to the store AFTER onReady, staggered, default-off.

C. The zone input: add a Sketch control (draw polygon) + /analyze buffer with a radius control (default 1 km),
   OR import a GeoJSON zone, OR read a live hazard service. /symbology the hazard zone (flood by FLD_ZONE blues
   low-alpha; fire severity ramp). /symbology facilities uniqueValue by CISA sector; esriSMSCircle only.
   /popup facilities: title {name}; show type, sector, address, and (post-analysis) distance-to-hazard.

D. The impact (the answer): using @strata/processing — /analyze pointsWithin facilities × the zone (in-zone
   flag + count per type/sector); /analyze intersect roads × zone → clip → road km in zone; /analyze aggregate
   ACS tracts intersecting the zone → population in zone; optional /analyze weighted-overlay pop × SVI for a
   vulnerability surface. Write the in-zone tag back onto each asset.

E. /panel statistics as KPI cards (surface=page): Population in zone, Hospitals in zone, Schools/Shelters,
   Fire/EMS/Police, Road km in zone, Structures in zone, weighted-vulnerability composite. Exception banner:
   hospitals + very-high-risk tracts inside the zone.

F. /panel chart: exposure by facility type (bar), population by tract (map-linked bar), facilities by CISA
   sector (stacked), road km by class (bar). /panel table exposure (Asset, Type, Sector, Distance-to-hazard,
   County, In-zone) — sortable, filter, row→zoom+flash, CSV/GeoJSON; in-zone rows tinted.

G. WIF: author AppLayout.connections — hazard toggle + sector bar + tract bar → in-place filter of map + table
   + KPIs (setDefinition, no remount); table rowSelect → zoomTo + a docked feature-info asset card; the analyze
   output recordsChange re-drives the KPI stats. Add a swipe comparing all-assets vs assets∩zone.

H. Controls + export: navigation/scale/geolocate, a Legend + BasemapPanel on canvas, Measure (returns to
   identify). Dock a LayerPanel in the right column (REQUIRED). Wire /export report (an impact-assessment PDF:
   legend/scalebar/north-arrow + exposure tables for a 44 CFR 201 packet), a per-county atlas, /export image,
   and a share deep-link. Add the wall toggle.
```

## 6. Verify (benchmark to Esri EMO / RAPT)
| Check | Pass |
|---|---|
| Hazard zone drawn/imported/read; buffer radius adjustable; flood/fire styled low-alpha | ✅ |
| "Population in zone" + facility/road/structure counts via pointsWithin/intersect/aggregate; matches a manual spot check | ✅ |
| Hazard toggle + sector/tract charts cross-filter map + table + KPIs **in place** (no remount) | ✅ |
| Exception banner (hospitals + very-high-risk tracts in zone) surfaces first; click filters | ✅ |
| Facilities categorized by **CISA 16 sectors**; exposure table row → zoom + flash; CSV/GeoJSON export | ✅ |
| **Impact-assessment PDF** exports with legend/scalebar + exposure tables (44 CFR 201 packet); per-county atlas | ✅ |
| FEMA NFHL loads via proxy (non-CORS) and keeps last good data on a failed refresh | ✅ |
| Layer panel toggles every hazard/facility/population layer; measure returns to identify | ✅ |
| Runs on Strata **and** ArcGIS; on-prem, no data egress | ✅ (judge) |

**On-par-or-better:** matches Esri Emergency Management Operations' *"Understand Impact"* People +
Infrastructure tabs and RAPT's Infrastructure/Hazards/Resilience layer taxonomy, and **exceeds on** the
AI-authored build, live cross-widget interactivity (draw a zone → everything updates in place), CISA-sector
styling, and the one-click 44-CFR-201 impact packet — MIT, on Strata or ArcGIS.

## 7. Harvest (gaps → strata-core)
Log anything missing: a first-class **summarize-within** op (sum ACS pop of tracts intersecting a drawn zone
in one call), a **draw-zone → analyze** binding (Sketch geometry straight into `/analyze buffer`+`within`), a
**CISA-sector palette** preset in `@strata/theme`, a **distance-to-hazard** helper written back onto each
asset, a **44-CFR-201 impact-report template** for `/export report`, and a **"population estimate for an
arbitrary polygon"** (areal-weighted ACS) helper. Out of scope this wave (name, don't build): **Hazus** dollar/
casualty loss, **drive-time** evacuation catchments (needs routing), live **DINS** damage write-back (editing
is the auth/editing wave — read-only today), and the conversational **"Ask the map"** query (off this release).

## 8. Sources
- Esri: [Emergency Management overview](https://www.esri.com/en-us/industries/emergency-management/overview) ·
  [Emergency Management Operations solution](https://www.esri.com/en-us/c/industry/public-safety/emergency-management-operations-solution) ·
  [Use Emergency Management Operations](https://doc.arcgis.com/en/arcgis-solutions/latest/reference/use-emergency-management-operations.htm) ·
  [Flood Impact Analysis](https://doc.arcgis.com/en/arcgis-solutions/11.3/reference/introduction-to-flood-impact-analysis.htm) ·
  [Target Hazard Analysis](https://doc.arcgis.com/en/arcgis-solutions/latest/reference/introduction-to-target-hazard-analysis.htm) ·
  [Hazard Mitigation Planning](https://doc.arcgis.com/en/arcgis-solutions/latest/reference/configure-hazard-mitigation-planning.htm) ·
  [Automatically summarizing disaster impact (ArcGIS blog)](https://www.esri.com/arcgis-blog/products/api-rest/public-safety/automatically-summarizing-the-potential-impact-of-disasters-with-arcgis)
- FEMA / standards: [RAPT](https://www.fema.gov/emergency-managers/practitioners/resilience-analysis-and-planning-tool) ·
  [RAPT hub](https://rapt-fema.hub.arcgis.com/) · [Hazus](https://www.fema.gov/flood-maps/products-tools/hazus) ·
  [National Flood Hazard Layer](https://www.fema.gov/flood-maps/national-flood-hazard-layer) ·
  [CISA Critical Infrastructure Sectors](https://www.cisa.gov/topics/critical-infrastructure-security-and-resilience/critical-infrastructure-sectors) ·
  [NSM-22 — National Security Memorandum on Critical Infrastructure Security and Resilience (2024, supersedes PPD-21)](https://www.cisa.gov/topics/critical-infrastructure-security-and-resilience/national-security-memorandum-critical-infrastructure-security-and-resilience) ·
  44 CFR Part 201 · Census ACS 5-yr.
- Vendors / adjacent: [Genasys–Esri](https://genasys.com/genasys-partner-network/esri/) ·
  [ICF hazard mitigation](https://www.icf.com/work/disaster-management/hazard-mitigation) · [One Concern](https://oneconcern.com/en/).
- Internal: `Emergency-Response-Watch-Center.md`, `2-Strata-Core-Plan-and-Spec.md`,
  `data_sources_{ca,md,national}.md`, `docs/reference/components.md`, `docs/reference/human-language.md`.

---

## Modernization (parity release)

> Applies the Experience-Builder-parity features (Phases 1–7). See `strata/recipes/MODERNIZATION.md` + `COMPONENT-MANIFEST.md` §8. Cross-cutting for every solution app: a structured **`theme`** (brand/hazard palette → hover/active/focus states + motion), app-shell (`header`/`footer`/`splash`), and `dataSource.{ sourceId }` linking (widgets sharing a source link with **no `connections`**).

- **Authored analysis:** the **`analysis`** widget is the core — `intersect`/`pointsWithin`/`buffer` over the hazard footprint × population/facilities; result publishes as an **output DataSource** for a linked `table` + KPIs.
- **Draw the zone:** `sketchComplete → selectByGeometry`; import a zone via **`FileDataSource`** (GeoJSON).
- **Report:** a **`window`** impact summary; hazard **`theme`**.
