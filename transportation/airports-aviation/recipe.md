# Recipe — Airports & Aviation: **The Altimeter** (Mobility & Logistics)

A reproducible path to an **airspace-clearance screening** app on strata-core: pick an airport, drop a
pin, type a proposed height — and read, in one column, **every ceiling that governs that spot, which
one bites first, and by how many feet**. Federal screening (14 CFR §77.9 notice criteria, §77.17
obstruction standards, the Part 77 horizontal and conical surfaces) and the **local California ALUC
referral gate** answered in the same click, on data that is already public. Two supporting pages carry
the rest of the sector's brief — an **airfield asset bench** (airside/landside, FAA AMDB + Caltrans)
and a **ground-access catchment** page.

> **Scope (honest).** This is a **screening pre-check**, not a determination. It computes the §77.9
> notice slopes and §77.17 obstruction standards, and the Part 77 **horizontal** (airport elevation
> +150 ft) and **conical** (20:1 for 4,000 ft) surfaces, from published runway geometry, published
> airport elevation, and a 3DEP ground elevation. It does **not** compute **TERPS** or
> **one-engine-inoperative (OEI)** surfaces — the two that actually cap downtown San José — and it does
> not author approach/transitional surfaces as 3D multipatches (that is ArcGIS Aviation Airports' /
> Transoft SkySAFE's OIS/OLS territory). It never says "approved"; it says *notice required* or *below
> notice criteria on these inputs*, always with the formula, the three inputs, their sources, and a
> link to `oeaaa.faa.gov`. Catchment is **geometric** (isochrone + census), **not** a behavioural
> air-service leakage study. Feature writes (obstruction-case status) need a writable ESRI backend.

Run the §5 prompt-script on a fresh strata-core project (or via `/new-app`). ESRI Web Map JSON is the
map contract; the app is read-only on Strata Serve.

---

## 0. Provenance

| | |
|---|---|
| **Source** | `https://tabaqat.net/solutions` — **Mobility & Logistics** section |
| **Name on site** | Airports & Aviation |
| **Tagline on site** | "Airside and landside assets, obstacles and catchment mapping" |
| **Scaffolded** | 2026-07-22 (`tabbed-workbench` placeholder) |
| **Researched & designed** | 2026-07-27 — see `DESIGN-PROPOSAL.md` |
| **Template** | **`open-design` ("height-ladder")** — the `tabbed-workbench` assignment is released back to the Mobility sector |
| **Catalog** | `aviation-catalog-ca.json` — 2,339 CA services, **245 role-tagged** across 10 roles, **23 external feeds (17 curl-verified live)** |

---

## 0.1 The working app in this folder

`app/` holds a **runnable demo of this design** — no build step, no API keys, everything fetched live
from the §3 services.

```bash
cd app && python3 server.py          # -> http://localhost:8043   (or double-click START-APP.command)
node test-geometry.mjs               # 35 assertions against LIVE FAA/Caltrans/Monterey/USGS data
```

| File | What it is |
|---|---|
| `app/index.html` | **Airports & Aviation** ("The Altimeter"), self-contained: airport picker over the 242 CA public airports, click-to-probe with a **✕ Clear pin** reset, the height ladder, verdict + margin + local ALUC gate, the runway→site section, the DOF neighbours table, Part 77 surfaces + notice-slope bands + LAANC ceilings, **on-map basemap and layer controls**, the catalog drawer over all 2,339 CA services, the printable referral packet, **dark by default** with a light mode, EN |
| `app/test-geometry.mjs` | Regression test that **extracts the shipped functions out of `index.html`** and checks them against live data — projection round-trip, centreline derivation from FAA polygons *and* Caltrans closed outlines, point-to-segment distance, the §77.19 primary-surface extension, the horizontal capsule, point-in-polygon, the `AIRPORT_ID` GUID trap, the worked §77.9/§77.17 screening at SJC, and every service's reachability |
| `app/server.py`, `app/START-APP.command` | Zero-dependency static server (MapLibre cannot run from `file://`) |
| `app/aviation-catalog-ca.json` | Copy of the built catalog the drawer reads |
| `presentation/index.html` | 10-slide deck (house template, prints to PDF one slide per page) |
| `presentation/linkedin-article.md` | ~730-word article + teaser post + a claims note citing every figure |

**Verified 2026-07-27 — all 35 assertions pass, plus a browser pass over the running app:**

| Check | Result |
|---|---|
| `centerlineOf` reproduces the published runway length from FAA polygons | ✅ 10,993 ft derived vs 11,000 ft published (both SJC runways) |
| …and recovers a centreline from Caltrans' closed outline | ✅ 4,380 ft where a naive first/last read gives 0 |
| Point-to-segment distance | ✅ a 3,000 ft normal offset measures 2,999.9 ft; nearest point lands 0.2 ft from the midpoint |
| §77.19 primary-surface extension | ✅ exactly +400 ft |
| Part 77 horizontal capsule | ✅ every vertex within 0.05% of the radius |
| The `AIRPORT_ID` GUID trap | ✅ `='SJC'` returns 0; the `GLOBAL_ID` join returns 12L/30R + 12R/30L |
| Worked screening at KSJC | ✅ 100:1 selected, §77.9(b) = 202.3 ft MSL, verdict NOTICE REQUIRED, margin −13 ft |
| Live services the app boots against | ✅ Caltrans 242 · DOF 49,950 CA · UASFM 370,441 · Monterey ALUCP 48 · 3DEP 45.6 ft (CORS `*`) |
| Signature loop, driven in a real browser | ✅ same pin: 130 ft AGL → **CLEAR +13 ft**; 320 ft AGL → **PRESUMED OBSTRUCTION −177 ft**; binding rung §77.9(b) both times |
| Basemap switch keeps the operational layers | ✅ Dark Matter → Voyager: all layer groups survive, 1,017 obstacles still drawn, runway ink re-paints |
| Layer panel show / hide / remove | ✅ hide sets `visibility:none`; remove also empties the source and strikes the row; restore re-pushes |
| Clear pin | ✅ marker, verdict, ladder, section, table and derived layers reset — the airport's 1,017 obstacles stay loaded |
| Bench + catchment pages | ✅ 217 AMDB features at KSJC (runways first, coded values decoded); 24 competing airports within 60 mi |

**What the demo app deliberately does not do:** it is a MapLibre app that *renders this design*, not a
`<StrataApp>` — §5 is what produces the real `layers.json` + `AppLayout` on strata-core. Catchment
rings are straight-line (drive-time isochrones need a routing key), the ALUC gate resolves only where
a county publishes zone geometry (Monterey today), and no verdict is ever an FAA determination.

## 1. Study — how the market frames this

**The question the buyer asks:** *"How tall can I build here before the FAA and the airport get a
say?"* — and its mirror at the counter: *"Does this permit application need an airport referral?"*

**The regulatory spine (the thing being computed).**
- **14 CFR §77.9** — notice to the FAA (Form **7460-1**, filed at `oeaaa.faa.gov`, **45 days** before
  construction) is required when a structure is **> 200 ft AGL**, *or* exceeds an imaginary surface
  rising **100:1 for 20,000 ft** from the nearest point of the nearest runway **longer than 3,200 ft**
  (**50:1 for 10,000 ft** for runways ≤ 3,200 ft; **25:1 for 5,000 ft** from a heliport).
- **14 CFR §77.17** — obstruction standards: **500 ft AGL** anywhere; **200 ft AGL** (or above the
  established airport elevation) within **3 NM** of an airport whose longest runway exceeds 3,200 ft,
  **rising 100 ft per additional NM to a 500 ft maximum**.
- **Part 77 imaginary surfaces (§77.19)** — the **horizontal** surface, a plane **150 ft above the
  established airport elevation**, bounded by arcs swung from each end of each runway's primary
  surface at **5,000 ft** for runways designated *utility* or *visual* and **10,000 ft** for all
  others; and the **conical** surface rising from that perimeter at **20:1 for 4,000 ft**. Approach
  and transitional surfaces exist but are not authored here (see Scope). ⚠ **Neither open layer
  carries the Part 77 runway designation** — the app must ask for it, defaulting to a documented
  approximation (`IAPEXISTS=1` or `LENGTH > 3,200 ft` ⇒ treat as *other*, i.e. 10,000 ft) and printing
  which branch it used.
- **FAA processing reality** — an aeronautical study is *"at least 60 days"* and proposals are worked
  in the order received. The cost of guessing wrong is a stop-work order on a crane.

**Reference solutions (benchmark + coexist, never copy).**
- **Esri — ArcGIS Aviation Airports** (the incumbent): the **Obstruction Identification Surfaces**
  toolset (multipatch/polygon OIS to FAA and ICAO specs, heliport surfaces, curved approaches, light
  signal clearance, OIS-intersection and vertical analysis against lidar), the airport data model, and
  **ICAO Annex 15 eTOD** support. Sits under Esri's Airports industry pages
  (*Airside & Landside Operations*, *Facilities & Asset Management*) alongside **ArcGIS Dashboards**
  and **ArcGIS Indoors**; named customers include Reno-Tahoe (FAA Airport GIS compliance), San
  Bernardino, Amarillo and Miami. This is a licensed desktop/enterprise product run by the airport's
  consultant on a master-plan cycle — **not** a public counter-facing screening map.
- **Transoft Solutions — AviPLAN + SkySAFE**: the non-Esri obstacle-limitation engine (airside
  planning, aircraft-manoeuvre simulation, obstacle clearance compliance and safeguarding), and the
  most visible vendor preparing airports for **ICAO Annex 14 Amendment 18** (the OLS → **OFS/OES**
  rework: adopted 2025, effective July 2025, **compliance 21 November 2030**).
- **Consultancy/survey side:** **NV5 Geospatial** (airspace analysis for vertically and
  non-vertically guided runways, ALP and FAA-compliance mapping), **Capitol Airspace Group**
  (obstruction-evaluation practice), Woolpert/Hanson-class AGIS survey firms.
- **Digital-twin entrants** (adjacent, not the same job): Siemens, SITA, Thales, Dassault, **Bentley**,
  **Hexagon**, **Veovo** (which has added a geospatial digital twin to its Intelligent Airport
  Platform), Honeywell; airside/tower surveillance from Indra, **ADB SAFEGATE**, Saab.
- **The public-facing analogues that prove the demand** — and that are each a single airport, hand-
  built: **Massport's Logan Airspace Map**, **San José Mineta's Downtown Height Limits** programme
  (crane-notification form + a developer fee that credits airline landing fees to offset OEI weight
  penalties), **RDU Height Zoning**, **Dane County's Airport Height Limitation Zone**, and the UK
  aerodrome-safeguarding permits at **Edinburgh** (OLS/IFP assessment; crane consultation within
  6 km above 10 m) and **Newcastle** (tall-equipment permit, 28 working days).
- **Catchment side:** the **ACI-NA** methodology series (2025–2026) and **ACRP**'s *Defining an Air
  Service Catchment Area* tool — both now explicit that radius/drive-time rings misstate demand and
  that mobile-location + ticketing (ZIP-OD) data is the standard; vendors include Flight BI/Fligence,
  Placer.ai-class mobility providers, and Aero Data Services.

**The gap we occupy.** **ACRP Research Report 195 — *Best Practices for Airport Obstruction
Management Guidebook*** tells airports to stand up a continuous obstruction-management programme and,
centrally, to **share the surfaces with surrounding land-use jurisdictions** — noting that surfaces
can be transferred to local government GIS or published on the internet, custom-built per airport.
That sharing step is where practice collapses: the surfaces sit in a consultant's geodatabase, the
permits are issued at a city counter, and the airport learns about the crane when it is already up.
Meanwhile the three numbers the screening needs are **all openly published now** — FAA
`US_Airport.ELEVATION`, FAA `Runways` geometry + `LENGTH`, and USGS 3DEP over a **keyless, CORS-open**
point API. **Nobody has put the arithmetic on an open map with the local referral answer beside it.**

**California makes it statutory.** Since 1967 (**PUC §21670**) each county with a public-use airport
must have an **Airport Land Use Commission** publishing an **Airport Land Use Compatibility Plan** on
a 20-year horizon, updated every 5–10 years, covering **noise, safety (both ground risk and airspace
protection consistent with Part 77 and PUC §§21658-21659), and overflight**, with development
typically within two miles referred to the ALUC; a local agency can override only by a two-thirds vote
after 45 days' notice to Caltrans' **Division of Aeronautics**, which publishes the **California
Airport Land Use Planning Handbook**. Santa Clara County's ALUC requires referral exhibits showing
**Airport Influence Area, Part 77 contours, CNEL contours, safety zones and flight paths**. But the
zone geometry is published **county by county, if at all** — Monterey publishes 48 polygons for 3
airports with the compatibility rule in an attribute; LA and San Diego publish via portals; most
publish a PDF. That fragmentation *is* the market.

**Our edge.** AI-authored, MIT/open, runs on **Strata *or* ArcGIS**, sovereign/on-prem, and — the
decisive one — **counter-facing**: the incumbents sell the hard 5% (OIS/OLS production) to the
airport's consultant; nobody sells the easy 95% (the screening answer) to the thousands of people per
year who actually need it. Coexistence, never "replace ArcGIS": an airport that owns ArcGIS Aviation
Airports should publish *its* authoritative surfaces into this app.

**Standards, specifications & organizations to speak fluently.**
- **FAA:** 14 CFR **Part 77** (§77.9 notice, §77.17 obstruction standards, imaginary surfaces),
  **Form 7460-1** and the **OE/AAA** system + Notice Criteria Tool (`oeaaa.faa.gov`), **ADIP** (the
  Airport Data and Information Portal — formerly the Airport GIS application, carrying **AGIS** and
  the **5010 Airport Master Record**), **eALP**, AC **150/5300-16/-17/-18** (geodetic control, imagery,
  and general survey standards for airport data), **Part 139** certification, **NASR** (28-day AIRAC
  CSV subscription), the **Digital Obstacle File**, **UAS Facility Maps + LAANC**.
- **ICAO:** **Annex 14** (aerodrome design; **Amendment 18** replaces classic OLS with **OFS**
  obstacle-free and **OES** obstacle-evaluation surfaces — compliance 21 Nov 2030), **Annex 15 / PANS-AIM**
  (**eTOD** Areas 1–4), **AIXM 5.1**, RTCA **DO-272** (AMDB) / EUROCAE ED-99.
- **California:** **PUC §§21670-21679.5** (ALUC/ALUCP), PUC §§21658-21659 (airspace protection),
  **Caltrans Division of Aeronautics** + the **California Airport Land Use Planning Handbook**, CNEL
  noise metric, CEQA (ALUCP updates are CEQA projects).
- **Bodies:** FAA (ARP / AAS-100 Airports Data & Airspace Branch), ICAO, EUROCONTROL (TOD Manual),
  Caltrans Aeronautics, county ALUCs, **ACRP/TRB**, **ACI-NA**, AAAE.

**Honest scope — what this is not.** Not a system of record (ADIP/AGIS is). Not an FAA determination
(OE/AAA is). Not TERPS or OEI analysis. Not an eTOD or AIXM production line. Not authoritative
approach/transitional surface geometry. Not a behavioural catchment study. Not applicable over
military fields (UFC 3-260-01 governs, not Part 77) — the app refuses a verdict there and says why.

## 2. UI design spec (front-loaded)

### 2.1 Layout (Template: **`open-design` — "height-ladder"**)

- **Template `open-design`** under the freestyle charter (`strata/recipes/COMPONENT-MANIFEST.md` §10):
  registry widgets and manifest config keys only, plus three §10.2 new widgets **each with a named
  day-1 fallback** (§2.6). Do **not** fall back to `split-dashboard`.
- **Why this silhouette:** the buyer's question *is* a height, so the answer is read on a **vertical
  feet-MSL axis**, not inferred from a colour on a plan view. Anti-collision (Mobility & Logistics):
  not `triage-console` (road/highway assets), not `launchpad` (rail transit), not `ops-command`
  (ports/maritime), not `nearby-finder` (logistics distribution), not `time-player` (incidents/flow).
  No sibling — in this sector or the library — uses a vertical axis as its navigation.
- **Signature loop:** *pin + height → three verified numbers (airport elevation · distance to nearest
  runway · site ground elevation) → the ladder recomputes → the binding rung names itself → the map
  shows which runway and which surface → the referral packet writes itself.*
- **Wiring floor:** ≥3 live `connections` on first render — this design ships **18** (§2.7), and five
  widgets link with **no connection at all** via `dataSource.fromWidget:"probe"`.

```
┌ HEADER ✈ AIRPORTS & AVIATION · airport ▾ KSJC · AIRAC 2026-07 · ◐ · [Referral packet] ───────────┐
│ HEIGHT LADDER (348px)          │            MAP (full-bleed, dark)                               │
│  site 37.38886,-121.89130 ✕clr │   ○ probe pin ——— 14,000 ft (2.30 NM) to RWY 12L/30R ———▶       │
│  ┌──────────────────────────┐  │   ░ Part 77 horizontal surface (airport elev +150 ft)           │
│  │ 585 ─ §77.17(a)(1) 500AGL│  │   ░ conical 20:1 · 4,000 ft                                     │
│  │ 412 ─ §77.19(b) conical  │  │   ▲ DOF obstacles within 1 NM                                   │
│  │ 285 ─ §77.9(a) 200 ft AGL│  │   ▤ ALUCP safety zone under the pin                             │
│  │     · §77.17(a)(2) @2.3NM│  │   ▨ LAANC cell — 200 ft (advisory)                              │
│  │ 215 ▓▓ YOUR STRUCTURE    │  │   ⃠ OEI / TERPS — not computed (labelled placeholder)           │
│  │ 202 ━━ §77.9(b) 100:1  ◀ binding                                              ▤ basemap       │
│  │  85 ─ site ground (3DEP) │  │                                                 ◈ layers        │
│  └──────────────────────────┘  │                                                 + / −           │
│  VERDICT  ⚠ NOTICE REQUIRED    │                                                                 │
│  binding: §77.9(b) 100:1 slope │                                                                 │
│  margin:  −13 ft    [file 7460-1 ↗]                                                              │
│  local:   ⬤ ALUC referral required — Monterey Co. zone 4                                        │
├────────────────────────────────┴─────────────────────────────────────────────────────────────────┤
│ SECTION  RWY 12L/30R ──────────────────── 14,000 ft ──────────────▶ site  (terrain + slope)       │
├──────────────────────────────────────────────────────────────────────────────────────────────────┤
│ NEIGHBOURS ▲ T-L TWR 954 AGL · study 2016AWP08468OE ↗ │ ▲ BLDG 210 AGL │ … row → zoom + flash    │
└──────────────────────────────────────────────────────────────────────────────────────────────────┘
```
*Worked example above, arithmetic verified in `app/test-geometry.mjs`:* KSJC, airport elevation
**62.3 ft MSL**, longest runway **11,000 ft** ⇒ **100:1 for 20,000 ft**. A site **14,000 ft** out,
ground **85 ft**, proposed **130 ft AGL** ⇒ top **215 ft MSL**. §77.9(b) allows `62.3 + 14,000/100 =
**202.3**`; §77.9(a) allows 285; §77.17(a)(2) allows 285; conical allows 412. Lowest is §77.9(b) ⇒
**NOTICE REQUIRED, binding §77.9(b), margin −13 ft.**
```
```
*Phone (`responsive.small`):* the ladder **is** the page (verdict chip pinned top, rungs scroll); the
map and section collapse into an `accordion`; the packet button stays visible. Every side-by-side
`row`/`splitter` in the tree carries a `responsive.small` collapse to `column`.

**Pages** (behind a `page-nav` in the header — but **page 1 alone answers the purpose sentence**):

| Page | Type | What it is |
|---|---|---|
| `clearance` | fixed | **The Altimeter** — the product |
| `bench` | fixed | **Airfield bench** — a `views` node (`nav:"tabs"`, `mapState` per tab): *Airside* · *Landside* · *Obstruction register* · *Airspace* |
| `catchment` | fixed | **Ground-access catchment** — isochrone/ring bands, competing airports, population per band (labelled geometric) |

`splash` (`once:true`) carries the scope disclaimer. Page `header` holds title + airport picker +
AIRAC cycle + packet button + `theme-switch`; `footer` holds attribution and the "screening only —
not an FAA determination" line.

**Map-anchored controls (bottom-right stack).** Three controls share one style and stack above the
native zoom cluster: **basemap** (a globe glyph → the keyless gallery), **layers** (a stack-of-planes
glyph → show / hide / **remove** per operational layer), then **+/−**. Glyphs follow the ESRI/Calcite
idiom and are inline SVG on `currentColor`, so they theme with the app. ⚠ MapLibre **prepends**
controls in a bottom corner, so the add order is inverted: add navigation *first* for the two buttons
to sit *above* it.

### 2.2 Theme

```jsonc
{ "mode": "dark",
  "colors": { "primary": "#1f4e9c",   // sectional-chart controlled-airspace blue
              "secondary": "#b5197f", // sectional magenta
              "success": "#1f7a4d",   // clear of all criteria
              "warning": "#c2711b",   // notice required (§77.9)
              "danger":  "#b3261e",   // presumed obstruction (§77.17)
              "info": "#4a6f8a", "light": "#f4f6f9", "dark": "#101a26" },
  "fonts": { "scale": "compact" },
  "variables": { "--strata-radius-md": "4px" } }
```
An **altimeter read off a sectional chart**: the app **launches dark** — the ladder, the verdict chip
and the aeronautical linework are the only saturated objects on screen, and the numbers carry in a
room. `theme-switch` flips to a light, chart-paper mode for public-counter and print use, and pairs
the basemap to it. Polygon fills ~40/255 so horizontal + conical + ALUCP + LAANC stack readably.

**Basemap gallery — keyless, OSM-derived only, user-switchable** (all five curl-verified
`200 image/png`): **CARTO Dark Matter** (default) · **CARTO Positron** · **CARTO Voyager** ·
**OpenStreetMap** · **OpenTopoMap**. The theme button pairs the basemap to the mode; the gallery can
then override that pairing independently. ⚠ **MapLibre paint cannot read CSS variables**, so the few
ink-coloured layers (runway centrelines, AMDB runway fills) must be **re-painted whenever the basemap
flips light↔dark** — otherwise the centrelines vanish into a dark basemap.

**Language: EN only.** `@strata/i18n` and `lang-switch` are deliberately **not used** here — the
screening output is a regulatory record quoting 14 CFR section numbers and an FAA study number, and a
translated verdict invites a dispute about which language governs. An Annex 14 / GACA variant is a
localisation project with its own regulatory copy, not a switch on this app.

### 2.3 KPI cards

Four, live via `stat` + a shared source — **no `connections` needed**:

| Card | Value | Threshold colouring |
|---|---|---|
| **Verdict** (`kpi`) | `CLEAR` / `NOTICE REQUIRED` / `PRESUMED OBSTRUCTION` | success / warning / danger |
| **Margin to binding rung** (`kpi`) | ± feet, with the rung named as `deltaLabel` | sign-coloured |
| **Height consumed** (`gauge`) | % of allowable height used (0–100, thresholds 70/95) | `invertColors` off |
| **Obstacles in view** (`kpi`, `stat:{field:"OBJECTID",op:"count"}`) | live count from `faa-dof`, recomputed on `extentChange` | neutral |

Plus a **local-gate chip** (`kpi`, `status`): *ALUC referral required* / *no published ALUCP geometry
for this county — referral status unknown* (never a green tick on missing data).

### 2.4 Charts & table

- **`typechart` (`chart`, histogram/bar)** — obstacles by `TRIM(Type_Code)` within the probe radius;
  click → `categorySelect` cross-filters map + table.
- **`heightchart` (`chart`, histogram)** — `AGL` distribution of nearby obstacles, so "is 208 ft
  unusual here?" is answerable at a glance.
- **`neighbours` (`table`, server-paged `AttributeTablePanel`)** — real columns:
  `Type_Code` · `AGL` · `AMSL` · `Verified` · `Lighting` · `Marking` · `Study` · `Date`.
  Row → `zoomTo` + `flash`; `Study` rendered as a link to `oeaaa.faa.gov`; CSV/GeoJSON export.
- **Bench:** `assettable` on the AMDB layers (`FAA_ID` · `DESIGNATOR` · `SURFACE`) and `carto`
  cross-filter widgets (category on `SURFACE`, histogram on `AGL`).

### 2.5 Capabilities to use (Phases 0–7)

- **`/analyze`** — `buffer` (notice-slope contour bands at 2,000-ft increments to 20,000 ft, each band
  carrying `max_msl`; also the 5,000/10,000 ft horizontal-surface radius and the 4,000 ft conical
  ring), `pointsWithin` (obstacles inside 1 NM of the probe), `aggregate` (population per catchment
  band). **No `hexbin`/`hotspot`** — clustering 49,950 obstacles just reproduces the population map.
- **WIF `connections`** — §2.7; plus `dataSource.fromWidget:"probe"` linking five widgets with none.
- **Panels/widgets** — `layer-panel` and `basemap` as **map-anchored controls** (§2.1: bottom-right,
  above the zoom cluster; the layer control carries show / hide / **remove** per layer, and is the
  single source of truth for visibility — the rung-isolate action writes to the same state, so no
  control can ever claim a layer is off while it is drawn), plus `legend`, `feature-info`,
  `data-actions`, `filter`, `query`, `analysis`, `near-me`, `add-data` (bench catalog drawer over the
  2,339-service library), `draw`/`measure` (both revert to `identify`), `bookmarks`, `share`,
  `status-bar`, `elevation`. A **✕ Clear pin** control resets the probe and every derived widget
  while leaving the airport's obstacle context loaded.
- **Plugins** — `@strata/plugin-search` (address → probe site, Nominatim), `@strata/plugin-routing`
  (catchment isochrones, OSRM/Valhalla), `@strata/plugin-statusbar`. **Not** `plugin-timeslider`:
  aviation data is AIRAC 28-day, so the cycle is a *label*, not an animation.
- **Composed export** — the **referral packet** (`/export report`: legend + scalebar + north arrow +
  the rung table with formulas and sources + the section profile + the ALUCP `Definition` and
  `ALUCP_link`), an **atlas** (one page per site for batch screening), an image, and a `share`
  deep-link with `setUrlParam` on scope.
- **Modernization (§8 patterns)** — structured `theme`, app-shell (`header`/`footer`/`splash`),
  `splitter`/`panel`/`window`/`views`+`mapState`, `RestDataSource` (catalog JSON + EPQS),
  `FileDataSource` (upload a site list for batch screening), `kpi`/`gauge` `stat` linking, `animate`
  (`fly` + `stagger` on the rungs — they read as an instrument settling; never on the map container).
- **Arcade** — renderer classing on `TRIM(Type_Code)`, AMDB coded-value decoding, and the verdict
  colour expression.
- **`@strata/i18n` / `lang-switch` — deliberately not used.** See §2.2: the output is a regulatory
  record quoting US CFR section numbers; a localised variant is its own project.
- **Writes 🔶** — obstruction-case status (`updateRecord`) only behind `assertEsriBackend`; the whole
  screening path is read-only and works on Strata Serve.

### 2.6 The three new widgets (§10.2/§10.3) and their day-1 fallbacks

| Registry key | Purpose | Emits | Day-1 fallback (ships without any core change) |
|---|---|---|---|
| **`airspace-probe`** | site + height + 3DEP ground elevation → §77.9/§77.17 screening result | `featureSelect`, `recordsChange` | **precomputed notice-slope contour bands** (`/analyze buffer`, 2,000-ft increments, each band carrying `max_msl`) + `near-me` to drop the pin + `feature-info` to report the band — point-in-polygon instead of live math, labelled *"banded to 2,000 ft — file for the exact figure"* |
| **`height-ladder`** | the rung stack on one feet-MSL axis, binding rung highlighted, rule citation per rung | `categorySelect` | vertical **`stacked-bar`** (`horizontal:false`, one series per rung) + **`gauge`** (% allowable height consumed) + **`kpi`** naming the binding rung |
| **`section-profile`** | runway threshold → site section: terrain, slope, obstacles, structure | — | the shipped **`elevation`** widget fed with 3DEP samples along the line + a `sparkline` for the slope + the formula printed as `text` |

All three obey the §10.2 contract: app-local `registry` override, `--strata-*` tokens only, logical
CSS properties throughout, map driven **through the store**, data via `dataSource`, cross-widget
behaviour declared in `connections` — never hard-wired to a sibling.

### 2.7 `connections` (18 — every wire uses a shipped emitter)

`buttonClick`/`timer`/`mapClick`/`sketchComplete` **emitters are Phase-2 pending**, so nothing
load-bearing depends on them; the new widgets emit only shipped trigger types.

| from | trigger | to | action | options |
|---|---|---|---|---|
| `probe` | `featureSelect` | `map` | `zoomTo` | — |
| `probe` | `featureSelect` | `packet` | `showHide` | `{ "hidden": false }` |
| `ladder` | `categorySelect` | `map` | `filter` | `{ "layerId": "p77-surfaces" }` |
| `ladder` | `categorySelect` | `zonecard` | `viewInTable` | — |
| `neighbours` | `rowSelect` | `map` | `zoomTo` | `{ "layerId": "faa-dof" }` |
| `neighbours` | `rowSelect` | `map` | `flash` | `{ "layerId": "faa-dof" }` |
| `neighbours` | `rowSelect` | `zonecard` | `viewInTable` | — |
| `map` | `featureSelect` | `zonecard` | `viewInTable` | — |
| `map` | `extentChange` | `kpi-obstacles` | `showStatistics` | — |
| `zonefilter` | `filterChange` | `map` | `filter` | — |
| `zonefilter` | `filterChange` | — | `setUrlParam` | `{ "param": "scope" }` |
| `typechart` | `categorySelect` | `map` | `filter` | `{ "layerId": "faa-dof" }` |
| `typechart` | `categorySelect` | `neighbours` | `filter` | — |
| `heightchart` | `rangeSelect` | `neighbours` | `filter` | — |
| `assettable` | `rowSelect` | `benchmap` | `zoomTo` | `{ "layerId": "amdb-runway" }` |
| `assettable` | `rowSelect` | `benchmap` | `flash` | `{ "layerId": "amdb-runway" }` |
| `assetchart` | `categorySelect` | `benchmap` | `filter` | — |
| `catchchart` | `categorySelect` | `catchmap` | `filter` | — |

**Zero-connection links** (`dataSource.fromWidget` / `sourceId`): `ladder`, `verdict`, `margin`,
`section`, `neighbours` all bind to `probe`; `catchtable` + `catchkpi` bind to the `analysis` widget
`bands`.

## 3. Data sources

All EPSG:4326 (reproject on ingest); `OBJECTID` is the OID throughout. **Maryland has no aviation
layers in `data_sources_md.md`** (verified) — the national FAA services below cover it, so the matrix
is California × National rather than the usual three columns.

| Role | California | National / federal | CORS · licence · gotcha |
|---|---|---|---|
| **Airport datum** (elevation, ident, approach type) | Caltrans `CHaviation/Public_Airport/FeatureServer/0` (242) · `Military_Airport` | **FAA `US_Airport/FeatureServer/0`** (874 in CA) | Both CORS-open, public domain. FAA `ELEVATION` = ft MSL, the Part 77 datum. `IAPEXISTS=1` (157 CA) picks the instrument-approach surface set |
| **Runways** (slope selector + geometry) | Caltrans `CHaviation/Airport_Runways` (852 centrelines, statewide) | **FAA `Runways/FeatureServer/0`** (polygons + `LENGTH`) | ⚠ **`Runways.AIRPORT_ID` is a GUID joining `US_Airport.GLOBAL_ID`, not the ident** — `where=AIRPORT_ID='SJC'` returns zero rows |
| **Obstacles** | — | **FAA `Digital_Obstacle_File/FeatureServer/0`** — **49,950 CA** / 641,207 national | ⚠ **`Type_Code` is space-padded to 18 chars**; use `TRIM()`. `Study` = the 7460 study number → deep-link `oeaaa.faa.gov` |
| **Site ground elevation** | — | **USGS 3DEP EPQS** `epqs.nationalmap.gov/v1/json` | **CORS `*`, keyless**, 1 m resolution, `units=Feet`. POINT service — one call per probe. Always allow a surveyed override |
| **Airside assets (AMDB)** | FAA AMDB covers **LAX SFO SAN SJC LGB MRY SBA SMX STS TNP** | **FAA `AM_Runway` / `AM_Taxiway` / `AM_Apron` / `AM_Building` / `AM_Hotspot`** (159 airports) | Partial coverage by design — statewide fallback is Caltrans `Airport_Boundaries` (221 polys for 242 airports) + `Airport_Runways`. Here `FAA_ID` **is** the plain ident |
| **Local land-use gate (ALUCP)** | **Monterey `Planning/Airport_Safety_Zones/FeatureServer/0`** (48 polys, 3 airports; `zone`,`Definition`,`ALUCP_link`) · LA County *Airport Influence Area* + LA City *RPZ/Inner Safety Zone* · San Diego SDCRAA ALUCP | — (no federal equivalent — this is state law) | ⚠ `zone` contains **nulls**. LA service URL must be resolved from the portal item at build; **SD `webmaps.sandiego.gov` timed out 2026-07-27** — re-test or ingest the downloadable boundaries via `/convert` |
| **Airspace** | — | **FAA `Class_Airspace`**, **`Special_Use_Airspace`** | `LOWER_VAL`/`UPPER_VAL` carry their own `_UOM` — read it before comparing to a height. `TIMESOFUSE` is free text, never a time filter |
| **Drone ceilings** | — | **FAA `FAA_UAS_FacilityMap_Data_V5`** (370,441 cells) | `CEILING` is **advisory**, not an authorisation — the UI must say so. `maxRecordCount` **1000**; always query by extent. Many near-identical siblings — V5 is current |
| **Aviation weather stations** | Caltrans `CHaviation/AWOS` | — | **Inventory only, no observations** — do not present as live weather |
| **Heliports** | Caltrans `CHaviation/Hospital_Heliports` (170) | FAA `ADHP` | `AGL_FT` = pad height (many rooftop). Heliports take the **25:1 / 5,000 ft** notice slope |
| **Catchment base** | Census TIGERweb + ACS; CA county/place boundaries | Census ACS `api.census.gov` | ACS wants a key at volume. Geometric method only (see Scope) |
| **Ground access** | Caltrans `CHhighway/SHN_Lines`, `All_Roads`, `Rest_Areas` | OSM / OSRM | Isochrones via `@strata/plugin-routing` |
| **Aerial-ops context** (CA-specific) | CAL FIRE `Helicopter_Water_Drops_(CAL_FIRE_AIRCRAFT_ONLY)` (**39,164 events, live**) · `CAL_FIRE_Aircraft_Tracking_public_view` (**3 rows — thin**) · Cal OES `FIRIS_Flight_Paths` (**3 lines, 2 attributes — unusable**) | — | Only the water-drops feed is live **and** dense; ship the other two as labelled, default-off toggles, never as a live-aircraft claim |
| **Bulk / offline** | — | **FAA NASR 28-day subscription** (CSV, AIRAC cycle) | Download-only; use `/convert` for on-prem builds. This is the refresh cadence behind every FeatureServer above |
| **The catalog itself** | `aviation-catalog-ca.json` — 2,339 services, **245 role-tagged** (10 roles), 23 external feeds | — | Built by `build_aviation_catalog.py`; bound as a `RestDataSource` behind the bench's `add-data` drawer |

Attribution: FAA Aeronautical Information Services (public domain), USGS 3DEP, Caltrans Division of
Aeronautics, Monterey County, CAL FIRE / Cal OES — each with "no warranty; not for navigation."

## 4. Verify each URL first (terminal)

Every command below was run on **2026-07-27** and its output is quoted in §3's gotcha column.

```bash
FAA=https://services6.arcgis.com/ssFJjBXIUyZDrSYZ/ArcGIS/rest/services

# 1. Airport datum — ELEVATION is the Part 77 datum; IAPEXISTS picks the surface set.
curl -s "$FAA/US_Airport/FeatureServer/0/query?where=IDENT='SJC'&outFields=GLOBAL_ID,IDENT,ICAO_ID,ELEVATION,IAPEXISTS,STATE&returnGeometry=false&f=json"
#  -> ELEVATION 62.3 ft MSL · KSJC · IAPEXISTS 1 · GLOBAL_ID ACA1CDD6-3606-47F8-9A7B-904DB1B3DBC5

# 2. THE TRAP: Runways.AIRPORT_ID is a GUID, not the ident. This returns NOTHING:
curl -s "$FAA/Runways/FeatureServer/0/query?where=AIRPORT_ID='SJC'&returnCountOnly=true&f=json"   # -> 0
#    Resolve the GUID first, then:
curl -s "$FAA/Runways/FeatureServer/0/query?where=AIRPORT_ID='ACA1CDD6-3606-47F8-9A7B-904DB1B3DBC5'&outFields=DESIGNATOR,LENGTH,WIDTH,COMP_CODE&returnGeometry=false&f=json"
#  -> 12L/30R and 12R/30L, both LENGTH 11000 x WIDTH 150 ft  => >3,200 ft => 100:1 for 20,000 ft

# 3. Obstacles — note the SPACE-PADDED Type_Code and the 7460 study number.
curl -s "$FAA/Digital_Obstacle_File/FeatureServer/0/query?where=State='CA'&returnCountOnly=true&f=json"   # -> 49950
curl -s "$FAA/Digital_Obstacle_File/FeatureServer/0/query?where=State='CA'&outFields=Type_Code,AGL,AMSL,Verified,Lighting,Study&returnGeometry=false&resultRecordCount=5&f=json"
#  -> 'TOWER             ' (18 chars!) · AGL/AMSL in ft · Study 2016AWP08468OE · Verified U|O

# 3b. THE SECOND TRAP (found while building the app): runway geometry is not a centreline.
#     FAA Runways are POLYGONS. Caltrans Airport_Runways are polylines — but at SJC they are
#     CLOSED RECTANGLE OUTLINES (first vertex == last), so a naive first/last read gives ZERO length.
curl -s "https://caltrans-gis.dot.ca.gov/arcgis/rest/services/CHaviation/Airport_Runways/FeatureServer/0/query?where=AP_ID='SJC'&outFields=AP_ID,NAME&outSR=4326&f=geojson"
#  -> 5 vertices, coords[0] == coords[4]; edge lengths 4383 / 132 / 4358 / 141 ft = a rectangle outline
#     Derive the centreline instead: drop the closing vertex, take the longest edge as the axis,
#     project every vertex onto it through the centroid, keep the two extremes.
#     On the FAA SJC polygons that reproduces the published LENGTH to within 7 ft (10,993 vs 11,000).
#     Caltrans' own vintage disagrees with the FAA there (4,380 / 9,108 ft derived vs 11,000 published),
#     so use FAA Runways as PRIMARY (geometry + LENGTH on the same feature) and Caltrans as fallback.

# 4. Site ground elevation — keyless, CORS `*`.
curl -sD- "https://epqs.nationalmap.gov/v1/json?x=-121.9289&y=37.3626&units=Feet&wkid=4326" | grep -i access-control
#  -> Access-Control-Allow-Origin: *      value 45.596 ft

# 5. AMDB coverage is PARTIAL — confirm before promising an airside bench.
curl -s "$FAA/AM_Runway/FeatureServer/0/query?where=1=1&outFields=FAA_ID&returnDistinctValues=true&returnGeometry=false&f=json"
#  -> 159 airports; CA = LAX SFO SAN SJC LGB MRY SBA SMX STS TNP

# 5b. THE THIRD TRAP: AMDB hides its identifiers and uses CODED-VALUE domains.
curl -s "$FAA/AM_Runway/FeatureServer/0?f=json"                       # read the domains
curl -s "$FAA/AM_Runway/FeatureServer/0/query?where=FAA_ID='SJC'&outFields=*&returnGeometry=false&f=json"
#  -> DESIGNATOR is NULL for runways; the identifier is in RWY_ID ('12R/30L').
#     SURFACE  1=Hard/Paved 2=Metal 5=Other than hard
#     RWY_OPER 2=Open 7=Closed 1=Closed indefinitely 3=Under construction 4=Repurposed as twy 5=Unknown
#     Never print the raw code. One airport returns ~217 AMDB features and BUILDINGS OUTNUMBER
#     everything — sort by kind or the two runways are buried.

# 6. California registry + the ALUC gate.
curl -s "https://caltrans-gis.dot.ca.gov/arcgis/rest/services/CHaviation/Public_Airport/FeatureServer/0?f=json"
#  -> FACILITY, AIRPORTID, STATECLASS, FNCTNLCLSS, MANAGER, MNGREMAIL, F5010URL, LATDD, LONGDD  (242 rows)
curl -s "https://maps.co.monterey.ca.us/server/rest/services/Planning/Airport_Safety_Zones/FeatureServer/0/query?where=1=1&outFields=zone,Airport,Definition,ALUCP_link&returnGeometry=false&f=json"
#  -> 48 polys · Monterey Regional 25 / Marina 17 / Salinas 6 · zone HAS NULLS

# 7. Drone ceilings — advisory only, 1000-record page limit, query by extent.
curl -s "$FAA/FAA_UAS_FacilityMap_Data_V5/FeatureServer/0/query?where=1=1&returnCountOnly=true&f=json"   # -> 370441
```

**The calculation, from these fields only** (print it in the UI beside every verdict):

```
site_top_msl   = ground_elev_ft (3DEP or surveyed) + proposed_height_ft_agl
slope          = 100 if longest_runway_LENGTH > 3200 else 50      # heliport: 25
reach_ft       = 20000 if slope == 100 else 10000                 # heliport: 5000
notice_msl(d)  = US_Airport.ELEVATION + d / slope                 # d = ft to nearest runway point
notice_required = proposed_height_ft_agl > 200
                  OR (d <= reach_ft AND site_top_msl > notice_msl(d))          # 14 CFR 77.9
# 77.17: the 200 ft is measured above AGL *or* above the established airport elevation,
# whichever is higher — take the max of the two datums, then add 100 ft per NM beyond 3 NM, cap +500.
datum_msl      = max(ground_elev_ft, US_Airport.ELEVATION)
allow_ft       = min(500, 200 + 100 * max(0, d_nm - 3))
obstruction    = proposed_height_ft_agl > 500                       # 77.17(a)(1)
                  OR (d_nm <= 6 AND site_top_msl > datum_msl + allow_ft)   # 77.17(a)(2)
horizontal_msl = US_Airport.ELEVATION + 150                     # 77.19(a)
radius         = 5000 if runway_designation in ("utility","visual") else 10000   # ASK — not in the data
conical_msl(d) = horizontal_msl + (d - radius) / 20             # 77.19(b): 20:1 for 4,000 ft beyond
```

## Guided wizard — **the prompts that assign the app's defaults**

Claude runs this like an ArcGIS Instant-App wizard: ask each group, **apply the default so "accept
all" builds a complete app**, confirm a one-line summary, then run §5. Launch with
`/recipe transportation_airports-aviation`. Every answer *sets an application default* baked into
`layers.json` / the `AppLayout`. Phrasing per `strata/docs/reference/human-language.md`.

| # | Wizard question | Options → **default** | The default it assigns |
|---|---|---|---|
| 1 · App | Title & AIRAC cycle? | free text → **"Airports & Aviation"**, current 28-day cycle | header title + `strata:notes.asOf` + footer cycle label |
| 2 · Airport | Which airport is the datum? | any FAA ident → **`SJC` (Norman Y Mineta San Jose Intl)** | initial extent, the airport/runway sources, and whether the AMDB airside bench is available |
| 3 · Region | Statewide or one field? | **one field + 20,000 ft study ring** · county · statewide CA | which services load and how much DOF is fetched (statewide = 49,950 obstacles — page it) |
| 4 · Rules | Which rungs on the ladder? `[multi]` | §77.9 notice · §77.17 obstruction · horizontal+conical · ALUCP zone · LAANC ceiling · tallest neighbour → **all six** | the `probe.rules` array + the ladder's rung order |
| 5 · Ground elevation | How is site elevation obtained? | **USGS 3DEP auto (with manual override)** · always manual · uploaded survey | the `elevationUrl` binding + what the packet records as the source |
| 6 · Local gate | Include the ALUC referral answer? | **yes — county ALUCP if published, "unknown" if not** · no | the `zonecard` + local-gate chip; never a green tick on missing geometry |
| 7 · Bench | Airside bench source? | **FAA AMDB where available, Caltrans statewide fallback** · Caltrans only · my own ALP layers | the bench `views` tabs and their `mapState` |
| 8 · Catchment | Include the catchment page? | **yes — 15/30/45-min isochrone bands (geometric)** · ring buffers · skip | the `analysis` widget config + the honest method label |
| 9 · Output | Referral packet + theme | **referral-packet PDF + batch atlas**; **dark (default)** · light chart-paper | wires `/export report` + atlas; `ThemeSpec.mode` + the `theme-switch` / basemap pairing |

**Then:** Claude echoes *"KSJC · one field + 20,000 ft ring · all six rungs · 3DEP auto · ALUC gate on
· AMDB bench · 15/30/45-min catchment · referral packet + atlas · dark"* and, on confirmation, runs
§5 — so the app opens **fully configured**.

## 5. Prompt-script (run in order)

```
A. /new-app — an "Airports & Aviation" open-design app ("height-ladder"): three pages
   (clearance / bench / catchment), structured DARK theme (primary #1f4e9c, secondary #b5197f,
   success/warning/danger per §2.2), app-shell header (title + airport picker + AIRAC cycle +
   [Referral packet] + theme switch), footer attribution + the "screening only, not an FAA
   determination" line, and a splash carrying the scope disclaimer (once:true). EN only — see §2.2.
   Install deps + run command.

B. /add-data the verified layers from §3 for the chosen airport: FAA US_Airport (filter STATE='CA'),
   FAA Runways (resolve the GUID from US_Airport.GLOBAL_ID first — see §4 trap 2), FAA
   Digital_Obstacle_File (State='CA', scoped to the 20,000 ft study ring), FAA Class_Airspace +
   Special_Use_Airspace, FAA UASFM V5 (query by extent, page 1000), the FAA AMDB layers if the airport
   has coverage else Caltrans Airport_Boundaries + Airport_Runways, Caltrans Public_Airport +
   Hospital_Heliports + AWOS, and the county ALUCP safety zones if published. Set the initial extent to
   the airport + its study ring. Keep DOF default-visible; stagger the UASFM grid after onReady.

C. /symbology + /popup — genuine ESRI drawingInfo/popupInfo on verified fields only.
   Obstacles: uniqueValue on TRIM(Type_Code) via Arcade, size-graduated by AGL, hollow when
   Verified='U'. Surfaces: low-alpha fills (~40/255) — horizontal blue, conical magenta, ALUCP zones a
   sequential ramp by zone, LAANC cells a light hatch labelled "advisory". Runways as heavy dark
   lines. Popups: obstacle title {Type_Code} {AGL} ft AGL, fields AMSL/Lighting/Marking/Verified/Date
   and Study as a link to oeaaa.faa.gov; ALUCP zone popup shows {Definition} and {ALUCP_link}; airport
   popup shows FACILITY/STATECLASS/MANAGER/MNGREMAIL/F5010URL.

D. /analyze — the numbers this app exists for. Compute the notice-slope contour bands with
   @strata/processing buffer (2,000 ft increments to 20,000 ft, each band carrying max_msl for the
   controlling runway length), the horizontal-surface radius (5,000 ft utility/visual vs 10,000 ft
   otherwise — ASK for the Part 77 runway designation, it is not in either open layer; default to the
   documented IAPEXISTS/LENGTH approximation and print which branch was used) and the conical ring
   (4,000 ft at 20:1), and pointsWithin for obstacles inside 1 NM of the probe. These
   bands ARE the day-1 fallback for `airspace-probe` — build them whether or not the widget ships.

E. /panel statistics as the KPI row: Verdict, Margin to binding rung, Height consumed (gauge),
   Obstacles in view, and the local-gate chip. Bind them with dataSource stat + fromWidget:"probe" so
   they update with no connections. The gate chip must render "referral status unknown" — never a
   green tick — where no ALUCP geometry is published.

F. /panel chart + /panel table — typechart (obstacles by TRIM(Type_Code)), heightchart (AGL
   histogram), and the neighbours AttributeTablePanel (Type_Code, AGL, AMSL, Verified, Lighting,
   Marking, Study, Date) with server paging, row → zoom + flash, and CSV/GeoJSON export. Add a carto
   cross-filter on the bench's obstruction register.

G. WIF: author AppLayout.connections — the 18 wires in §2.7. Verify each emitter is shipped
   (table→rowSelect, chart→categorySelect/rangeSelect, filter→filterChange, map→featureSelect/
   extentChange); do NOT wire buttonClick/timer/mapClick/sketchComplete, whose emitters are Phase-2
   pending. Register the three §2.6 widgets app-locally via <StrataApp registry={{...}}>; ship each
   one's named fallback in the same layout so the app is complete either way.

H. Controls + export: navigation/scale/geolocate; put the LayerPanel and BasemapPanel on the map as
   bottom-right controls ABOVE the zoom cluster (MapLibre prepends in bottom corners — add navigation
   first), with ESRI-idiom glyphs, wrapping layer names, and per-layer show/hide/REMOVE; Legend,
   Measure + Draw (both revert to identify), search (Nominatim) → probe site, a Clear-pin reset, and a
   status bar. Re-paint the ink-coloured layers on every basemap light↔dark flip (MapLibre paint
   cannot read CSS variables). Wire /export report as the referral packet (legend + scalebar + north arrow + the rung table
   with formulas and sources + the section profile + the ALUCP Definition/link), a batch atlas (one
   page per site), /export image, and a share deep-link with setUrlParam on scope.
```

## 6. Verify (benchmark: ArcGIS Aviation Airports · Transoft SkySAFE · the FAA Notice Criteria Tool)

| Check | Pass |
|---|---|
| Silhouette is the vertical **height ladder**; distinct at a glance from every Mobility sibling (triage-console · launchpad · ops-command · nearby-finder · time-player) | ⛏ |
| The signature loop works end-to-end: pin + height → verdict + binding rung + margin, in one screen | ⛏ |
| ≥3 `connections` fire on first render (design ships 18); five widgets link via `fromWidget` with none | ⛏ |
| Every `layerId` + field verified against the service (§4) — GUID join resolved, `Type_Code` trimmed | ⛏ |
| The §77.9/§77.17 arithmetic matches a hand-check against the FAA Notice Criteria Tool for 3 test sites | ⛏ |
| Verdict never says "approved"; every result prints its formula, its three inputs, their sources, and the `oeaaa.faa.gov` link | ⛏ |
| Missing ALUCP geometry renders "referral status unknown", not a green tick | ⛏ |
| Military fields refuse a civil verdict and explain UFC 3-260-01 | ⛏ |
| OEI/TERPS shown as a labelled not-computed placeholder rung — silence never reads as clearance | ⛏ |
| LAANC ceiling labelled advisory, not an authorisation | ⛏ |
| `responsive.small` collapses every side-by-side row; the ladder is legible as the phone page | ⛏ |
| Basemap **keyless throughout** — the user-switchable gallery is OSM-derived only (Dark Matter · Positron · Voyager · OSM · OpenTopoMap); everything EPSG:4326; `OBJECTID` is the OID | ⛏ |
| Layer control is the **single source of truth** for visibility — no control reads "off" while its layer is drawn | ⛏ |
| Ink-coloured layers re-paint on a light↔dark basemap flip (runway centrelines stay visible) | ⛏ |
| Clear-pin resets the probe and every derived widget, and keeps the airport's obstacle context | ⛏ |
| AMDB coded values decoded (never a raw `1`/`2`), runway id read from `RWY_ID`, runways sorted above buildings | ⛏ |
| Referral-packet PDF exports with the rung table + section + ALUCP `Definition`/`ALUCP_link`; batch atlas paginates | ⛏ |
| Runs on Strata **and** ArcGIS; the whole screening path is read-only (writes 🔶 behind `assertEsriBackend`) | ⛏ |

**On-par-or-better:** matches the FAA Notice Criteria Tool's screening question while adding the map,
the neighbours, the section, the local ALUC answer and the exported packet — none of which the FAA
tool has; and it reaches the 95% of users (developers, city counters, ALUC staff) that ArcGIS Aviation
Airports and SkySAFE, as licensed consultant-side OIS/OLS engines, do not serve. **Honest gap:** no
TERPS, no OEI, no authored approach/transitional multipatches, no eTOD/AIXM production, and geometric
(not behavioural) catchment.

## 7. Harvest (gaps → strata-core)

Log as strata-core issues: a **`height-ladder`** widget (a value on a regulated vertical axis with
per-rung provenance — generalises far beyond aviation: flood BFE vs. finished-floor elevation, utility
conductor clearance, height-overlay zoning, well-casing depth) — **this design's harvest candidate,
and a numbered template if it earns reuse twice**; a **`section-profile`** widget (terrain + a sloped
surface + projected features on one section); a **slope-surface analyze op** (`slopeSurface(origin,
ratio, reach)` returning banded polygons with a `max_msl` attribute — the generic form of the notice
bands); a **point-elevation DataSource kind** wrapping 3DEP/EPQS-style APIs; a **`TRIM()` helper** for
fixed-width ESRI attribute domains; and a **regulatory-citation popup element** (rule text + formula +
input provenance). Bigger asks (the honest boundary): **3D/multipatch OIS authoring** and a **TERPS/OEI
engine** — both currently on-hold/out-of-scope (3D/Scene is on hold per the manifest); **AIXM 5.1 /
eTOD export**; and **ICAO Annex 14 Amendment 18 OFS/OES** surface generation for the 2030 deadline,
which is the natural international follow-on.

## 8. Sources

**Regulation & standards**
- FAA — [14 CFR §77.9 notice criteria (eCFR)](https://www.ecfr.gov/current/title-14/chapter-I/subchapter-E/part-77/subpart-B/section-77.9) ·
  [14 CFR Part 77 (eCFR)](https://www.ecfr.gov/current/title-14/chapter-I/subchapter-E/part-77) ·
  [Form 7460-1](https://www.faa.gov/documentLibrary/media/Form/FAA_Form_7460-1_042023.pdf) ·
  [OE/AAA portal](https://oeaaa.faa.gov/) ·
  [Part 77 notification on airports](https://www.faa.gov/airports/central/engineering/part77) ·
  [Obstruction Evaluation / Airport Airspace Analysis](https://www.faa.gov/airports/northwest_mountain/engineering/airspace_analysis)
- FAA data & programmes — [ADIP](https://adip.faa.gov/) ·
  [Airports GIS / eALP programme](https://www.faa.gov/airports/planning_capacity/airports_gis_electronic_alp) ·
  [Airports Data & Airspace Branch (AAS-120)](https://www.faa.gov/about/office_org/headquarters_offices/arp/offices/aas/aas100/aas120) ·
  [NASR 28-day subscription](https://www.faa.gov/air_traffic/flight_info/aeronav/aero_data/NASR_Subscription/) ·
  [FAA AIS open data (adds-faa)](https://adds-faa.opendata.arcgis.com/) ·
  [Digital Obstacle File](https://adds-faa.opendata.arcgis.com/datasets/e202ff4e4cf943bda02ff63c0c44c9b7_0/about) ·
  [UAS Facility Maps](https://www.faa.gov/uas/commercial_operators/uas_facility_maps) ·
  [UAS Data Exchange (LAANC)](https://www.faa.gov/uas/getting_started/laanc)
- ICAO / international — [Annex 14 Aerodromes](https://store.icao.int/en/annex-14-aerodromes) ·
  [Adoption of Amendment 18 to Annex 14 Vol I](https://www.icao.int/sites/default/files/APAC/Meetings/2025/2025%20Workshop%20on%20Implementation%20of%20New%20ICAO%20Annex/Training%20Materials/SL-2025-23_amendment-18-to-Annex-14-Vol-I.pdf) ·
  [SKYbrary — eTOD](https://skybrary.aero/articles/electronic-terrain-and-obstacle-data-etod) ·
  [EUROCONTROL Terrain & Obstacle Data Manual ed. 3.0](https://www.eurocontrol.int/sites/default/files/2021-07/eurocontrol-tod-manual-ed-3-0.pdf) ·
  [IAA guidance on Annex 14 surfaces](https://www.iaa.ie/docs/default-source/publications/advisory-memoranda/aeronautical-services-advisory-memoranda-(asam)/guidance-material-on-aerodrome-icao-annex-14-surfaces.pdf)
- California — [Caltrans Division of Aeronautics — Airport Land Use Planning](https://dot.ca.gov/programs/aeronautics/airport-land-use-planning) ·
  [California Airport Land Use Planning Handbook](https://dot.ca.gov/-/media/dot-media/programs/aeronautics/documents/californiaairportlanduseplanninghandbook-a11y.pdf) ·
  [California airport land use planning (legal overview)](https://www.aviationairportdevelopmentlaw.com/articles/california-airport-land-use-planning/) ·
  [Santa Clara County ALUC](https://plandev.santaclaracounty.gov/hearings-and-committees/airport-land-use-commission) ·
  [San Diego ALUC mapping tool](https://www.san.org/Airport-Projects/Land-Use-Compatibility/ALUC-Mapping-Tool)

**Competitive / benchmark**
- Esri — [ArcGIS Aviation Airports overview](https://www.esri.com/en-us/arcgis/products/arcgis-aviation-airports/overview) ·
  [features](https://www.esri.com/en-us/arcgis/products/arcgis-aviation-airports/features) ·
  [OIS toolset (ArcGIS Pro)](https://pro.arcgis.com/en/pro-app/latest/tool-reference/aviation/an-overview-of-the-obstruction-identification-surfaces-toolset.htm) ·
  [Heliport OIS toolset](https://doc.esri.com/en/arcgis-pro/latest/tool-reference/aviation/an-overview-of-the-heliport-surfaces-toolset.html) ·
  [Perform obstacle analysis (tutorial)](https://learn.arcgis.com/en/projects/perform-obstacle-analysis-with-arcgis-aviation-airports/) ·
  [Airside & landside operations](https://www.esri.com/en-us/industries/airports/business-areas/airside-landside-operations) ·
  [Facilities & asset management](https://www.esri.com/en-us/industries/airports/business-areas/facilities-asset-management)
- Non-Esri — [Transoft AviPLAN airside planning](https://www.transoftsolutions.com/emea/aviation/software/airport-design-operations/aviplan/) ·
  [Transoft — preparing for ICAO's updated obstacle standards (SkySAFE)](https://www.transoftsolutions.com/aviation/resources/knowledge-center/preparing-for-icaos-updated-obstacle-standards/) ·
  [NV5 Geospatial — airport GIS, ALP & FAA compliance](https://www.nv5.com/geospatial/solutions/airports/) ·
  [Capitol Airspace — OE/AAA process](https://www.capitolairspace.com/obstruction-evaluation/faa-obstruction-evaluation-airport-airspace-analysis-oeaaa-process/) ·
  [Veovo — intelligent airport platform](https://veovo.com/) · [ADB SAFEGATE](https://adbsafegate.com/) ·
  [Airport digital-twin market (vendor landscape)](https://www.fortunebusinessinsights.com/airport-digital-twin-technology-market-117248)
- Public analogues — [Massport Logan Airspace Map](https://www.massport.com/logan-airport/about-logan/logan-airspace-map) ·
  [SJC Downtown Height Limits](https://www.flysanjose.com/downtown-height-limits) ·
  [RDU height zoning](https://www.rdu.com/airport-authority/environmental-programs/height-zoning/) ·
  [Dane County Airport Height Limitation Zone](https://danecountyplanning.com/Zoning/AHLZ) ·
  [Edinburgh aerodrome safeguarding](https://corporate.edinburghairport.com/about-us/aerodrome-safeguarding) ·
  [Newcastle tall-equipment operations](https://www.newcastleairport.com/corporate/aerodrome-safety/tall-equipment-operations/)
- Research / doctrine — [ACRP Research Report 195 — Best Practices for Airport Obstruction Management Guidebook](https://nap.nationalacademies.org/catalog/25399/acrp-research-report-195-best-practices-for-airport-obstruction-management-guidebook) ·
  [ACRP WebResource 7 — obstruction management library](https://crp.trb.org/acrpwebresource7/) ·
  [ACRP — Defining an Air Service Catchment Area](https://crp.trb.org/wp-content/uploads/sites/7/2016/10/E1_Tool2-DefiningAirServiceCatchmentArea.pdf) ·
  [ACI-NA — Redefining airport catchment areas](https://airportscouncil.org/2025/12/11/redefining-airport-catchment-areas-for-a-changing-aviation-landscape/) ·
  [ACI-NA — Comparing data sources for catchment analysis](https://airportscouncil.org/2025/07/22/comparing-data-sources-for-airport-catchment-analysis/)

**Internal** — `DESIGN-PROPOSAL.md` · `build_aviation_catalog.py` → `aviation-catalog-ca.json` ·
`../APP-TEMPLATE-LIBRARY.md` (assignment: `open-design` "height-ladder"; `tabbed-workbench` released) ·
`../../data_sources/data_sources_ca.md` · `strata/recipes/COMPONENT-MANIFEST.md` (§8 modernization,
§10 freestyle charter) · `strata/docs/guide/app-design.md` · `strata/docs/reference/human-language.md`

---

## Modernization (parity release)

> Applies the Experience-Builder-parity features (Phases 1–7). See `strata/recipes/MODERNIZATION.md`
> + `COMPONENT-MANIFEST.md` §8. Cross-cutting: a structured **`theme`**, app-shell
> (`header`/`footer`/`splash`), and `dataSource.{ sourceId }` linking.

- **Source linking is the architecture, not a garnish:** one `probe` output feeds `ladder`, `verdict`,
  `margin`, `section` and `neighbours` through `dataSource.fromWidget` — five widgets, **zero
  `connections`**. The `analysis` widget's catchment `bands` does the same for `catchtable`/`catchkpi`.
- **Layout nodes:** `splitter` (ladder rail \| map, resizable), `panel` (the left ladder rail — the
  tool dock moved onto the map as the bottom-right control stack), **`window`** (the referral packet,
  `open:false`, opened by `showHide`), `views` + `mapState` (the bench tabs drive the map),
  `accordion` (the phone collapse).
- **Data-source kinds:** `RestDataSource` for `aviation-catalog-ca.json` and the 3DEP/EPQS lookup;
  `FileDataSource` to upload a site list for **batch screening** → the atlas export.
  **No `StreamDataSource`** — no aviation feed here is genuinely real-time.
- **Motion:** `fly` + `stagger` on the ladder rungs (they settle like an instrument); nothing on the
  map container.
- **Theme:** one structured `ThemeSpec` derives hover/active/focus states and the type scale;
  `theme-switch` swaps the default dark ⇄ light chart-paper mode and pairs the basemap to it; the
  on-map keyless basemap gallery can then override that pairing.
- **Writes 🔶:** obstruction-case status via `updateRecord` behind `assertEsriBackend`; everything
  else is read-only and works unchanged on Strata Serve.
