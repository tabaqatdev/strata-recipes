# Recipe — Network Operations Map (Utilities, Energy & Telecom)

*Shipped name: **Network Operations Map** (internal codename "Grid Watch" appears in some prose below).*

A reproducible path to a **shared, keyless network-operations picture** on strata-core, for
California: a **system-condition strip** (grid posture · customers-out · active-PSPS · peak wind), a
full-bleed **network map** fusing electric transmission + service territories + substations with the
**live state** (Cal OES ~5-minute outages, PSPS areas, NDFD wind, CPUC HFTD fire-threat tiers), a
**stress watch rail** ranking territories by wind × fire-threat × dependent density, and a
**de-energization lens** that shows **who loses power** (electricity-dependent residents, critical
water/wastewater, transmission inside) when you select a territory or **sketch a footprint** — with the
**PSPS notice & operating brief generated from live state**. Designed **open-design** from 2026 research
beyond ESRI (§1), with a second research pass that **internet-verified the CA data endpoints before any
synthesis**. Full rationale: `DESIGN-PROPOSAL.md`. Data catalog: `DATA.md` + `uno-catalog-ca.json`.

> **Scope (honest).** This is **situational awareness, not SCADA control**. No live telemetry, no true
> feeder topology, no switching/FLISR/Volt-VAR, no DER dispatch — it fuses **public** feeds to inform a
> human decision, never to actuate a device, and it sits outside the NERC/CIP bulk-electric perimeter.
> The de-energization consequence fan-out is **geographic area-containment (heuristic)** — every item
> is confidence-tagged and the lens says so. Substations are **OpenStreetMap `power=substation` (ODbL —
> attributed)**, the only public statewide set; CEC's authoritative layer is **499 token-gated** and
> never synthesized. PSPS layers legitimately return **0 rows off-event** — the app ships an explicit
> empty state plus a 2019 replay. CAISO/EIA grid condition has **no geometry** — a header chip, not a
> map layer. Posture/notice writes need an ESRI backend 🔶 (local + toast otherwise). Coexistence with
> ArcGIS Utility Network (the model of record) and the ADMS incumbents — never "replace."

---

## 0. Provenance

