# Recipe — Campus Operations: **"The Standards Rack"** (Education & Research)

> **RESEARCHED 2026-07-30.** Replaces the 2026-07-22 scaffold. §1 is page-verified, §3/§4 are
> curl-verified live, and the full evidence — every count, field list, value distribution and trap — is in
> **`DATA.md`**. The design rationale, the rejected silhouettes and the capability sweep are in
> **`DESIGN-PROPOSAL.md`**.

Buildings, assets, space and safety on one campus map — measured against the lines somebody else drew.

---

## 0.1 The working app in this folder — **built and driven**

```
cd app && python3 server.py     # → http://localhost:8061/
node test-rack.mjs              # 90 assertions  — the arithmetic and the live data
node test-render.mjs            # 143 assertions — boots the shipped script and drives the loop
```

**233 live-data assertions, all green 2026-07-30.** `app/index.html` is the whole app — self-contained,
MapLibre from CDN, no build step. See `app/README.md`.

It fetches the campus spine live (CSCD `/4` 145 · `/5` 266 · CSU Stanislaus `Buildings/0` 72, CORS `*`)
and carries the BOG 2020 rulebook transcribed verbatim, including all **32 §57028 TOP allowances**. The
interior is uploaded: `make-sample.py` writes a **synthetic** inventory over the **real** building list,
every row stamped `PROVENANCE=SYNTHETIC-DEMO`, with an undismissable banner, a renamed CSV export and a
`@media print` rule that forces the banner onto the exhibit.

**Driving it produced four corrections that the written design had wrong.** All are now asserted.
Two came out of the test harness:

1. **The what-if was answering the wrong question.** The proposal said pinning a building "recomputes
   the ratio without it". Implemented literally — drop its rooms *and* its meetings — the ratio moved
   **0.2 points**, because losing the building also lost the WSCH it carried, so load fell with
   capacity. That is not the question a facilities director asks. The counterfactual must be
   **capacity removed, load retained** — losing a building does not lose the students. Corrected, the
   same click moves lecture from **86.7 % → 73.4 %**, and the card now says which half of the ratio
   moved. (`test-render.mjs` §7.)
2. **An empty inventory rendered a confident red zero.** §57029 and §57030 compute a load from the
   header inputs alone, so with no rooms loaded the Office, Library and AV rungs read **0.0 %** — a
   claim, not an absence. With no inventory at all the five space rungs are now **grey / "no data"**;
   an empty *category* inside a loaded inventory still reads 0 %, because that is a real finding.
   (`test-render.mjs` §2.)

Running it in a browser found four more, guarded by `test-render.mjs` §1a/§1b:

3. **The property book's map was permanently empty.** Its map lives inside a hidden page, so it is
   0 x 0 at boot and fires `load` *after* `selectCampus()` has already run and skipped it on the
   `getSource` guard — and nothing retried. One `syncMap()` now hydrates any map at the moment it
   becomes paintable, which also replaced the bespoke re-hydration the theme switch was carrying.
4. **Campus land drew in default grey** on page 1 because the Clery bucket the `fill-color` matches
   on was only stamped by page 2's render. The legend promised teal and the map delivered grey.
5. **The header had drifted from its own skeleton.** The identity line omitted the campus, the
   standard's **vintage** sat in the rail instead of header line 2, and the **`term` picker the design
   specifies was never built** — so load was pooled across every term in the upload, double-counting
   contact hours. Rebuilt to §3's two-line skeleton; the term now scopes `compute()`, not just the ledger.
6. **One nowrap toolbar held the whole page open.** The ledger's header row propagated a 553 px
   min-content floor up the flex chain past a 500 px viewport, and because the body clips its overflow
   the rack's values and distances fell off the right edge on a phone. `min-width:0` down the chain.

A later review pass (English-only, one map control cluster, resizable panes) surfaced the report that
**"some places don't have data, like Sierra College"**, which turned out to be two more bugs plus a
presentation failure:

7. **A campus switch could leave the previous campus's buildings on the map.** Boot auto-selects a
   campus; a user pick before that resolves left *both* selections running, and the slower one won.
   `selectCampus` now stamps each selection and only the newest commits.
8. **Multi-site districts fitted the union of their properties.** Sierra College is five properties
   spanning **1.09° of longitude** (Rocklin · Nevada County · Tahoe-Truckee · Roseville · open space),
   so the map zoomed out until every campus was an invisible speck — which reads as "no data". The map
   now fits the property CSCD flags **MAIN** (span 1.09° → 0.014°) and a **site strip** names the rest.
   Note the fit could not key off `cleryBucket`: under 34 CFR 668.46 every teaching site is on-campus
   geography *for its own campus*, so all four of Sierra's sites bucket as `oncampus` — correctly.
9. **"No data" was indistinguishable from "nothing published here."** The layers drawer now says
   *"No public building footprints are published for this campus"*, and a basemap that fails to load
   says so instead of rendering black.