| | |
|---|---|
| **Source** | `https://tabaqat.net/solutions` — **Utilities, Energy & Telecom** section (the one solution without a scaffold; created fresh at the user's request) |
| **Name** | **Network Operations Map** (codename "Grid Watch") |
| **Tagline** | "The live operating picture of the whole grid — condition, outages, and the de-energization decision on one board" |
| **Created** | 2026-07-26 |
| **Template** | `open-design` — silhouette **"operating-picture"** (per `DESIGN-REQUEST-PROMPT.md`) |

---

## 1. Study — how the market frames this (research 2026-07-26, beyond ESRI)

**The question the buyer asks:** *"Is the grid healthy right now, where is it stressed, and — before we
pull the switch — who loses power?"*

**Reference solutions (benchmark + coexist, never copy).** The market has two walls and an empty middle:
- **ESRI — ArcGIS Utility Network / Electric Utility Network Foundation** is the *asset & connectivity
  **model of record***, shared across design/operations/planning — and is **explicitly not** SCADA,
  real-time operations, or outage response. It supplies the model the ADMS vendors consume.
- **The ADMS/EMS/OMS incumbents** own the real-time control room: **GE Vernova GridOS** ("grid
  orchestration": AEMS + ADMS + WAMS), **Schneider EcoStruxure ADMS** (DMS + OMS + SCADA on one model,
  Guidehouse #1), **Oracle Utilities NMS** (OMS + DMS + ADMS, 6 of the top-10 US utilities), **Survalent
  SurvalentONE** (SCADA + OMS + DMS, strong in muni/co-op), **Siemens Spectrum Power**, **Hitachi Energy
  Network Manager**, **AspenTech OSI monarch/Spectra**. All are **licensed, closed, single-tenant, and
  SCADA-integrated** — walled inside one utility's cyber perimeter. **DERMS** layers on to orchestrate
  distributed resources within distribution constraints.
- **The open feeds sit siloed:** **PowerOutage.us** (aggregated outage layer, ~10-min), **CAISO Today's
  Outlook** (real-time BA demand/net-demand/reserves), **EIA-930 Hourly Electric Grid Monitor** (hourly
  demand/generation for all 65 lower-48 BAs), **gridstatus.io** (open REST for ISO load/fuel-mix),
  **CPUC HFTD fire-threat map** + each **IOU's 7-day PSPS forecast**.

**Our edge (the empty intersection).** The people who most need a *cross-utility, system-level*
operating picture — an **EOC, a CPUC analyst, a mutual-aid coordinator, a small muni/co-op, an overseas
utility** — are exactly the people **not** sitting at a SCADA-integrated ADMS. No incumbent sells a
**shared, keyless, sovereign** picture that fuses the *public* feeds into one board, and **nobody joins
the PSPS de-energization decision to dependent-customer consequence** (electricity-dependent residents +
critical water) on one map. AI-authored, MIT/open, runs on **Strata *or* ArcGIS**, on-prem-capable,
cross-widget interactive. Coexistence, never "replace ArcGIS."

**Standards & bodies to speak fluently.** **IEC CIM** (61970 EMS/transmission · 61968 DMS/distribution)
the network-model lingua franca; **IEEE 2030.5 / CA Rule 21** (DER interconnection); **IEEE 1366**
reliability indices (**SAIDI · SAIFI · CAIDI · MAIFI · ENS**, with the T_MED major-event-day
normalization); **NERC** (mandatory bulk-system reliability standards: TOP/IRO/BAL/EOP/CIP) under
**FERC**; **CAISO** (the CA balancing authority — alerts/warnings/Flex Alerts); **CPUC** — **GO 166**
(emergency-preparedness standards, annual reports due Apr 30), the **HFTD** fire-threat tiers (T1 HHZ /
T2 elevated / T3 extreme), **PSPS** oversight, and **Wildfire Mitigation Plans** (Energy Safety/OEIS).
ADMS functions to name: **FLISR**, **Volt/VAR Optimization**, network reconfiguration, state estimation.

**Honest scope (what this is *not*).** Not a system of record (that's ArcGIS UN), not a control system
(that's the ADMS), not authoritative outage cause/ETR (scraped aggregates), not a reproduction of a
utility's private PSPS Fire-Potential-Index model. It **visualizes published forecasts and risk
overlays and fuses public feeds** — situational awareness that *informs* a decision.

## 2. UI design spec (front-loaded)

### 2.1 Layout (Template: `open-design` — "operating-picture")
- Charter/manifest §10; three §10.3 candidates (`condition-strip`, `deenergization-lens`, `feed-bar`)
  with day-1 fallbacks (§2.6). **Anti-collision (utilities sector):** distinct from all five siblings —
  not `ranked-list` (outage-incident owns the customers-affected rank; here the rank is a *stress watch
  rail* subordinate to the map + lens), not `chart-board` (asset-condition), not `insets-grid`
  (coverage-capacity), not `triage-console` (field-crew work orders), not `sidebar-explorer`
  (water-environmental). Across the pack: not `ops-command` (KPI-tile board — here a *condition strip* +
  network map + decision lens) and not `scoreboard`. Harvest candidate: **`operating-picture`**.

```
┌ HEADER: ● LIVE · CAISO ▾NORMAL · utility ▾ · op-period ▾ · wall · [Stage PSPS notice] ─────────────┐
│ CONDITION STRIP:  ⚡Reserve 24%  ·  Customers out 61,240 ↘  ·  PSPS active 0  ·  Peak wind 47 mph ↗  │
├────────────────────────────────┬───────────────────────────────────────────────────────────────────┤
│ STRESS WATCH RAIL (320px)      │                 FULL-BLEED NETWORK MAP                             │
│ ▲ North Valley OA   STRESS 82  │  transmission (kV-weighted) · territories (IOU/POU) · substations   │
│   47mph · HFTD T3 · 1.2k dep.  │  ▲ (OSM) · outage areas (Cal OES ~5-min, age-badged) · PSPS areas   │
│ ▲ Foothills Circuit STRESS 74  │  · NDFD wind grid · HFTD tiers ·                                    │
│ ● Coastal Metro    STRESS 31   │  DE-ENERGIZATION LENS (select territory / sketch footprint):        │
│   sorted wind×HFTD×dependents  │   ⚡→ 3,180 electricity-dependent residents · 2 critical WWTP        │
│   click → map filters + lens   │   ⚡→ 5 water-system areas · 41 mi transmission (heuristic)          │
├────────────────────────────────┴───────────────────────────────────────────────────────────────────┤
│ FEED BAR: CalOES outages ⬤4m · CAISO NORMAL ⬤5m · NDFD wind ⬤hourly · PSPS ⬤event-only · OSM ⬤static │
└─────────────────────────────────────────────────────────────────────────────────────────────────────┘
```
- Pages: `ops` (fixed) + `notice` (print/export view). Catalog drawer (the CA network library)
  inherited from Lifelines/COP. Phone: condition strip → header; watch rail → scroll body; row tap →
  map+lens full-screen sheet; [Stage notice] pinned. Every side-by-side `row`/`splitter` has
  `responsive.small`.

### 2.2 Theme
Dark control-room base (`mode:"dark"`), one electric-amber `primary` `#f5a623`; semantic `danger
#e03131` (out / HFTD T3), `warning #f5c542` (elevated / stress), `success #2b8a3e` (nominal), `info
#4dabf7` (water/dependents). **Stress is a computed arcade tint** (wind×HFTD×dependents); **suggested-by-
feed = outlined chip, operator-confirmed = solid** (epistemic state as a pixel, harvested from
Lifelines). kV-weighted transmission widths; outage areas age-badged; wind grid a low-alpha wash
(~40/255 fills). Per-feed freshness dots + cadence labels. Wall mode; EN+AR/RTL (Gulf utility audiences).

### 2.3 The differentiators
1. **Public-feed fusion into one operating picture** — the cross-utility system-level view no
   single-tenant ADMS sells.
2. **The condition strip** — grid posture from CAISO/EIA + statewide customers-out + active-PSPS +
   peak-wind, a *posture bar* (not a nine-tile KPI wall).
3. **The stress watch rail** (weighted-overlay): territories/areas scored wind × HFTD × dependent
   density — the proactive PSPS triage, subordinate to the map.
4. **The de-energization lens** (`pointsWithin`): select a territory or **sketch a footprint** →
   who-loses-power fan-out (dependent residents + critical water + transmission inside), confidence-
   tagged. Labeled heuristic until feeder topology exists.
5. **Generated PSPS notice & operating brief** — from live state, zero re-typing.
6. **Honest empty/epistemic states** — PSPS "no active event," OSM-substation attribution, CAISO
   "no geometry" chip, suggested-vs-confirmed.

### 2.4 Wiring (→ `AppLayout.connections`, implemented in §5)
rail `rowSelect` → map `filter`/`zoomTo`/`flash` + lens `showHide`/`showStatistics`; outage/PSPS
`featureSelect` → lens `selectByGeometry`/`viewInTable`; draw `sketchComplete` → lens what-if; lens item
`rowSelect` → `zoomTo`/`flash`; `timer` → `refresh` (~5-min); CAISO/utility ▾ → `filter` all; map
`extentChange` → condition `showStatistics`; [Stage notice] → `export`; confirm posture `buttonClick`
→ `updateRecord` 🔶/`message`; drawer `rowSelect` → `showHide`.

### 2.5 Capabilities (sweep in DESIGN-PROPOSAL §8)
weighted-overlay (stress score) · processing `pointsWithin` (who-loses-power join) + `buffer`/
`aggregate`/`overlay` · draw (sketch footprint) · routing (nearest alternate-fed critical facility,
sparingly) · timeslider (PSPS 2019 replay / outage history) · export (staged notice + brief + map pack)
· arcade (stress + suggested/confirmed) · RestDataSource/sourceId · timer→refresh · status-bar/legend/
data-actions · theme+i18n · feature-arcgis/auth 🔶. Deliberately unused: swipe/compare, media-pager,
scroll-story, insets-grid, near-me, elevation/embed/video.

### 2.6 §10.3 New-widget blocks (fallbacks ship day 1)
1. **`condition-strip`** → *fallback* `row` of `kpi` + `sparkline` + arcade tint on a shared `sourceId`.
2. **`deenergization-lens`** → *fallback* `draw`/`selectByGeometry` → `pointsWithin` → grouped `table` +
   `button` → `export`.
3. **`feed-bar`** → *fallback* `text` chips + `timer → refresh` (reused from Lifelines' harvest).

## 3. Data sources

*A role × region matrix. California is the built demo; MD/National columns note the portable analog.
Every CA cell is a curl-verified endpoint — see §4 and `DATA.md` for the exact fields, counts and
gotchas. Everything EPSG:4326 (reproject on ingest).*

| Role | California (built demo) | National / portable analog |
|---|---|---|
| **Transmission** | CEC `Transmission_Line/FeatureServer/**2**` — 6,839; `OBJECTID`, `Name/kV`(string)`/Owner/Status/Length_Mile` | HIFLD `Electric_Power_Transmission_Lines` (NASA-NCCS mirror; env-blocked here) |
| **Service territories** | Cal OES IOU `…UtilityServiceTerritories_ExportFeatures/FeatureServer/**1**` — 6 (`Name`); non-IOU `…/1` — 53 (`Acronym/Utility/Type`) | HIFLD Retail Service Territories |
| **Substations** | **OpenStreetMap `power=substation`** — 3,964 (342 nodes + 3,622 ways) via `overpass.kumi.systems`; **ODbL** | OSM Overpass anywhere; HIFLD substations (US) |
| **Balancing / paths / LSE** | CEC `Balancing_Authority` (9, `NAME`) · `Major_Transmission_Paths` (40, `PATH` = CAISO-OASIS join) · `ElectricLoadServingEntities_IOU_POU` (53, `Sales_GWh_1990..2025`) | EIA-860; ISO/RTO footprints |
| **Live outages** | Cal OES `Power_Outages_(View)/FeatureServer` /0 incidents (63) · /1 areas (33) · /2 by-county (58); **~5-min** (`cacheMaxAge 300s`); `ImpactedCustomers/Cause/EstimatedRestoreDate`, county `Number_Impacted_Customers` + `Impacted_Cutomers_Not_Planned` (sic). **Customers-out KPI sums `ImpactedCustomers` on `/0`** — the `/2` rollup `Number_Impacted_Customers` is a View that **intermittently returns a null aggregate** (7,274 ↔ null). | **PowerOutage.us** GeoJSON (national, ~10-min) |
| **PSPS de-energization** | `Statewide_PSPS_Current_Active_Outage_Areas_(Public)/0` — **0 off-event** (`County/Status/UtilityCompany/EventName`); sibling `…Active…/0` — ALL-CAPS `UTILITY/OUTAGE_ID/COUNTY/CITY` (incompatible) | IOU PSPS 7-day forecast portals (PG&E/SCE/SDG&E) |
| **Wind threat** | Cal OES `NDFD_Maximum_Wind_Speed_Today_View/FeatureServer/**2**` — 17,263 grid; `OBJECTID`, `Value` (mph) | NWS NDFD grids (national) |
| **Fire-threat tiers** | Cal OES `CPUC_Fire_Threat_Areas/FeatureServer` /0 Tier 3 (1) · /1 Tier 2 (1); **OID `FID`; fields `FID/FID_1` only — tier is in the layer name, NO `TIER`/`THREAT` column**; full HFTD at `services2.arcgis.com/VofPZYDe2pLxSP5G …CPUC_High_Fire_Threat_District/0` | state WUI / fire-risk layers |
| **Grid condition** | **EIA-930** `api.eia.gov/v2/electricity/rto/region-data/data/` `respondent=CISO` (keyless DEMO_KEY; **no geometry**); CAISO Today's Outlook (endpoint unconfirmed from sandbox) | EIA-930 any BA; gridstatus.io ISO REST |
| **Dependents (residents)** | emPOWER `…Electrically_Dependent_Benificiaries_Zip_Codes/0` — 1,758; **misspelled service; corrupt aliases — bind raw** `ZIP_CODE/POPULATION/power_dependent_devices_dme/athome_dialysis_3mo/ventilators_13mo` | HHS emPOWER (national ZIP dataset) |
| **Dependents (water)** | `CriticalWasteWaterTreatmentPlants/0` — 6 (**OID `OBJECTID_1`**; `CWP_NAME/CWP_CITY/CWP_STATUS`) · `California_Drinking_Water_System_Area_Boundaries/0` — 4,672 (**OID `FID`**; `PWSID/NAME/D_POPULATI`) | EPA ECHO / SDWIS |

All CA cells verified 2026-07-26 (curl `?f=json`). CORS: the arcgis.com hosted orgs are CORS-open; the
Overpass mirror + EIA are reachable server-side (fetch at build/proxy, not in-browser). Reproject any
non-4326 source on ingest. Full field lists + gotchas in **`DATA.md`**.

## 4. Verify each URL first (terminal)

```bash
CALOES="https://services.arcgis.com/BLN4oKB0N1YSgvY8/arcgis/rest/services"
CEC="https://services3.arcgis.com/bWPjFyq029ChCGur/arcgis/rest/services"
# Transmission — CEC layer /2 (OBJECTID, Length_Mile; kV is a STRING):
curl -s "$CEC/Transmission_Line/FeatureServer/2/query?where=1%3D1&returnCountOnly=true&f=json"   # 6839
# IOU territories — LAYER /1 (the /0 path 404s; field Name only):
curl -s "$CALOES/CAInvestorOwnedUtilityServiceTerritories_ExportFeatures/FeatureServer/1?f=json" | py -c "import sys,json;d=json.load(sys.stdin);print(d['objectIdField'],[f['name'] for f in d['fields']])"
# Live outages by county — the fusion feed (misspelled field is real):
curl -s "$CALOES/Power_Outages_(View)/FeatureServer/2/query?where=1%3D1&outFields=NAME,Number_Impacted_Customers,Impacted_Cutomers_Not_Planned&f=json" | head -c 600
# PSPS current — expect 0 rows off-event (build the empty state):
curl -s "$CALOES/Statewide_PSPS_Current_Active_Outage_Areas_(Public)/FeatureServer/0/query?where=1%3D1&returnCountOnly=true&f=json"   # 0
# NDFD wind — LAYER /2 only; Value in mph:
curl -s "$CALOES/NDFD_Maximum_Wind_Speed_Today_View/FeatureServer/2/query?where=1%3D1&returnCountOnly=true&f=json"   # 17263
# CPUC fire-threat — tier is in the layer NAME; fields are FID/FID_1 only:
curl -s "$CALOES/CPUC_Fire_Threat_Areas/FeatureServer/0?f=json" | py -c "import sys,json;d=json.load(sys.stdin);print(d['name'],d['objectIdField'],[f['name'] for f in d['fields']])"
# emPOWER — misspelled service; bind RAW fields, aliases are corrupt:
curl -s "$CALOES/emPOWER_Electrically_Dependent_Benificiaries_Zip_Codes/FeatureServer/0?f=json" | py -c "import sys,json;d=json.load(sys.stdin);print(d['objectIdField'],len(d['fields']),'fields')"
# Critical wastewater — OID is OBJECTID_1, not OBJECTID:
curl -s "$CALOES/CriticalWasteWaterTreatmentPlants/FeatureServer/0?f=json" | py -c "import sys,json;d=json.load(sys.stdin);print(d['objectIdField'])"   # OBJECTID_1
# OSM substations (main endpoint 406s here — use the kumi mirror; ODbL):
curl -s -H "Accept: application/json" --data-urlencode 'data=[out:json][timeout:180];area["ISO3166-2"="US-CA"][admin_level=4]->.ca;(node["power"="substation"](area.ca);way["power"="substation"](area.ca););out count;' https://overpass.kumi.systems/api/interpreter
# EIA-930 grid condition (keyless DEMO_KEY; no geometry):
curl -s "https://api.eia.gov/v2/electricity/rto/region-data/data/?api_key=DEMO_KEY&frequency=hourly&data[]=value&facets[respondent][]=CISO&facets[type][]=D&sort[0][column]=period&sort[0][direction]=desc&length=1"
```
Gotchas: CEC `Substation` returns **499 Token Required** — use OSM. `Power_Outages` is a **View** (5-min
cache), `IncidentId` alias misspelled "Indicent ID", area `displayField` misconfigured to `Shape_Length`.
`California_Water_Delivery_Pump_and_Power_Houses` is **layer /1** (KML-derived, OID `OID`, `Name`+HTML
`PopupInfo` only). Everything `outSR=4326`.

## Guided wizard — the prompts that assign the app's defaults

Claude runs this like an ArcGIS Instant-App wizard: ask each group, **apply the default so "accept all"
builds a complete app**, confirm a one-line summary, then run §5. Launch with `/recipe
utilities_network-operations`.

| # | Wizard question | Options → **default** | The default it assigns |
|---|---|---|---|
| 1 · App | Title & as-of? | → **"Network Operations Map — California"**, today | header + `strata:notes.asOf` |
| 2 · Region | Which area? | **statewide + utility ▾** · one operating area | initial extent + territory filter |
| 3 · Network | Asset spine? | **transmission + territories + OSM substations** · +LSE/paths | map layers |
| 4 · Live feed | Fusion feed? | **Cal OES outages (~5-min)** · my utility feed | the who's-out join source |
| 5 · Threat | PSPS driver? | **NDFD wind + CPUC HFTD** · custom weather | stress-score inputs |
| 6 · Stress score | Weights? | **wind×HFTD×dependents (documented)** · custom | weighted-overlay arcade |
| 7 · Consequence | Dependents? `[multi]` | **emPOWER + critical WWTP + drinking-water areas** | lens fan-out layers |
| 8 · Posture store | Confirm/override? | **local (template)** · ESRI layer 🔶 | writes path |
| 9 · Outputs | `[multi]` | **PSPS notice PDF + operating brief + CSV** | export wiring |
| 10 · Theme | | **control-room dark / EN+AR / wall** | ThemeSpec |

## 5. Prompt-script (run in order)
```
A. python build_uno_catalog.py — confirm library/role-tagged counts + external verified. RestDataSource.
B. /new-app — "Network Operations Map — California", open-design per §2.1: pages ops + notice; control-room dark
   ThemeSpec (§2.2) with stress + suggested/confirmed + freshness arcade tokens; header (LIVE, CAISO ▾,
   utility ▾, op-period, wall, Stage PSPS notice).
C. CONDITION STRIP: kpi+sparkline row — reserve/condition (CAISO/EIA chip), customers-out (Cal OES /0
   incidents `ImpactedCustomers` sum — NOT the /2 rollup View, which flickers null; retry-on-null),
   active-PSPS (PSPS current count), peak wind (NDFD /2 max), transmission miles (CEC /2 Length_Mile sum).
D. NETWORK MAP: transmission (kV-weighted) + IOU/POU territories + OSM substations + Cal OES outage
   areas (~5-min timer→refresh, age badge) + PSPS areas (empty-state aware) + NDFD wind wash + HFTD tiers.
E. STRESS RAIL: weighted-overlay(wind × HFTD × dependents) per territory/area → ranked rows (suggested
   tint; confirm→solid via updateRecord 🔶 else local+toast); rowSelect → map filter/zoom/flash + lens.
F. DE-ENERGIZATION LENS (window): territory/area featureSelect or draw sketchComplete → pointsWithin
   fan-out (emPOWER ZIPs, critical WWTP, drinking-water areas, transmission inside) → confidence-tagged
   table + [Stage notice]; "situational awareness, not a switching order" label pinned.
G. FEED BAR: freshness chips + cadence (~5-min / hourly / event-only / static); CAISO/EIA condition as
   labeled header chip until a parsed feed exists.
H. [Stage PSPS notice] → export: the notice + operating brief from live state (condition, top-stress
   circuits, who-loses-power counts, feed vintages) + map pack. Catalog drawer inherited. Wall; EN/AR.
I. Verify §6; log gaps to §7.
```

## 6. Verify
| Check | Pass |
|---|---|
| Condition-strip KPIs live: customers-out (from /0 `ImpactedCustomers`, not the flaky /2 View), peak wind, PSPS, transmission — and they re-scope to a selected territory | ☐ |
| Stress rail: weighted score wind×HFTD×dependents; suggested tint vs confirmed solid; overrides attributed | ☐ |
| De-energization lens: select/sketch → real dependents + critical water + transmission inside; heuristic label visible | ☐ |
| PSPS off-event renders an explicit "no active PSPS" empty state (never fake data); 2019 replay works | ☐ |
| Substations labeled OpenStreetMap (ODbL); CEC 499-token layer never silently substituted | ☐ |
| CAISO/EIA condition is a cadence-labeled chip, not a faked map layer | ☐ |
| Staged PSPS notice + brief generated from state; feed vintages printed; zero re-typing | ☐ |
| Every `layerId` + field verified against the service (§4/DATA.md); OBJECTID is the OID | ☐ |
| `responsive.small` collapses every row/splitter; basemap keyless; EPSG:4326; runs on Strata **and** ArcGIS | ☐ |
| Beats the bar: public-feed fusion + PSPS decision + dependent-consequence exist together in NO incumbent (ADMS, PowerOutage.us, ArcGIS UN) | ☐ (judge) |

## 7. Harvest (gaps → strata-core)
- **`operating-picture`** silhouette (live-network ops board — generalizes to water utilities, telecom
  NOCs, ports, pipelines); **`condition-strip`** + **`deenergization-lens`** + **`feed-bar`** widgets
  (fallbacks shipped) — §10 candidates. `feed-bar`/`condition-strip` now used twice (Lifelines + here)
  → promote to numbered templates per the Harvest rule.
- The **suggested/confirmed epistemic tint** and **honest empty-state** patterns belong in theme/arcade.
- Parsers worth building: CAISO Today's Outlook / EIA-930 condition → header chip; a PowerOutage.us
  GeoJSON adapter as a portable national fusion feed.
- Catalog spine now serves four products (Atlas, COP, Lifelines, Grid Watch) — the role-tagging
  convention belongs in the shared builder.
- **Framework gotchas surfaced while building the native app** (→ strata-core issues; see the
  `strataapp-declarative-widget-gotchas` memory): the declarative `chart` widget crashes (it needs a
  `charts` array — the shipped `monitor` template crashes identically); the standalone `legend` needs
  `layers` injected via context; **`FeatureLayerDataSource` never queries a remote service** (it
  aggregates an empty in-memory `rows`), so live KPIs/tables need custom fetching widgets or an injected
  `queryFn`. These forced the custom-widget path below and are the highest-value fixes to upstream.

## 9. As-built (native `<StrataApp>`, shipped 2026-07-26)

The shipped app is a **native `<StrataApp>`** at `strata/examples/utilities-network-operations/`
(driven by `layers.json` + `app.json`), published to the site as a static Vite bundle at
`/static/apps/utilities_network-operations/` (`base:"./"`), on live California feeds. Because the
declarative registry widgets can't fetch remote services or drive the interactions, **four widget types
are overridden with app-local custom components** (manifest §10.2) — the §10.3 candidates *became* the
real implementation, not their fallbacks:

- **`kpi` → `KpiLive`** — self-fetching aggregate KPIs that **re-scope to the selected territory**
  (spatial `outStatistics` within the territory envelope) and revert to statewide on clear. The five:
  customers-out, active outage areas, PSPS active, peak wind, transmission miles.
- **`stress-rail` → `StressRail`** — the stress rail as **scored cards**: name + colored score badge
  (red ≥66 / amber 40–65 / green <40) + `{wind} mph` · `HFTD` · `{dependents} dep.` chips, ranked by
  `0.45·windN + 0.35·hftdN + 0.20·depN` (per-territory spatial queries: NDFD wind max, CPUC HFTD
  intersect, emPOWER dependent-devices sum).
- **`map` → `MapLive`** — wraps `StrataMap`, captures the imperative map handle, and adds three **on-map
  controls** (top-right, under the zoom buttons): **Home** (default extent), **Layers** (show/hide +
  remove, store-driven), **Basemap** (keyless OSM/CARTO picker with **z5-California tile thumbnails**).
- **`table` → `SimpleTable`** — the live-outage table.

**Signature interaction (built):** click a stress card or outage row → the map **zooms to the feature
extent + pans + flashes + opens a popup**, and (territory) the KPIs re-scope; click the same row again
to **clear**. Clicking a feature on the map opens a popup **titled with the layer name + the full field
table** (`popupInfo.title` injected per layer at load). Theme: control-room dark, amber primary,
semantic status colors; keyless CARTO dark basemap; EPSG:4326; `responsive.small` collapses the rows.

**Designed-but-not-yet-in-the-native-build** (demonstrated in the bespoke standalone `app/index.html`,
or planned for the next increment): the **de-energization lens window** (who-loses-power fan-out), the
**draw/sketch what-if**, the **[Stage PSPS notice] export**, the **feed bar**, and the **CAISO/EIA
condition chip**. The native build ships the condition-strip KPIs, network map, stress rail, outage
table, on-map controls and layer-named popups.

**On-site copy (published):** the detail page uses the shortened AppMeta description at the standard
`.lead` size; the Utilities category is **Available now**; the Network Operations Map card is a live
link. Runs on Strata *or* ArcGIS; nine apps now live in the catalog.

## 8. Sources
- **ESRI:** doc.arcgis.com — Intro to Electric Utility Network Foundation; esri.com — UN design-to-operations.
- **ADMS/EMS/OMS/DERMS:** GE Vernova GridOS (Network Operations · ADMS · DERMS · ADMS-vs-DERMS);
  Schneider EcoStruxure ADMS; Oracle Utilities NMS/ADMS; Survalent (FLISR/Loss-of-Voltage); Siemens
  Spectrum Power; Hitachi Energy Network Manager; AspenTech OSI monarch; IEEE-PES DERMS.
- **Public/open feeds:** PowerOutage.us (+ "use our data"); CAISO Today's Outlook; EIA-930 Hourly
  Electric Grid Monitor; gridstatus.io; CPUC power-outage maps; SCE PSPS Weather Dashboard; PG&E PSPS.
- **Standards/regulators:** ENTSO-E CIM; IEEE 1366 reliability indices; IEEE 2030.5 / CA Rule 21;
  FERC reliability explainer; NERC TOP-003-8; CPUC GO 166 annual reports; CPUC fire-threat maps.
- **Data:** `.private/data_sources/data_sources_ca.md` (crawl 2026-07-16) → `uno-catalog-ca.json`
  (`build_uno_catalog.py`); endpoint verification 2026-07-26 (see `DATA.md`).
- **Internal:** `DESIGN-PROPOSAL.md` · `DATA.md` · `../DESIGN-REQUEST-PROMPT.md` · manifest §10 ·
  `strata/docs/guide/app-design.md`.

---

## Modernization (parity release)
> Native: RestDataSource catalog spine (4th product on it) · timer→refresh ~5-min loop · arcade stress
> + suggested/confirmed epistemic classing · weighted-overlay stress score · pointsWithin who-loses-
> power fan-out · export-from-state PSPS notice & brief · splitter/panel/window shells · honest empty
> state (PSPS off-event) + OSM/ODbL attribution as first-class states · app-shell header/footer/splash ·
> `dataSource.sourceId` linking (condition strip updates with no `connections`).