10. **The real scarcity, measured.** All **411** CSCD properties have geometry — **0.00 %** missing.
   But **exactly 1 of the 200 institutions (0.5 %) publishes building footprints**, so 199 campuses
   had land and nothing inside it. The layers drawer now offers **footprints from OpenStreetMap**
   on demand for any campus (46 inside Sierra's Rocklin polygon, clipped from 61 in the bbox),
   attributed ODbL and labelled **context only** — OSM ways carry no building id, so they cannot
   join to an inventory and never yield an ASF. User-triggered, never wired into boot: Overpass is a
   free public endpoint that returns 429/504 often enough to need two tries and a plain-English
   failure message (`DATA.md` T8a).
11. **The property book's panes were not resizable.** Page 1 got three dividers in the previous
   pass and page 2 got none — yet its ASR property schedule is the widest table in the app and the
   thing a Clery officer most wants to drag open. Both pages now carry the same three.
12. **Every abbreviation now explains itself in place.** DGE, FTEF, ASF, WSCH, TOP, FCI, the four
   Clery categories, capacity vs load, and all eleven rungs carry an **i** icon whose tooltip gives
   the definition and, where one exists, the section it comes from. The wording lives in a single
   `GLOSS` table so it cannot drift between the header, the rack and the ledger, and the bubble is
   one element on `<body>` because the rail, the side panel and the ledger are all `overflow:auto`
   and would clip a CSS tooltip. Suppressed in the printed exhibit.
13. **"Sierra College looks empty on the map" — three compounding causes, all measured.**
   (a) The **site strip crushed the map**: five properties wrapped it onto four rows and, in a flex
   column whose ledger is a fixed 230 px, that height came straight out of the map — down to
   **27 px**, at which MapLibre silently never finishes loading its style. The strip is now one
   scrolling row and `.boardtop` has a 240 px floor.
   (b) The camera **flew** between campuses; when the frame loop is starved the animation never
   arrives and the map simply stays on the previous campus. A campus change now **jumps**.
   (c) Our own sources were added only on `load`, which fires **at most once and not at all** when
   the basemap style never resolves — so a slow CDN left a black rectangle with the campus data
   sitting unused in memory. The map now hydrates on **`styledata`** as well, `installLayers` is
   idempotent, and after 8 seconds without a basemap it falls back to a blank style and says so:
   *"the map is drawn without it — your campus, buildings and Clery geography are all still here."*
14. **"Stanislaus is the only campus that renders."** Measured across seven institutions, every one
   loaded its properties into the map source and moved the camera to the right longitude — the data
   path was never broken. What differed was **visibility**: Stanislaus is the single campus with
   published footprints, and its 72 bright polygons made it *look* rendered, while every other campus
   was a 16 %-opacity wash and a 2 px line on a dark basemap. The boundary now carries a **casing**
   (8 px glow + 2.4 px line) and the fill steps to **0.30** when there are no buildings to read
   through, dropping back to 0.16 when there are. The footprint offer also moved out of the layers
   drawer onto the **map itself**, where the emptiness is — *"No published footprints for UC Davis.
   [Get them from OpenStreetMap]"* — dismissible per campus. A third hydration hole closed with it:
   `styledata` can fire while a style is still resolving, so the guard skipped every firing with
   nothing left to retry; the map now also hydrates on **`idle`**.
15. **⚠ The panels were showing one campus's inventory under another campus's name.** Reported as
   *"Stanislaus is the only campus that renders on the map **and on the panels**"*. The uploaded
   inventory was never cleared on a campus change, so after switching to Pasadena City College the
   rack still read **24,400 ASF**, the ledger still listed CSU Stanislaus's 25 rooms, and the
   derivation card still showed its WSCH — all under the heading "Pasadena City College". In an app
   whose entire claim is that the arithmetic can be checked, silently re-attributing one campus's
   data to another is the worst failure available, and it had shipped. An inventory now belongs to
   exactly one campus: switching clears rooms, schedule, condition, the term, the pinned building
   and the synthetic banner, and the space rungs return to "no data" rather than to the last
   campus's numbers.
16. **The demo now works for every campus, not just the one whose CSVs ship.** "Load demo inventory"
   generates a synthetic inventory over **that campus's own buildings** — fetching footprints from
   OpenStreetMap first if it has none — seeded so it is stable, filtered so parking structures and
   canopies do not acquire classrooms, and sized against the DGE and FTEF in the header so §57029
   and §57030 land near the standard instead of miles under it. Room sizes come from the statutory
   per-station figures and the seeded PRNG, **never from footprint geometry** (T12 holds; the suite
   asserts no path from `Shape__Area` into `synthInventory`). Pasadena City College: 35 footprints →
   28 rooms, lecture **85.6 %** deficit, laboratory **138.3 %** surplus — the same true story the
   shipped demo tells, computed from its own buildings.

The demo's numbers also turned out to teach the recipe's central point better than any caption:
classrooms used hard (52.0 hrs/wk vs the 48-hour line) land in **deficit at 86.7 %**, while
under-used laboratories (21.5 hrs vs 27.5) read as an apparent **surplus at 140.7 %** — because
§57032 sizes your need from the hours you actually schedule. `test-rack.mjs` §13 pins that inversion
so a future change cannot quietly break it.

---

## 0. Provenance

| | |
|---|---|
| **Source** | `https://tabaqat.net/solutions` — **Education & Research** section |
| **Name on site** | Campus Operations |
| **Tagline on site** | "Buildings, assets, space and safety on one campus map" |
| **Scaffolded** | 2026-07-22 (template `launchpad`, provisional) |
| **Researched & designed** | 2026-07-30 |
| **Template** | **`open-design` — "standards-rack"**. `launchpad` released back to the library |
| **Demo region** | California — CSU Stanislaus (Turlock) geometry on the statewide CSCD spine |

---

## 1. Study — how the market frames this

### 1.1 The question the buyer asks

> *"The Chancellor's Office says we don't qualify for a new lecture building. Facilities says three of ours
> are falling down. The Clery officer needs the property list by October 1. They are all talking about the
> same square feet — so why can't I see them on one map, and why can't I show my work?"*

### 1.2 The lines a California campus is measured against

Every one of these is a **published threshold** applied to the **same square feet**, by a **different
authority**, on a **different clock**:

| Line | Authority | Decides |
|---|---|---|
| Capacity / load **100 %** | Title 5 CCR **§57020** → BOG *Policy on Utilization and Space Standards* (2020 revision) | state capital-outlay eligibility |
| Classroom room use **≥ 53 hrs** of a 70-hour week (**48** below 140,000 WSCH) | **§57021** | the capacity half of that ratio |
| Classroom station occupancy **≥ 66 %**; laboratory **≥ 85 %** | **§57023 / §57024** | ditto |
| **20 ASF** per classroom station; lab ASF by TOP code | **§57025 / §57028** | ditto |
| Laboratory room use **≥ 27.5 hrs** / 70-hour week | **§57022** | ditto |
| **FCI 0.10** | APPA convention, carried in the FUSION Facility Condition Assessment | renovate vs. replace; the deferred-maintenance ask |
| Clery geography — on-campus · on-campus student housing · non-campus · public property; ASR due **Oct 1** | **34 CFR 668.46** | which crimes must be disclosed, and where liability sits |
| *"An annual inventory of all facilities of the district"* | **Ed Code §81821(e)** | why the data exists at all |

**The formula is one line** (§57032): `ASF/STN ÷ (Hrs/Wk × STN Occ) × 100 = ASF per 100 WSCH`.
Worked, verbatim from the policy: `20 ÷ (53 × 0.66) × 100 = 57.2` — and `20 ÷ (48 × 0.66) × 100 = 63.1` for a
campus under 140,000 WSCH, and `55 ÷ (27.5 × 0.85) × 100 = 235` for a biological-science laboratory.

**The vintage moves the line.** The 2020 revision **raised Lecture standards 33 % and Office 25 %**. Any
ratio computed against the 2010 table is now wrong, and published pre-2020 ratios are not comparable to
post-2020 ones. A capacity/load number without its standard vintage on it is not a number.

### 1.3 Reference solutions (benchmark + coexist, never copy)

**Esri — the incumbent.** `esri.com/en-us/industries/higher-education/roles/administrators` (fetched
2026-07-30) names **ArcGIS Indoors**, **Indoor GIS**, **ArcGIS Field Maps** and **ArcGIS for AutoCAD** across
three workflows: campus and space mapping · real-time asset management and tracking · safety and situational
awareness. Named customers: University of Rhode Island (replaced a legacy CAFM — Esri's own *ArcUser*
write-up is titled "Replacing a Legacy CAFM System with ArcGIS Indoors"), SUNY Cortland, **UC San Diego**
(1,158 acres), University of Hawai'i at Mānoa, ETH Zürich, UT Austin. Esri also runs a **Campus Operations**
community blog and works with the **Campus FM Technology Association**.

**Non-Esri — where the market actually is.** This category is IWMS/CAFM, not GIS:

| Vendor / product | Owns | Blind to |
|---|---|---|
| **AssetWorks AiM / ReADY** — trade press calls it the dominant higher-ed platform, *"over one billion square feet across hundreds of institutions"* | O&M, space, real estate | the regulator's arithmetic; the map |
| **Archibus (Eptura)**, **Planon**, **FM:Systems**, **IBM TRIRIGA**, **Accruent** | IWMS: space, lease, maintenance, capital projects | floor plans are CAD polylines, not geography |
| **Concept3D / CampusBird**, **MazeMap**, **Pointr** | the public campus map, wayfinding | anything a planner files |
| **Ad Astra**, **CollegeNET 25Live** | scheduling; hours-per-week | condition, hazard, Clery |
| **Gordian / Sightlines** | FCI + deferred-maintenance benchmarking | delivered as consulting, not a live surface |
| **Occuspace** and sensor peers | real occupancy | inventory, standards, compliance |
| **FUSION** (Foundation for CA Community Colleges) — the CCC system of record; a web DB of **>90 M sq ft**, activated May 2003, carrying the FCA and every district's five-year capital outlay plan | the inventory | it is a form, not a map |

Each owns **one column**. None puts the columns side by side against their published lines, on a map, with
the arithmetic visible.

**The market's own numbers** (used only as framing, never rendered as this campus's data): campus classrooms
average roughly **40 % utilization even at peak instruction hours** (Occuspace, Feb 2026); US higher ed
carries a **$750–950 bn** deferred-maintenance and mission-critical backlog and spends on the order of
**$79 bn a year** on space nobody occupies.

### 1.4 Our edge

AI-authored, MIT/open, runs on **Strata *or* ArcGIS**, sovereign/on-prem — which matters when the inputs are
a student-schedule extract and a police incident log. And one thing the commercial field is structurally
unable to offer: **we can show the arithmetic, because nothing proprietary is hidden inside it.** A vendor
whose scoring engine *is* the product cannot open it; ours is a published regulation, and opening it is the
feature.

Coexistence: Indoors keeps the floor plans, AiM keeps the work orders, FUSION keeps the inventory of record.
This app reads their exports and answers the question none of them is shaped to answer.

### 1.5 Standards, specifications & organizations to speak fluently

- **Title 5 CCR §§57020–57032** and the **BOG Policy on Utilization and Space Standards** (2020) — the CCC
  rulebook. Authority cited in the policy: Ed Code **66700, 70901, 81805, 81836**; reference: **81805**.
- **Ed Code §81821(e)** — the annual district facilities inventory.
- **FUSION** — Facility Utilization Space Inventory Options Net; the CCC space inventory, FCA and five-year
  capital outlay plan submission system.
- **CCC Space Inventory Handbook** (Chancellor's Office; re-issued under facilities memo FS-21-06).
- **TOP codes** — Taxonomy of Programs; the key on which §57028's lab allowances are indexed.
- **WSCH** — Weekly Student Contact Hours, 8 a.m.–10 p.m. (the "70-hour week").
- **NCES FICM** — *Postsecondary Education Facilities Inventory and Classification Manual*: the national
  space-code vocabulary (100 classroom, 200 laboratory, 300 office, 400 study, 500 special use, 600 general
  use, 700 support, 800 health, 900 residential).
- **APPA** — the FCI convention and total-cost-of-ownership language. **IFMA**, **BOMA** — the FM/measurement
  bodies. **SCUP** — campus planning.
- **Clery Act / 34 CFR 668.46**; the *Handbook for Campus Safety and Security Reporting*; the ASR/AFSR and
  its **Oct 1** deadline; the OPE Campus Safety and Security data collection (keyed on **OPE ID**).
- **IPEDS / UNITID** — the federal institution key.
- **DSA / the Field Act** — Division of the State Architect review for K-12 and community-college
  construction. **AB 300 (1999)** — the CSU/UC seismic building inventory.
- **IMDF** (Apple Indoor Mapping Data Format), **OGC IndoorGML**, **Esri AIIM**, **IFC/BIM** — the indoor
  interchange formats an institution's floor plans will arrive in.

### 1.6 Honest scope — what this is **not**

- **Not the system of record.** FUSION/AiM/Archibus stay authoritative for the inventory; this reads their
  export and stamps it with a date.
- **Not an indoor mapping product.** No floor-plan editing, no unit/level model, no wayfinding, no routing.
  If the client needs those, they need ArcGIS Indoors and we coexist with it.
- **Not a live occupancy sensor.** The utilization rungs read **scheduled** hours from a term extract.
  Scheduled ≠ occupied — that divergence is precisely why the industry's peak-utilization figure is 40 % —
  and the rung is labelled *"scheduled room use, per §57021"* for that reason.
- **Not a Clery determination.** CSCD's `C_TYPE` is a land-ownership classification by GreenInfo Network. The
  app renders it as **suggested**; the institution's Clery officer confirms or overrides every row.
- **Not a submission tool.** It produces the exhibit and the CSV; the district still files through FUSION.
- **Approximated:** the public-property band is a **buffer** of the campus boundary, not a
  right-of-way-accurate parcel analysis, and it is labelled as such.

---

## 2. UI design spec (front-loaded)

### 2.1 Layout (Template: **`open-design` — "standards-rack"**)

- **Template** `open-design` under the freestyle charter (`strata/recipes/COMPONENT-MANIFEST.md` §10):
  registry widgets and manifest config keys only, plus two §10.3 app-local widgets each with a named
  fallback (§2.6). **Never** fall back to `split-dashboard`.
- **Why this shape:** the buyer's mental model is literally *"am I over or under the line?"*. So the
  navigation is a **rail of rungs — one per published threshold — each drawn identically**: label · citation ·
  current value · bar · a fixed **line marker** · a signed distance. One motif, **eleven** times; the app
  teaches itself in about four seconds.
- **Signature loop:** `rung → map repaints + ledger narrows + the derivation card prints §57032 with this
  campus's own numbers → click a "mover" building → zoom + flash + ledger scoped → the ratio recomputes
  without it → export the exhibit.`
- **The signature accent is "show your work"** — the derivation card. It is the axis on which an open,
  on-prem tool structurally beats a commercial IWMS.
- **Two pages:** `rack` (eleven rungs in three books — Space 5, Utilization 4, Condition & compliance 2)
  and `book` (the Clery property book — the same rail motif with the four geography buckets as its rungs).
- **One map control cluster** (zoom · zoom-out · zoom-to-campus · basemap · layers), Calcite-idiom inline
  SVG, replacing MapLibre's own zoom cluster so there is only one set of buttons.
- **Six drag handles** — rail ⇄ map, map ⇄ side, board ⇄ ledger, on **both** pages.
- **Every abbreviation carries a definition** — one `GLOSS` table, 42 terms, hover/focus, hidden in print.
- **Wiring floor:** **32 connections** authored, ≥3 required (`docs/guide/app-design.md` §3). Five more
  widgets link with **zero** connections through `dataSource.sourceId`.
- **Responsive:** `responsive.small` collapses the splitter to a column, rack first (`dock:"top"`), map at
  45 vh, KPI trio to a `flow-row` of chips. `.boardtop` keeps a **240 px floor** — below that MapLibre
  silently never finishes loading a style, and the map goes blank rather than small.

ASCII skeleton, the full `AppLayout` JSON, and the phone behaviour: **`DESIGN-PROPOSAL.md` §3–§4**.

### 2.2 Theme

Structured `ThemeSpec`, **dark** — the rack is a console read at a glance, often on a wall or in an evening
operations room — with a real **light** mode behind `theme-switch` that is also the print document: every
exhibit prints light whichever mode is on screen.

```jsonc
{ "mode": "dark",
  "colors": { "primary": "#4fd1c5", "secondary": "#a0aec0", "success": "#68d391", "info": "#63b3ed",
              "warning": "#f6ad55", "danger": "#fc8181", "light": "#f7f6f3", "dark": "#0d1512" },
  "fonts": { "scale": "default", "mono": "ui-monospace, Menlo, monospace" },
  "variables": { "--strata-radius-md": "6px", "--strata-motion-base": "160ms" },
  "overrides": { "kpi": { "--strata-mono": "ui-monospace, Menlo, monospace" } } }
```

Semantic status is load-bearing: `success` = past the line, `warning` = within 5 points, `danger` = short.
Basemap keyless and mode-paired (`basemapForTheme` → CARTO Dark Matter / Positron), and the building-share
ramp inverts with the mode so the strongest contribution is always the most salient ink. Polygon fills at ~40/255
so a building reads through three stacked overlays. Tabular monospaced numerals wherever a figure meets a
threshold. **No container `animate`** — a thresholds console that slides is a console you distrust.
**English only** (the AR/RTL leg was removed on review — see §0.1). The string table survives so every
user-facing phrase stays in one place, but there is one locale and no direction switching.

### 2.3 KPI cards

Four, all live `kpi.stat` bound to a shared `dataSource.sourceId` so they update on filter/selection with
**no `connections`**:

| Card | `stat` | Source | Threshold colour |
|---|---|---|---|
| **Capacity ASF** | `{ field: "ASF", op: "sum" }` | `rooms` | — |
| **Load ASF (§57032)** | `{ field: "LOAD_ASF", op: "sum" }` | `rooms` | — |
| **Distance to the line** (`gauge`) | `{ field: "CAP_LOAD_PCT", op: "avg" }`, `thresholds: [90, 100]` | `rooms` | danger < 90, warning 90–100, success ≥ 100 |
| **Acres classified** (page 2) | `{ field: "Acres", op: "sum" }` | `campus-land` | vs. 100 % of owned acres |

Plus `clery-kpi-pub`, bound `fromWidget: "clery-buf"` — the public-property band's acreage, computed at
runtime by the `analysis` buffer.

### 2.4 Charts & table

- **`chart` — "Movers"**: a bar chart, `field: "BLDG_ID"`, `valueField: "ASF"`, `operation: "sum"`,
  `limit: 12`, over the `rooms` source. Emits `categorySelect` → `zoomTo` + `flash` on the map and `filter`
  on the ledger. Real columns, no invention: `BLDG_ID` and `ASF` are declared required columns of the
  institution's inventory export.
- **`carto` (page 2)**: a `category` widget on `campus-land.C_Type` + a `formula` widget summing
  `campus-land.Acres`. Both fields verified live (`DATA.md` §2.1).
- **`table` — the room ledger**: `virtualize: true`, `viewportHeight: 260`, `oidField: "OBJECTID"`, columns
  `BLDG_ID · ROOM · TOP_CODE · SPACE_TYPE · ASF · STATIONS · HRS_WK · OCC_PCT · FCI · CLERY_BUCKET`.
  `rowSelect` → `zoomTo` + `viewInTable`. CSV/GeoJSON export native.
- **`table` (page 2) — the ASR property schedule**: `NAME · C_Type · Acres · F_Type · CONFIRMED_BY`. The
  CSV *is* the Clery geography appendix.

### 2.5 Capabilities to use (Phases 0–7)

Layout `splitter` + `panel` + `accordion` + `window` · widgets `map legend carto filter table chart kpi gauge
feature-info data-actions analysis text button add-data share print status-bar theme-switch basemap layer-panel
page-nav` · triggers `categorySelect filterChange rowSelect featureSelect extentChange recordsChange
flash clear` · actions `filter zoomTo flash viewInTable showStatistics setUrlParam` · `dataSource.sourceId`
linking (5 widgets, no wires) and `fromWidget` (1) · **`FileDataSource`** CSV upload for the whole interior ·
`@strata/processing` **buffer** for the public-property band · Arcade `valueExpression` on the buildings
renderer · structured theme, app-shell (`header`/`footer`/`splash`), i18n EN+AR.

**Deliberately not used** — with reasons, not omissions: `views`/`mapState` (the rack is already the state
machine), `time slider` (no honest time axis — see the rejection of the week-grid silhouette in
`DESIGN-PROPOSAL.md` §2), routing/isochrones (no travel-time question), search (you don't geocode your own
campus), `weighted-overlay` (the weights are in statute, not in the user's opinion), hexbin/hotspot
(density over 72 buildings on 212 acres is noise), atlas (one campus is one page), `RestDataSource`/
`StreamDataSource` (a space inventory changes annually; live-looking widgets would be theatre).

**Not wired, and why** — `buttonClick`, `viewChange`, `pageChange`, `mapClick`, `sketchComplete`, `timer`:
the trigger types and dispatchers ship, but the **widget emitters are pending** (Manifest §4.1). The "Read
the standard" `button` + `std-ref` `window` therefore ship *unwired*, logged in §7. A connection that never
fires is worse than none.

Full sweep table: `DESIGN-PROPOSAL.md` §8.

### 2.6 The two new widgets (§10.2/§10.3) and their day-1 fallbacks

| Key | Purpose | Emits | Fallback that ships day 1 |
|---|---|---|---|
| **`standards-rack`** | N rungs, each a published threshold in one repeated form (label · cite · value · bar · line marker · signed distance) | `categorySelect` `{field:"STANDARD", value:rung.id}` | an `accordion` of `stacked-bar` (capacity vs load) + a `gauge` per rung with `thresholds` at the line |
| **`derivation-card`** | Prints the statute's arithmetic as a ledger with this campus's numbers substituted | — (terminal display) | a `table` of `{term, value}` rows under a `text` widget holding the formula + citation |

Full contracts: `DESIGN-PROPOSAL.md` §9.

### 2.7 `connections` (32 — every wire uses a shipped emitter)

Full table with the user-visible behaviour of each: `DESIGN-PROPOSAL.md` §5. The spine:

```jsonc
{ "from": "rack-space", "trigger": "categorySelect", "to": "map",    "action": "filter", "options": { "layerId": "buildings" } },
{ "from": "rack-space", "trigger": "categorySelect", "to": "ledger", "action": "filter" },
{ "from": "rack-space", "trigger": "categorySelect", "to": "deriv",  "action": "showStatistics" },
{ "from": "rack-space", "trigger": "categorySelect",                 "action": "setUrlParam", "options": { "param": "standard" } },
{ "from": "movers",     "trigger": "categorySelect", "to": "map",    "action": "zoomTo", "options": { "layerId": "buildings" } },
{ "from": "ledger",     "trigger": "rowSelect",      "to": "map",    "action": "zoomTo", "options": { "layerId": "buildings" } },
{ "from": "map",        "trigger": "featureSelect",  "to": "ledger", "action": "filter" },
{ "from": "map",        "trigger": "extentChange",   "to": "kpi-cap","action": "showStatistics" },
{ "from": "clery-buf",  "trigger": "recordsChange",  "to": "clery-kpi-pub", "action": "showStatistics" }
```

---

### 2.8 As shipped — the surface the build added

The design in §2.1–2.7 is what was specified; these are the capabilities the working app needed and now
has. Each is asserted in `app/test-render.mjs` (section in brackets).

| Shipped | Why it exists |
|---|---|
| **Map control cluster** — zoom · out · home · basemap · layers *(§11a)* | §8 had recorded `basemap` and `layer-panel` as *deliberately not used*. Wrong in practice: with no basemap control the user cannot leave the theme's basemap, and with no layer control a campus with no footprints is indistinguishable from a broken map |
| **Basemap drawer** — CARTO Dark Matter / Positron / Voyager, keyless *(§11a)* | the house rule is keyless OSM-derived only; the drawer enforces it by offering nothing else |
| **Layers drawer** + the OSM offer *(§11c)* | 199 of 200 campuses publish no footprints; the drawer says so and offers OpenStreetMap on demand |
| **Six resizable panes**, pointer + keyboard *(§11b)* | the ASR property schedule is the widest table in the app and is what a Clery officer drags open |
| **Multi-site strip** *(§10a)* | 101 of 200 institutions hold more than one property; Sierra College's five span 1.09° of longitude |
| **Glossary — 42 terms, `ⓘ` in place** *(§11d)* | "show your work" is not only the derivation card; an abbreviation nobody can expand is the opposite of it |
| **Term picker** *(§3)* | load is a per-term quantity; pooling terms double-counts contact hours |
| **Per-campus demo generator** *(§11c1)* | the shipped CSVs are keyed to one campus's building ids; every other campus generates over its own buildings, seeded, filtered, sized against the header's DGE/FTEF |
| **Campus casing + 0.30 fill when empty** *(§11c2)* | the ~40/255 low-alpha rule is for *stacked* overlays; with one polygon and nothing inside it, 0.16 is invisible |
| **Blank-style fallback + hydration on `load`/`styledata`/`idle`** *(§11e)* | the map is the evidence for every number; it must render with no third-party tile server at all |
| **English only** *(§11)* | the AR/RTL leg was built and then removed on review — the audience for this build is Californian |

## 3. Data sources

**Full inventory, field lists, counts, value distributions and 12 traps: `DATA.md`.** Everything below was
fetched live **2026-07-30**. All EPSG:4326 on ingest.

### 3.1 The headline

`CA-DATA-CATALOG.md` §5 recorded this recipe's theme as *"inherently client-owned … no public CA feed
exists"*. That is **still true of the interior** and **now false of the exterior**:

| Catalogue said missing | Verified to exist |
|---|---|
| campus boundaries | **CSCD 2025** `University_lands` (145) + `CA_Community_Colleges` (266), carrying **both** `IPEDS_ID` and `OPE_ID` |
| campus building footprints | **CSU Stanislaus `Buildings`** — 72 polygons, `Access-Control-Allow-Origin: *` |
| campus property classification | CSCD `C_TYPE` already splits MAIN / OFF-CAMPUS / **STUDENT HOUSING** / OPEN SPACE |
| rooms, ASF, stations, schedule, FCI | ⛔ still client-owned — **upload path, no substitute, do not synthesize** |

### 3.2 Role × region

*(Maryland was not swept — the demo campus and the binding authority are Californian. The third column is
where this recipe genuinely differs.)*

| Role | California (verified 2026-07-30) | National | Client-supplied (upload) |
|---|---|---|---|
| **Campus land** ★★ | CSCD 2025 `…/California_School_Campus_Database_2025/FeatureServer/4` (145) · `/5` (266) | IPEDS points only | campus GIS boundary |
| **Buildings** ★ | `services6.arcgis.com/rX5atNlsxFq7LIpv/…/Buildings/FeatureServer/0` (72, CORS `*`) | OSM / Overture (ODbL) | CAD/BIM footprint export |
| Institution registry | Cal OES `…/HigherEducation/FeatureServer/0` (679, `UNITID`) · `…/Universities_and_Colleges_in_California/FeatureServer/0` (694) | IPEDS directory | — |
| K-12 variant | CDE `…/fdvHcZVgB2QSRNkL/…/SchoolSites2526/0` (9,946) · `DistrictAreas2526/0` (936) · `DistrictSites2526/0` (1,009) | NCES CCD | — |
| UC property | UCOP `…/UC_Land_Parcels_and_Tribal_Lands_WFL1/FeatureServer/10` (3,439) | — | — |
| **Room inventory** | ⛔ | ⛔ | **required** — FUSION / AiM / Archibus |
| **Term schedule** | ⛔ | ⛔ | **required** — Banner / Colleague / PeopleSoft / 25Live |
| Condition (FCI) | ⛔ FUSION FCA is login-gated | Gordian is commercial | optional — FUSION FCA export |
| Clery statistics | ⛔ | `ope.ed.gov/campussafety` — **manual CSV, no API** (§4) | optional — campus police log |
| Wildfire hazard | `services.gis.ca.gov/…/Environment/Fire_Severity_Zones/MapServer` (layers 0–3) | — | — |
| Flood | Cal OES `…/100_Year_Floodplain/FeatureServer/`**`4`** ⚠ | FEMA NFHL | — |
| Seismic | CGS `gis.conservation.ca.gov/…/CGS_Earthquake_Hazard_Zones` · `…/Liquefaction/MapServer` | USGS | — |
| Transit / commute | Caltrans `…/CHrailroad/CA_HQ_Transit_Stops/FeatureServer/0` | — | — |
| The rulebook | BOG 2020 policy PDF, extracted verbatim → app config | — | client's own standard profile |
| Basemap | keyless OSM · CARTO Positron/Voyager/Dark · OpenTopoMap | | |

### 3.3 The demo campus

**CSU Stanislaus, Turlock.** CSCD `/4`: `NAME='California State University, Stanislaus'`, `IPEDSID=110495`,
`Category='CSU'`, `C_Type='MAIN CAMPUS'`, `Acres=211.735` — **plus** a second row,
`C_Type='OFF-CAMPUS PROPERTY - OPEN SPACE'`, `Acres=4.452`. So the demo has a genuine on-campus /
non-campus pair to classify on day one. Buildings extent (4326):
`-120.86185463, 37.52267657 → -120.84965314, 37.52912008`.

⚠ **Standards mismatch, stated openly:** CSU Stanislaus is a **CSU**; the capacity/load rulebook shipped by
default is the **CCC's** (Title 5 §57020). CSU and UC report utilization to the Department of Finance under
their own conventions. Label the demo *"CCC standards, illustrated on CSU Stanislaus geometry"* — the rack's
`rungs` are config precisely so the rulebook is swappable. See `DESIGN-PROPOSAL.md` §11 R5/R6.

---

## 4. Verify each URL first (terminal)

```bash
# ── 1. THE CAMPUS SPINE. GreenInfo Network, California School Campus Database 2025.
#      7 layers; SR 3857 -> always request outSR=4326; oid OBJECTID on every layer.
B=https://services1.arcgis.com/4ZKi1B1zTblbwgWB/arcgis/rest/services/California_School_Campus_Database_2025/FeatureServer
curl -s "$B?f=json" | jq '.layers[] | {id, name}'
#  -> 0 Schools_Current_Stacked · 1 School_Centroids · 2 School_Property · 3 Closed_Schools
#     4 University_lands · 5 CA_Community_Colleges · 6 NonPublicK12School_EducationOwnedLands
curl -s "$B/4/query?where=1=1&returnCountOnly=true&f=json"   #  -> {"count":145}
curl -s "$B/5/query?where=1=1&returnCountOnly=true&f=json"   #  -> {"count":266}
curl -s "$B/4?f=json" | jq -r '.fields[].name'
#  -> OBJECTID NAME IPEDSID Category SourceA Campus C_Type Location URL Notes Acres F_Type
curl -s "$B/5?f=json" | jq -r '.fields[].name'
#  -> OBJECTID CC_ID NAME CC_ID_1 IPEDS_ID OPE_ID DISTRICT SOURCEA CAMPUS C_TYPE STREET CITY ZIP
#     COLLEGE_UR CAMPUS_URL CAMPUS_MAP PUBLIC_K12 CAMPUS_SET CAMPUS_HOU NOTES ACRES Shape_Leng
#     ^^ IPEDS_ID and OPE_ID — the IPEDS key AND the Clery filing key, on a polygon. Nothing else
#        in the CA catalogue carries both.

# ── 2. THE C_TYPE TRAP. Two spellings of the same class; blank Category is '' for some rows and ' '
#      for others. Never use '='.
curl -s "$B/4/query?where=1=1&outFields=Category,C_Type,F_Type&returnDistinctValues=true&returnGeometry=false&f=json"
#  -> C_Type: MAIN CAMPUS | OFF-CAMPUS PROPERTY | OFF-CAMPUS PROPERTY - HOUSING
#             OFF-CAMPUS PROPERTY - OPEN SPACE      <-- CSU rows, hyphen
#             OFF-CAMPUS PROPERTY OPEN SPACE        <-- UC rows, NO hyphen
curl -s -G "$B/4/query" --data-urlencode "where=Category='CSU'" --data-urlencode "returnCountOnly=true" --data-urlencode "f=json"
#  -> {"count":32}      Category='UC' -> {"count":16}     C_Type='MAIN CAMPUS' -> {"count":110}
curl -s "$B/5/query?where=1=1&outFields=C_TYPE&returnDistinctValues=true&returnGeometry=false&f=json"
#  -> STUDENT CAMPUS, MAIN | STUDENT CAMPUS | OFF-CAMPUS PROPERTY - STUDENT HOUSING
#     OFF-CAMPUS PROPERTY - OPEN SPACE
#     'STUDENT CAMPUS, MAIN' -> {"count":114}   (of 116 colleges — near-complete)
#     CAMPUS_HOU='Yes'       -> {"count":9}     <-- Clery "on-campus student housing" candidates

# ── 3. THE DEMO CAMPUS. CSU Stanislaus buildings — the ONE public, CORS-open CA campus footprint
#      service found in a full AGOL sweep.
S=https://services6.arcgis.com/rX5atNlsxFq7LIpv/arcgis/rest/services/Buildings/FeatureServer/0
curl -s "$S?f=json" | jq '{name, geometryType, objectIdField, sr: .extent.spatialReference.latestWkid}'
#  -> Buildings | esriGeometryPolygon | OBJECTID | 2227    <-- NAD83 CA State Plane III, FEET
curl -s "$S/query?where=1=1&returnCountOnly=true&f=json"                    #  -> {"count":72}
curl -s "$S/query?where=1=1&outFields=Type&returnDistinctValues=true&returnGeometry=false&f=json"
#  -> Community Center | Education | General/Residential | Industrial | Medical | Recreation
curl -s "$S/query?where=1=1&returnExtentOnly=true&outSR=4326&f=json"
#  -> -120.86185462834715, 37.522676565224359 -> -120.84965314260177, 37.529120082225063
curl -sD- -o /dev/null -H "Origin: https://example.com" "$S?f=json" | grep -i access-control
#  -> Access-Control-Allow-Origin: *          <-- browser-safe, no proxy needed
curl -s "$S/query?where=1=1&outFields=Build_ID,Name&resultRecordCount=6&returnGeometry=false&f=json"
#  -> 001 Vasche Library · 002 Bizzini Hall · 003 "Centeral Plant"  <-- MISSPELLED AT SOURCE.
#     Name='Central Plant' returns zero rows. Join on Build_ID (a zero-padded STRING: "001").

# ── 4. THE INSTITUTION REGISTRY. IPEDS/HIFLD schema, republished by Cal OES.
H=https://services.arcgis.com/BLN4oKB0N1YSgvY8/arcgis/rest/services/HigherEducation/FeatureServer/0
curl -s "$H/query?where=1=1&returnCountOnly=true&f=json"            #  -> {"count":679}
curl -s -G "$H/query" --data-urlencode "where=STABBR='CA'" --data-urlencode "returnCountOnly=true" --data-urlencode "f=json"
#  -> {"count":679}   (the layer is already CA-only; STABBR is not a useful filter here)
curl -s "$H?f=json" | jq -r '.fields[].name' | head -20
#  -> OBJECTID CONTROL INSTCAT INSTSIZE ICLEVEL HLOFFER GROFFER UGOFFER SECTOR MEDICAL HOSPITAL
#     HBCU TRIBAL LANDGRNT LOCALE OBEREG UNITID INSTNM IALIAS ADDR ...

# ── 5. THE TWO POISONED LAYERS. Do not bind either.
curl -s "https://services.arcgis.com/BLN4oKB0N1YSgvY8/arcgis/rest/services/CaliforniaPublicSchools/FeatureServer/0?f=json" \
  | jq -r '.fields[].name' | head -12
#  -> OBJECTID Loc_name Status Score Match_type Match_addr LongLabel ShortLabel Addr_type Rank ...
#     ^^ that is the output schema of an Esri GEOCODING JOB, not a school register. Someone
#        published a locate result. Same for CaliforniaPrivateSchools. Use CDE instead:
curl -s "https://services3.arcgis.com/fdvHcZVgB2QSRNkL/arcgis/rest/services/SchoolSites2526/FeatureServer/0/query?where=1=1&returnCountOnly=true&f=json"
#  -> {"count":9946}    fields: CDSCode SchoolName DistrictName SchoolType SchoolLevel EnrollTotal ...

# ── 6. THE FLOODPLAIN LAYER-INDEX TRAP.
F=https://services.arcgis.com/BLN4oKB0N1YSgvY8/arcgis/rest/services/100_Year_Floodplain/FeatureServer
curl -s "$F?f=json"   | jq '.layers[] | {id, name}'    #  -> {"id": 4, "name": "100 Year Floodplain"}
curl -s "$F/0?f=json" | jq '{name, geometryType}'      #  -> {"name": null, ...}   <-- NOT "empty layer"
#     The only layer id is 4. Always read the SERVICE root before assuming /0.

# ── 7. CLERY HAS NO API. Confirm before designing a feed that cannot exist.
curl -s -o /dev/null -w "%{http_code}\n" "https://ope.ed.gov/campussafety/Feed/Files/2024/Crime2024.zip"  # -> 404
curl -s -o /dev/null -w "%{http_code}\n" "https://ope.ed.gov/campussafety/api/datafile/list"              # -> 404
#     The Data Analysis Cutting Tool is a session-driven SPA download. Treat Clery statistics as an
#     ANNUAL MANUAL CSV IMPORT joined on OPE_ID (from layer /5). Never render it as live.

# ── 8. OVERPASS IS RATE-LIMITED. Ingest-time fallback only, never a runtime dependency.
curl -s --data-urlencode 'data=[out:json][timeout:50];way["building"](37.523,-120.855,37.537,-120.840);out count;' \
  https://overpass-api.de/api/interpreter        #  -> 1295 ways
#     Third call inside a minute -> HTTP 429. A name-regex query timed out at 52 s. ODbL: attribute.

# ── 9. THE ONE NUMBER YOU CANNOT GET FROM GEOMETRY.
#     Shape__Area on the buildings layer is FOOTPRINT in square feet of EPSG:2227 State Plane.
#     It is NOT gross square feet and it is NOT assignable square feet. A three-storey building is
#     understated by exactly 3x, and ASF/GSF is institution-specific (typ. 0.60-0.65).
#     => ASF comes from the uploaded inventory or the rung renders GREY. There must be no code path
#        that can produce an ASF the inventory did not supply.
```

---

## Guided wizard — **the prompts that assign the app's defaults**

Launch with `/recipe education_campus-operations`. Ask each group, **apply the default so "accept all" builds
a complete app**, confirm a one-line summary, then run §5.

| # | Wizard question | Options → **default** | The default it assigns |
|---|---|---|---|
| 1 · App | Title & as-of date? | free text → **"Campus Operations — the standards rack"**, today | header title + `strata:notes.asOf` |
| 2 · Campus | Which campus? | pick from CSCD `/5` (266 CCC) or `/4` (145 UC/CSU) → **CSU Stanislaus, Turlock** (the one campus with public building footprints) | `definitionExpression` on `campus-land`; `initialState.viewpoint` = the buildings extent |
| 3 · Rulebook | Which space standards? | **CCC — BOG 2020 (Title 5 §57020)** · CSU/DOF profile · UC profile · generic FICM/APPA | the `rungs[]` constants: 20 ASF/STN, 53 (48) hrs, 0.66, 27.5, 0.85, the §57028 TOP table — and the `vintage` string stamped on every exhibit |
| 4 · Campus size | Above or below 140,000 WSCH? | **below → the 48-hour line (63.1 ASF/100 WSCH)** | which branch of §57021 the lecture rung uses |
| 5 · Inventory | Load your space inventory now? | CSV URL / skip → **skip** (rungs render grey with "load your inventory") | `FileDataSource` `rooms`; required columns `BLDG_ID ROOM TOP_CODE SPACE_TYPE ASF STATIONS` |
| 6 · Schedule | Load a term schedule? | CSV URL / skip → **skip** | `FileDataSource` `schedule`; `BLDG_ID ROOM DAYS START END ENROLLED TERM` |
| 7 · Condition | Load a FUSION FCA export? | CSV URL / skip → **skip** | `FileDataSource` `fca`; `BLDG_ID FCI RENEWAL_COST CRV`; the FCI rung's line at **0.10** |
| 8 · Safety | Include the Clery property book? | **yes** · no | page 2; `C_Type` seeded as **suggested**, confirmation queue on |
| 9 · Public band | Public-property band width? | **50 ft** · 100 ft · none | the `analysis` buffer distance, labelled as an approximation |
| 10 · Site risk | Overlay hazard context? | **wildfire + flood + seismic** · none | FHSZ (0–3), floodplain (**layer 4**), CGS zones, at 40/255 alpha |
| 11 · Look | Dark or light? | **dark** (a console) · light (prints, and every exhibit prints light anyway) | `ThemeSpec.mode` + `basemapForTheme` |
| 12 · Map controls | Basemap + layer buttons on the map? | **yes** · zoom only | one control cluster; keyless CARTO basemaps only |

---

## 5. Prompt-script (run in order)

```
A. /new-app — "Campus Operations — the standards rack", Template: open-design.
   Structured ThemeSpec (§2.2), light, primary #0f6e63. App-shell: header (title · campus filter ·
   term filter · loader · page-nav · theme-switch), footer (attribution · share · print),
   splash explaining what runs on public data and what needs an upload. Two pages: `rack`, `book`.
   Register the two app-local widgets (§2.6) with their fallbacks behind a flag.

B. /add-data — the verified spine, all outSR=4326:
     campus-land  CSCD 2025 /4 University_lands           (145)
     campus-ccc   CSCD 2025 /5 CA_Community_Colleges      (266)
     buildings    CSU Stanislaus Buildings/0              (72, SR 2227 -> 4326)
     inst-pts     Cal OES HigherEducation/0               (679)
     fhsz         Fire_Severity_Zones/MapServer 0-3
     flood100     100_Year_Floodplain/FeatureServer/4     <-- layer 4, not 0
     seismic      CGS_Earthquake_Hazard_Zones
     transit      Caltrans CA_HQ_Transit_Stops/0
   Set initialState.viewpoint to the buildings extent (§4 step 3).

C. /symbology + /popup — genuine ESRI drawingInfo/popupInfo on verified fields only.
     buildings   uniqueValue on Type (6 values) for the base draw; classBreaks driven by an Arcade
                 valueExpression bucketing the selected standard's contribution. NO esriSMSPath.
     campus-land uniqueValue on C_Type, fill alpha 40/255, distinct outline for '- HOUSING'.
                 Handle BOTH spellings of the open-space class (Trap T3).
     popupInfo   buildings: Name, Build_ID, Type, Address.  campus-land: NAME, C_Type, Acres, F_Type.

D. /panel — the FileDataSource loaders: rooms, schedule, fca (optional), clery-log (optional).
   Declare required columns; render every dependent rung GREY until the file lands. Never impute ASF.

E. /analyze — buffer campus-land by the wizard's band width -> the public-property band, published as
   an output; bind clery-kpi-pub with dataSource.fromWidget. Label it an approximation, not a
   right-of-way analysis.

F. /panel statistics + /panel chart + /panel table —
   KPIs: Capacity ASF, Load ASF, distance-to-line gauge (thresholds 90/100), acres classified.
   Chart: "Movers" — bar, BLDG_ID x sum(ASF), limit 12.
   Tables: room ledger (10 columns, virtualized) and the ASR property schedule.

G. WIF: author AppLayout.connections — the 32 rows in DESIGN-PROPOSAL.md §5. Shipped emitters ONLY;
   leave the button/window unwired and log it. Verify the signature loop end to end.

H. Controls + composed export: measure, scale, legend, basemapSwitcher; status-bar in EPSG:4326;
   print (board-packet exhibit, with the standard's vintage on it); share deep-link; table CSV as the
   Clery geography appendix.
```

---

## 6. Verify

| Check | Pass |
|---|---|
| Silhouette is `standards-rack`; distinct from all four Education siblings and from every rail elsewhere | ✅ `DESIGN-PROPOSAL.md` §2 anti-collision table |
| ≥3 `connections` fire; the signature loop works end to end | ✅ 32 authored, all on shipped emitters; +5 widgets link via `sourceId` with none |
| Every `layerId` + field verified against the live service (§4) | ✅ 145 · 266 · 72 · 679 · 9,946 · 936 · 3,439, field lists in `DATA.md` |
| No wire uses a pending emitter | ✅ `buttonClick`/`viewChange`/`pageChange`/`mapClick`/`sketchComplete`/`timer` unwired, logged §7 |
| `responsive.small` collapses every side-by-side row | ✅ both splitters → column; panel → `dock:"top"`; KPI trio → `flow-row` |
| Basemap keyless; everything EPSG:4326; `OBJECTID` is the OID | ✅ CARTO/OSM only; `outSR=4326` on every request; OID verified per layer |
| No number is derived from geometry that the inventory must supply | ✅ Trap T12 / §4 step 9 |
| Clery classification rendered as **suggested**, never asserted | ✅ page-2 caveat text + `CONFIRMED_BY` column |
| Standard **vintage** visible in the header and on every export | ✅ `rack.props.vintage` |
| Writes guarded and degrade to read-only | ✅ `updateRecord` behind `assertEsriBackend`; Strata Serve path exports the confirmation queue as CSV |
| Runs on Strata **and** ArcGIS | ✅ reads only; the single write leg degrades as above |
| An inventory belongs to exactly one campus | ✅ switching clears rooms/schedule/condition, the term, the pin and the banner (`test-render.mjs` §11c1) |
| Every campus renders, with or without buildings | ✅ casing + 0.30 fill; 7 institutions measured loading features and moving the camera (§11c2) |
| The map cannot be crushed or left blank | ✅ 240 px floor, one-row site strip, blank-style fallback, hydration on `load`/`styledata`/`idle` (§11e) |
| Every abbreviation is explained in place | ✅ 42 `GLOSS` terms, keyboard-reachable, print-suppressed (§11d) |
| Only keyless basemaps are offered | ✅ the drawer lists three CARTO styles and the suite asserts no keyed provider appears (§11a) |

---

## 7. Harvest (gaps → strata-core)

0. **Two of these are now answered in-app, and stay logged for the registry.** This build ships a real
   `<input type=file>` (issue 1) and its own map control cluster (`basemap` + `layer-panel` equivalents),
   because the shipped widgets could not do either. Both remain open *as registry gaps*.
1. **`add-data` cannot pick a local file.** Its prop is `onAddLayer({url, kind, title})`. `FileDataSource`
   ships (CSV via `parseCsv`, GeoJSON upload) but there is no shipped *widget* for a file input. This recipe
   is an upload-first app; file the issue. **Blocking-ish** — day 1 accepts a URL to the export.
2. **`button` does not emit `buttonClick`.** Blocks the "Read the standard" → `window` `showHide` wire, which
   is the natural home for a citation panel. Manifest §4.1 lists the emitter as pending.
3. **`table` has no `orderBy`/`sort` prop** in the manifest, so "worst FCI first" has to come from a filter
   or from the chart. A declarative default sort would remove a widget from this design.
4. **`chart` has no threshold/reference-line option.** Every chart in this recipe wants the same 2 px line
   the rack draws. A `referenceLine` prop on `SavedChart` would make `standards-rack`'s fallback much closer
   to the real thing.
5. **Harvest candidates:** `standards-rack` (a rail of published thresholds as navigation) and
   **`derivation-card`** (show-your-work as a first-class widget). The latter is wanted by at least three
   other researched recipes that compute a number a regulator can challenge —
   `financial-services_climate-risk-regulatory-reporting`, `environment_agriculture-land-use` (LESA), `transportation_airports-aviation`
   (the §77 margin). Promote per the `APP-TEMPLATE-LIBRARY.md` harvest rule if reused twice.
6. **Catalogue corrections** (proposed text in `DATA.md` §5): add CSCD 2025, CSU Stanislaus buildings, CDE
   `SchoolSites2526` and UCOP `UC Parcels` to `CA-DATA-CATALOG.md` §4; mark
   `CaliforniaPublicSchools`/`CaliforniaPrivateSchools` as geocoder spoil and `EducationHoursTracker_PublicView`
   as a false positive; amend the §5 *Facilities / floor plans* gap row to say **interior** only.

---

## 7a. Presentation & article

| File | |
|---|---|
| `presentation/index.html` | 10-slide tabaqat deck, 16:9, keyboard + click nav, prints one slide per page. Opens on *"Your laboratories look like a surplus"*; the rack, the derivation ledger and the 2020-vs-2010 swap are drawn in the deck's own vocabulary from the app's real output |
| `presentation/linkedin-article.md` | ~1,310-word article, a 177-word teaser, and a **27-row claims note** tracing every figure to the command or document it came from — plus an explicit *"do not claim"* list and three gaps to volunteer if asked |

The article leads on the inversion (§57032 sizes need from scheduled hours, so an idle laboratory
computes as a surplus) and closes on the vintage (86.7 % → 115.3 % from a document date). Both are
asserted in the suite, so the deck and the app cannot drift apart.

## 8. Sources

**Internal** — `../APP-TEMPLATE-LIBRARY.md` · `../DESIGN-CONTEXT.md` · `../DESIGN-REQUEST-PROMPT.md` ·
`strata/recipes/COMPONENT-MANIFEST.md` (§3 registry, §4 triggers/actions, §10 freestyle charter) ·
`strata/docs/guide/app-design.md` · `strata/docs/reference/human-language.md` ·
`../../data_sources/data_sources_ca.md` · `../../data_sources/CA-DATA-CATALOG.md` · `DATA.md` ·
`DESIGN-PROPOSAL.md`.

**Regulation & standards** (fetched/extracted 2026-07-30)
- *Board of Governors of the California Community Colleges — Policy on Utilization and Space Standards*,
  2020 revision, 5 pp. §§57020–57032 quoted verbatim in `DATA.md` §2.4.
- Title 5 CCR §57020 (incorporation by reference); Ed Code §§66700, 70901, **81805**, 81821(e), 81836.
- *California Community Colleges Space Inventory Handbook* (Chancellor's Office; facilities memo FS-21-06).
- CCCCO *Five-Year Capital Outlay* report (2024-25) and the BOG project-scoring criteria.
- FUSION — Foundation for California Community Colleges (>90 M sq ft; Facility Condition Assessment).
- Clery Act, 34 CFR 668.46 — geography categories and the Oct 1 ASR deadline; OPE Campus Safety and Security
  data collection (`ope.ed.gov/campussafety`).
- NCES **FICM**; APPA (FCI); IFMA; BOMA; SCUP; DSA / Field Act; AB 300 (CSU/UC seismic inventory).

**Competitive** (page-verified 2026-07-30)
- Esri — *Smart Campus Operations · GIS for Campus Facilities* (higher-education administrators page);
  *Replacing a Legacy CAFM System with ArcGIS Indoors*; *Migrating to ArcGIS Indoors to Manage University
  Infrastructure* (ArcUser, URI); the Esri Community **Campus Operations** blog and the Campus FM Technology
  Association.
- AssetWorks (AiM / ReADY); Archibus/Eptura; Planon; FM:Systems; IBM TRIRIGA; Accruent; Concept3D;
  MazeMap; Ad Astra; CollegeNET 25Live; Gordian (*Space Utilization: Optimization Strategies for the Campus
  of the Future*); Occuspace (2026 campus-underutilization report, via Business Wire).

**Data** — GreenInfo Network *California School Campus Database 2025*; CSU Stanislaus campus buildings;
Cal OES *HigherEducation* / *Universities and Colleges in California*; CA Dept. of Education 2025-26 school,
district-site and district-area layers; UCOP *UC Land Parcels*; CAL FIRE/OSFM FHSZ; Cal OES floodplains;
CA Geological Survey earthquake hazard zones; Caltrans HQ transit stops; OpenStreetMap/Overpass (ODbL).
All endpoints, counts and field lists: **`DATA.md`**.

---

## Modernization (parity release)

> Applies the Experience-Builder-parity features (Phases 1–7). See `strata/recipes/MODERNIZATION.md` +
> `COMPONENT-MANIFEST.md` §8.

- **Structured theme** — the whole look from one `primary` hex, with `success`/`warning`/`danger` carrying
  real semantics (past / near / short of the line) and a `kpi` override for tabular monospace.
- **App-shell** — `header` (title · campus · term · loader · nav · theme · language), `footer`
  (attribution · share · print), and a `splash` that says plainly which rungs run on public data and which
  need a file.
- **DataSource linking** — `sourceId: "rooms"` links `kpi-cap`, `kpi-load`, `gauge-line`, `rm-info` and
  `ledger` with **zero** `connections`; `fromWidget` feeds the public-property KPI from the `analysis`
  output. Live `kpi.stat` / `gauge.stat` throughout.
- **`FileDataSource`** — CSV upload is the recipe's entire interior (rooms, schedule, FCA, incident log).
  This is the flagship use of the Phase-7 data-source kinds.
- **Layout nodes** — `splitter` (rail ⇄ board, board ⇄ ledger), `panel` (the rack, `dock:"top"` on phones),
  `accordion` (the three books), `window` (the standard's text, declared and honestly unwired).
- **`analysis` widget** — the `buffer` tool publishing its result as an output another widget consumes.
- **Deliberate abstentions** — no `animate`, no `autoPlay`, no time slider, no `RestDataSource`/
  `StreamDataSource`. A console that computes a number a regulator will challenge earns trust by holding
  still.
