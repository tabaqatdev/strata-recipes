# Recipe — Agriculture & Land Use: **The Ledger** (Natural Resources & Environment)

> **Researched recipe.** Replaces the 2026-07-22 `time-player` scaffold (archived as
> `RECIPE-v1-time-player.md`). §1 Study is page-verified; §3 Data sources and §4 Verify are
> curl-checked live on **2026-07-27** — every field name, count and acreage below was measured from a
> real response, not written from memory. The design rationale, the three candidate silhouettes and
> the capability sweep are in **`DESIGN-PROPOSAL.md`**; this file is the build spec.

*Pick a place and a pair of survey cycles: read the ledger of what its farmland became — acre by
acre, class by class — and score what the next project would cost.*

---

## 0. Provenance

| | |
|---|---|
| **Source** | `https://tabaqat.net/solutions` — **Natural Resources & Environment** section |
| **Name on site** | Agriculture & Land Use |
| **Tagline on site** | "Crop, parcel and land-use mapping with condition and change over time" |
| **Scaffolded** | 2026-07-22 (`time-player`) |
| **Researched & redesigned** | 2026-07-27 |
| **Template** | **`open-design` — "ledger-book"** (the `time-player` assignment is released back to the sector) |
| **Data catalog** | `agland-catalog-ca.json` — 2,339 CA services, 348 role-tagged across 10 roles, + 24 external feeds (20 verified-live) |

**Why the template changed.** The tagline's "change over time" reads as a time player, and a time
player is the one silhouette this data cannot honestly carry: DWR's `Idle` class did not exist before
2021, so an animation would show idle farmland going from 0 to 813,945 acres in a single frame. The
axis is real; the continuity is not. The redesign keeps the axis and marks the breaks. Full
reasoning: `DESIGN-PROPOSAL.md` §2.

---

## 0.1 The working app in this folder

`app/` is a **built, running demo** of this design — self-contained, keyless, live.

```bash
cd app && python3 server.py        # http://localhost:8046   (or double-click START-APP.command)
node test-ledger.mjs               # 103 live-data assertions — all green 2026-07-27
```

| File | What it is |
|---|---|
| `app/index.html` | The whole app: MapLibre 4.7.1 + keyless CARTO raster, no build step, no key, no proxy |
| `app/server.py` | Zero-dependency static server (MapLibre cannot run from `file://`) |
| `app/START-APP.command` | Double-click launcher |
| `app/test-ledger.mjs` | **103-assertion live regression suite** — extracts the shipped tables/functions out of `index.html` and checks them against real DWR, DOC and USDA responses |
| `app/agland-catalog-ca.json` | The catalog, alongside the app |
| `presentation/index.html` | **10-slide deck** (tabaqat-branded, 16:9, arrow-key / click / `f`-for-fullscreen). Serve it: `python3 -m http.server 8047 --directory presentation` |
| `presentation/linkedin-article.md` | **~800-word LinkedIn article** + teaser post + a claims note sourcing every figure |

**What it does.** Pick a county and two survey cycles (the ribbon, or the explicit From/To pickers —
**both endpoints are independently settable**); the transition matrix fills with **real per-field
acreage**; click a cell and the map, the balance, the field table **and the KPI row** narrow to
exactly those acres; **click the same cell again to release the filter**; click a field and the
**California LESA Model** is computed live from USDA SSURGO, the survey's own ¼-mile Zone of
Influence, and the 2025 Williamson Act roll — every factor printing its source and its branch.

**Its chrome.** Launches **dark** (`theme-switch` → the light printable mode, each paired to a keyless
basemap). The **layer list** and the **basemap gallery** sit in one control group directly above the
zoom cluster, dimensioned and chromed to match it (29 px, 4 px radius, the same ring, a 1 px divider)
with ESRI/Calcite glyphs — MapLibre's own zoom, scale and attribution are themed to match. The layer
list is the **single source of truth for visibility**; the cell-isolate writes the same state.

**Measured on 2026-07-27, Madera County 2018 → 2023** (10,229 / 10,389 fields, 9,437 matched, **7.6 s**):

```
              D        C        V       YP        G        F        T        P       IX
D             ·       76      272      606      534      460    1,954      115    4,431
V         4,022      113        ·      367      457        –      357      389    2,338
YP        5,198      662      621        ·        –        –      116       15      121
P           829       55      195        –      782    4,865    1,056        ·      618
IX        3,506      190      192        –    1,335        4       59    1,016        ·
   changed class 56,242 ac · out of active ag −11,274 · into active ag +6,303 · NET −4,971 ac
```

That is the business in one screen: **4,022 acres of vineyard replanted to orchard**, **5,198 acres of
2018 young perennial now bearing**, **4,431 acres of orchard pulled to idle** — and **3,506 acres of
idle coming back**. Madera's new plantings fell from 7,200 ac (2018) to 1,315 ac (2023).

**The LESA leg reproduces the §4.10 filed study exactly: 45.2 → band 40–59 → SA 13.5 < 20 → not
significant.** A live screening (SSURGO point → LCC + Storie → weighted score) returns in **~0.7 s**.

## 1. Study — how the market frames this

**The question the buyer asks.** *"How many acres of farmland did we lose here, from what to what —
and is this next project's impact significant?"* A county planner has to write that number in a staff
report; a CEQA consultant has to defend it at a hearing; a Groundwater Sustainability Agency has to
report it in a GSP; a land trust has to target the next easement with it.

**Reference solutions (benchmark + coexist, never copy).**

| Product | What it actually is | Where it stops |
|---|---|---|
| **DOC California Important Farmland Finder (CIFF)** — `maps.conservation.ca.gov/dlrp/ciff/` | The state's own front end to the statutory record, backed by four **asynchronous** GPServers (`CIFFAnalysisTool`, `CIFFPolygon`, `CIFFPointBuffer`, `CIFFPolygonBuffer`) | Draw a polygon, submit a job, wait, read a table of *what is inside this shape now*. No from→to accounting, no cycle pair, no score |
| **Esri ArcGIS Solutions — Agricultural Parcel Analysis** | The Esri incumbent for this vertical: four apps (Experience Builder app, Field Maps mobile map, ArcGIS Notebook, Agricultural Review Request) built for **local assessors valuing agricultural parcels** over parcels, appraisal units and USDA soil data | It is an **appraisal** workflow — visualising parcel characteristics and soil composition to support a valuation and an exemption request. It does not do conversion accounting or CEQA significance |
| **Esri / Impact Observatory Sentinel-2 10m Land Cover Explorer** | Global annual LULC 2017–present, 9 classes, beautiful | "Crops" is one undifferentiated bucket. It can say *was this cropped*; never *what crop, on what irrigation, planted when* |
| **Climate FieldView · John Deere Operations Center · xarvio · OneSoil · EOS Crop Monitoring · Taranis · Trimble Ag** | The agtech field: in-season condition, scouting, variable rate, yield, for an **operator on land they already farm**. xarvio alone serves ~130,000 farmers over 20M ha off Sentinel-2 | None of them are public-agency tools and none of them touch a statutory record. They answer *how is my crop doing*, not *how many acres left agriculture* |
| **Regrid · AcreValue** | Parcels (156M+ US/Canada records) and farmland value / soil productivity / crop mix | Commercial, parcel-and-price framed. Regrid is the honest answer to our missing parcel leg — as a paid input, not a competitor |
| **MapBiomas** (Brazil, + Amazonia/Chaco/Pampa) | **The nearest idiom in the world**: annual land-use/land-cover series on Google Earth Engine at 30 m, with a **transition matrix** detailing from→to change activities, maps and algorithms public | Regional to South America. Nobody has applied the transition-matrix idiom to California's biennial statutory farmland record |
| **Global Forest Watch** (WRI / GLAD) | Open-source global forest-change monitoring, satellite + AI | Forest, not farmland; loss, not classified conversion |
| **American Farmland Trust — *Farms Under Threat*** | The advocacy analysis: 465,900 CA acres converted; up to **797,358 acres** projected lost by 2040, 67% of it on the best land; Riverside, San Bernardino and Fresno in the national top 20 | A report and a static hub, published on a multi-year cycle — not an operational tool for the next agenda packet |

**Our edge.** AI-authored, MIT/open, runs on **Strata *or* ArcGIS**, sovereign/on-prem,
cross-widget interactivity. Concretely, three things nobody else does together: (1) **the ledger** —
from→to acreage between two chosen official vintages, as a navigable matrix; (2) **the
comparability ribbon** — the methodology breaks in the official record marked and excluded by
construction; (3) **the LESA score computed live** from published inputs, with every factor's source
printed beside its points. Coexistence, never "replace ArcGIS": CIFF stays the system of record and
the app hands off to it.

**Standards, specifications & organizations to speak fluently.**

- **Gov. Code §65570** — mandates DOC to report farmland conversion **biennially on even years** and
  maintain the Important Farmland Maps; the reason a 20-vintage series exists at all.
- **The California Agricultural LESA Model** — created by **SB 850 (Ch. 812, Statutes of 1993)**,
  published as the **CDC 1997 Instruction Manual**; the optional-but-widely-adopted methodology for
  CEQA agricultural significance. Weights and thresholds in §2.3 below.
- **CEQA** — Guidelines **§15126.2(a)** (identify significant effects), **§15360** (environment),
  **§21060.1(a)** (agricultural land = **Prime + Statewide Importance + Unique**), **§15370**
  (mitigation — the basis for agricultural conservation easements as CEQA mitigation).
- **The Williamson Act** (California Land Conservation Act, 1965) — 10-year rolling contracts,
  9-year non-renewal wind-down, plus **Farmland Security Zones** (20-year). 15.6M acres enrolled.
- **SGMA** (2014) — GSAs, GSPs, critically overdrafted subbasins; the force behind the idle acreage.
- **DOC Multibenefit Land Repurposing Program (MLRP)** — AB 252; 23 approved implementation projects
  (13 in 2025) over **4,800+ acres** across six block-grant regions (Pajaro Valley, Madera, Merced,
  Tule, Turlock, Kaweah); $200M in the 2024 Prop 4 climate bond, $32M for FY2025-26.
- **FMMP classification** (Prime / Statewide Importance / Unique / Local Importance / Grazing), derived
  from **USDA land inventory and monitoring criteria as modified for California**.
- **USDA NRCS** — SSURGO, **Land Capability Classification**, **Web Soil Survey**; the **California
  Revised Storie Index** (UC Div. of Agricultural Sciences, Special Publication 3203, 1978).
- **Land IQ / DWR Statewide Crop Mapping** — field-scale, random-forest supervised classification
  against ground reference, contracted to exceed 95% accuracy and reporting ~98% since 2018
  (96.5% on the Land IQ legend / 98.3% on the DWR legend for WY2018).
- **PPIC** — the San Joaquin Valley land-transition literature: up to **900,000 acres** (20% of
  irrigated land) contracting under SGMA, ~500,000 in the best case; SGMA-ready crops and
  land-repurposing as the alternatives to bare fallow.

**Honest scope — what this is *not*.**

- **Not a determination of significance.** It computes the LESA score and reports the CDC 1997 Table 9
  threshold band. The lead agency determines significance; the app hands off with the packet.
- **Not a system of record.** CIFF and the county assessor are the record. This is an accounting and
  screening surface over published data.
- **Not observation.** Nothing here is sensed. Every number traces to a published survey with a
  vintage. The fastest input in the whole app is biennial.
- **Not an in-season crop product.** No NDVI, no yield, no condition. That is the agtech field's
  business and we do not enter it.
- **Approximated, and labelled as such:** the `I + X` reconciliation of the idle series (§4.2); the
  Surrounding Protected Resource Land factor, buffered from 321 easement **points** (§4.6); the
  Water Resources Availability branch, where the published data does not carry drought restrictions;
  and the Project Size factor wherever no parcel source is available (§4.7).

## 2. UI design spec (front-loaded)

### 2.1 Layout (Template: **`open-design` — "ledger-book"**)

- **Template `open-design`** under the freestyle charter (`strata/recipes/COMPONENT-MANIFEST.md` §10):
  registry widgets and manifest config keys only, with the §10.2 escape hatch for the three new
  widgets in §2.6, each with a named day-1 fallback. **Never fall back to `split-dashboard`
  silently** — if a piece cannot be built, keep the ledger silhouette and stub with `placeholder`.
- **Why this silhouette:** the buyer's question is an acreage between two dates, so the answer is a
  double-entry account, not a pair of choropleths. A matrix also makes the methodology break visible
  *per cell*, which is the only thing that makes a 38-year series defensible in a hearing.
- **Anti-collision (Natural Resources & Environment):** not `compare-swipe` (forestry-vegetation),
  not `zone-lookup` (mining-extraction), not `scoreboard` (water-resources), not `sidebar-explorer`
  (conservation-protected-areas), not `chart-board` (climate-hazard). No sibling uses a **matrix grid
  as navigation** — that, not the palette, is the differentiator. (The paper design also leaned on
  shipping light; the build inverted the theme, so the silhouette carries the distinction alone.)
- **Signature loop:** *comparable cycle pair → matrix fills with measured acres → click a cell → map,
  table, class-mix and balance all narrow to exactly those acres → open one polygon → LESA score
  computes from live SSURGO/FMMP/Williamson/district data → packet exports with a source beside every
  figure.*
- **Wiring floor:** ≥3 live `connections` on first render (`docs/guide/app-design.md` §3). This
  design ships **27** (§2.7).

**Pages.** Three behind a `page-nav`; **page 1 alone answers the purpose sentence.**

1. **`ledger` (fixed)** — `splitter` `orientation:"h"` `sizes:[32,68]`:
   `panel`(dock `left`, width 420, title "The Ledger") holding `ribbon` → `ledger` → `balance`;
   and a `section` `mode:"fixed"` holding the `map` with `layer-panel` + `basemap` as map-anchored
   controls above the zoom cluster. Below the splitter, a `row` with the four `kpi`s, then a `row`
   with `celltable` | `classmix`. A `window id:"scorecard"` (`open:false`, `modal:true`) hosts the
   LESA drill-in. `responsive.small` collapses the splitter to a `column` and the bottom rows to an
   `accordion`.
2. **`scorecard` (fixed)** — `splitter` [`panel`(factor rail) | map]: `soilquery` (`query`),
   `zoi` (`analysis`), `scorecard` (`lesa-scorecard`), `lesagauge`, `lesaverdict`, `factortable`,
   `factorcard` (`feature-info`).
3. **`field` (fixed)** — a `views` node (`nav:"tabs"`, `mapState` per view): *Crop class* ·
   *Irrigation* · *Idle + unclassified* · *Perennial age*, with `cropcarto` and `croptable`.

`splash` (`once:true`) carries the scope disclaimer. ASCII skeleton: `DESIGN-PROPOSAL.md` §3.

### 2.2 Theme

```jsonc
{ "mode": "dark",                       // the build inverted the paper choice — see §0.1 / proposal §9a.6
  "colors": { "primary":   "#7bb36a",   // field green, lifted for a dark ground
              "secondary": "#c08a52",   // soil / ledger ink — also the matrix magnitude ramp
              "success":   "#5cc08a",   // acres IN  (credit)
              "danger":    "#e0705c",   // acres OUT (debit)
              "warning":   "#e0b33c",   // idle + unclassified — the uncertain column
              "info":      "#4a6f8a", "light": "#e8e6df", "dark": "#14171a" },
  "fonts": { "scale": "compact" },
  "variables": { "--strata-radius-md": "3px", "--strata-elevation-1": "none" } }
```
Light mode is the same six roles at their paper values: `primary #3d6b35 · secondary #8c5a2b ·
success #2f7a4f · danger #a33a2a · warning #b8860b · light #f7f5ef · dark #23261f`.

**A ledger, not a dashboard — and it launches dark.** The paper design launched light; driving it
inverted that. On screen the matrix and the balance read far better on a dark ground, and light's real
job is **print and the public counter**, not the working session. Dark (CARTO Dark Matter) is the
default; `theme-switch` flips to the light "ledger paper" mode (CARTO Positron on `#f7f5ef`). Both
keep the ledger's grammar: near-square corners, no card shadows, hairline rules, tabular mono on every
figure so the matrix columns align down the page.

- **Colour only where a sign exists.** Off-diagonal cells shade on a **sequential magnitude ramp** in
  `secondary`; the **diverging red↔green** lives on the Δ net column and the balance line. A diverging
  ramp on a cell was a paper error — nothing "goes negative" from Vineyard to Orchard.
- Ramps from `@strata/theme/palettes` so they survive colour-blind review in both modes.
- Basemaps keyless and OSM-derived (Dark Matter · Positron · Voyager · OpenStreetMap · OpenTopoMap);
  the theme pairs one to the mode, an explicit pick from the gallery overrides that pairing.
- **MapLibre paint cannot read CSS variables** — re-paint ink-coloured layers on every light↔dark
  flip or they vanish (carried over from the aviation build; asserted in the suite).
- **Theme MapLibre's own controls too.** Left alone, the zoom cluster stays white on the dark ground
  and the map controls stop reading as one column.
- Polygon fills at ~40/255 so farmland class, subbasin and Williamson layers stay legible stacked.
- **EN in the California instance** (the output quotes US regulatory text); **AR/RTL is scoped as a
  separate Gulf build** with its own regulatory copy and classification scheme — the same business
  (irrigated acreage retired under a water mandate), not a language toggle. See `DESIGN-PROPOSAL.md` §6.

### 2.3 KPI cards

Four live `kpi`s, tabular numerals, threshold-coloured, all bound to a shared source or
`fromWidget:"ledger"` so they recompute on filter/selection with **no `connections`**:

| id | label | stat | measured statewide value (2026-07-27) |
|---|---|---|---|
| `kpi-lost` | Prime → Urban, this cycle pair | `{field:"polygon_ac", op:"sum"}` on the differenced set | computed per pair |
| `kpi-idle` | Idle + unclassified | `{field:"ACRES", op:"sum"}` where `SYMB_CLASS IN ('I','X')` | **1,461,915 ac (2022)** · 1,083,212 (2023) · 1,121,576 (2024) |
| `kpi-new` | New plantings (young perennial) | `{field:"ACRES", op:"sum"}` where `SYMB_CLASS='YP'` | **74,203 ac (2024)**, down from 228,346 (2016) — **−67%** |
| `kpi-nr` | Williamson Act in non-renewal | `{field:"Acreage", op:"sum"}` where `WA_Type='Nonrenewal'` | **155,080 ac** across 1,560 contracts (2025) |

Plus `kpi-inview` (converted acres in the current extent), driven by
`map extentChange → showStatistics`, and `lesaverdict` on the scorecard page (status-coloured:
`ok` < 40, `warn` 40–79, `critical` ≥ 80).

**The strip is scoped, not static** (added in the build — proposal §9a.8). With nothing selected it
reads county-wide, as above. The moment a matrix cell is clicked it **re-labels and recomputes for
that transition** — *acres in this transition · share of all changed acres · fields · median field
size · dominant irrigation* — and marks itself scoped with an accent rule. Releasing the cell reverts
it. Five county totals sitting above a filtered matrix are worse than useless: they invite the reader
to attribute them to the selection.

`lesagauge` is a `gauge` with `stat` on the LESA score, `thresholds: [40, 80]` — % of the
automatic-significance ceiling consumed.

### 2.4 Charts & table

- **`classmix`** — `stacked-bar` (`horizontal:true`), one segment per farmland class in the selected
  cell; emits `categorySelect` → cross-filters map + table.
- **`sizedist`** — `chart` `kind:"histogram"` on `polygon_ac` within the cell: how a planner tells
  incremental subdivision from one large project.
- **`celltable`** — server-paged `AttributeTablePanel` on the differenced set. Real columns:
  `County`, `polygon_ac`, `PriorValue`, `polygon_ty`, `YrChngd`, `FirstMap`, `upd_year`. Row →
  `zoomTo` + `flash` + `viewInTable`; CSV/GeoJSON export.
- **`factortable`** (scorecard) — columns `factor`, `rawValue`, `points`, `weight`, `weighted`,
  `source`, `branch`. The `source` column is not decoration; it is the reason the number is
  defensible.
- **`croptable`** (field bench) — real columns from DWR: `COUNTY`, `SYMB_CLASS`, `MAIN_CROP`,
  `IRR_TYP1PA`, `YR_PLANTED`, `ACRES`, `HYDRO_RGN`, `MULTIUSE`, `DataStatus`.
- **`cropcarto`** — `carto` with `category` widgets on `SYMB_CLASS` and `IRR_TYP1PA`, `histogram` on
  `YR_PLANTED`.

### 2.5 Capabilities to use (Phases 0–7)

Full sweep with the deliberate non-uses: `DESIGN-PROPOSAL.md` §8. The concrete moves:

- **`/analyze`** — `overlay` (difference two FMMP vintages → the matrix), `buffer` (¼-mile Zone of
  Influence for the LESA SA factors), `aggregate` (acreage rollups by class/county/subbasin),
  `pointsWithin` (easement points inside the ZOI).
- **WIF `connections`** — the 27 in §2.7.
- **Panels/widgets** — `splitter`/`panel`/`window`/`views`/`accordion`; `table`, `carto`, `chart`,
  `filter`, `query`, `feature-info`, `data-actions`, `kpi`, `gauge`, `stacked-bar`, `analysis`,
  `weighted-overlay`, `near-me`, `add-data`, `draw`, `measure`, `bookmarks`, `share`, `page-nav`,
  `theme-switch`, `basemap`, `layer-panel`, `legend`, `status-bar`.
- **Plugins** — `@strata/plugin-search` (address/APN → site, Nominatim), `@strata/plugin-statusbar`.
  **Not** `@strata/plugin-routing` (no travel-time question) and **not**
  `@strata/plugin-timeslider` (§2.1 — the central refusal).
- **DataSource** — `RestDataSource` for `agland-catalog-ca.json` and the USDA SDA POST;
  `FileDataSource` to upload a project boundary or a batch site list. **No `StreamDataSource`** —
  nothing here is real-time and a live badge would be a lie.
- **Composed export** — the **staff-report packet** (PDF: map + legend + scalebar + north arrow +
  matrix + balance + a source line per figure) and an **atlas** (one page per parcel in a batch).
- **Modernization (`COMPONENT-MANIFEST.md` §8)** — structured `ThemeSpec`, app-shell
  (`header`/`footer`/`splash`), `dataSource.sourceId`/`fromWidget` linking, `animate` + `stagger`.

### 2.6 The three new widgets (§10.2/§10.3) and their day-1 fallbacks

Full blocks (props, triggers, actions, dataSource) in `DESIGN-PROPOSAL.md` §9. Summary:

| key | purpose | emits | **day-1 fallback (ships without any new code)** |
|---|---|---|---|
| `transition-matrix` | from-class × to-class acreage grid between two cycles; diverging signed-acres ramp; every cell selectable | `categorySelect`, `recordsChange` | Precompute the matrix with `/analyze overlay`, publish as a small JSON source, render as **`carto` category rows + a `stacked-bar` per row + a `table`** for the full grid + a `sparkline` per row. Loses the grid and the ramp; keeps every number and the cell→map isolate |
| `lesa-scorecard` | the six statutory factors, their weights, LE/SA subtotals, final score, Table 9 verdict, each with its source | `recordsChange`, `categorySelect`, `featureSelect` | **`weighted-overlay`** — whose `criteria {field,label,weight}[]` contract *is* the LESA structure — seeded with the six statutory weights, plus `gauge` + `kpi` + `table`. Loses the automatic 40–59 / 60–79 sub-threshold logic (read off the table); keeps every number and source |
| `comparability-ribbon` | the vintage axis with breaks marked and their evidence; the cycle-pair picker that refuses invalid pairs | `categorySelect` | Horizontal **`stacked-bar`** across the vintage axis (comparable = `primary`, breaks = `warning`) + a `text` callout naming each break + a `filter` as the picker. Loses click-to-pick and the guard; keeps the disclosure |

All three obey §10.2: app-local `registry` override, `--strata-*` tokens only, logical CSS
properties, map driven through the store, data via `dataSource`, cross-widget behaviour in
`connections`.

### 2.7 `connections` (27 — every wire uses a shipped emitter)

Full table with the user-visible behaviour per row: `DESIGN-PROPOSAL.md` §5. Load-bearing set:

```jsonc
{ "from":"ledger", "trigger":"categorySelect", "to":"map",       "action":"filter",   "options":{"layerId":"fmmp-current"} },
{ "from":"ledger", "trigger":"categorySelect", "to":"celltable", "action":"filter" },
{ "from":"ledger", "trigger":"categorySelect", "to":"classmix",  "action":"filter" },
// the cell is a TOGGLE — a second click on the selected cell releases the filter everywhere
{ "from":"ledger", "trigger":"clear",          "to":"map",       "action":"filter",   "options":{"where":null} },
{ "from":"ledger", "trigger":"clear",          "to":"celltable", "action":"filter",   "options":{"where":null} },
{ "from":"ledger", "trigger":"clear",          "to":"classmix",  "action":"filter",   "options":{"where":null} },
{ "from":"ledger", "trigger":"clear",          "to":"scorecard", "action":"showHide", "options":{"hidden":true} },
{ "from":"ribbon", "trigger":"categorySelect", "to":"ledger",    "action":"filter" },
{ "from":"ribbon", "trigger":"categorySelect",                   "action":"setUrlParam", "options":{"param":"cycles"} },
{ "from":"ribbon", "trigger":"categorySelect", "to":"methodcard","action":"viewInTable" },
{ "from":"regionfilter","trigger":"filterChange","to":"map",     "action":"filter" },
{ "from":"regionfilter","trigger":"filterChange","to":"ledger",  "action":"filter" },
{ "from":"regionfilter","trigger":"filterChange",                "action":"setUrlParam", "options":{"param":"region"} },
{ "from":"celltable","trigger":"rowSelect","to":"map",      "action":"zoomTo","options":{"layerId":"fmmp-current"} },
{ "from":"celltable","trigger":"rowSelect","to":"map",      "action":"flash", "options":{"layerId":"fmmp-current"} },
{ "from":"celltable","trigger":"rowSelect","to":"sitecard", "action":"viewInTable" },
{ "from":"celltable","trigger":"rowSelect","to":"scorecard","action":"showHide","options":{"hidden":false} },
{ "from":"map","trigger":"featureSelect","to":"sitecard",  "action":"viewInTable" },
{ "from":"map","trigger":"extentChange", "to":"kpi-inview","action":"showStatistics" },
{ "from":"classmix","trigger":"categorySelect","to":"map",      "action":"filter" },
{ "from":"classmix","trigger":"categorySelect","to":"celltable","action":"filter" },
{ "from":"scorecard","trigger":"categorySelect","to":"factorcard","action":"viewInTable" },
{ "from":"scorecard","trigger":"featureSelect", "to":"map",       "action":"zoomTo" },
{ "from":"croptable","trigger":"rowSelect","to":"fieldmap","action":"zoomTo","options":{"layerId":"dwr-crop-2023"} },
{ "from":"cropcarto","trigger":"categorySelect","to":"fieldmap", "action":"filter" },
{ "from":"cropcarto","trigger":"categorySelect","to":"croptable","action":"filter" }
```

Plus four **zero-connection** source links (`dataSource.fromWidget`), which are the point of the
design: `balance` + `kpi-lost` + `kpi-idle` + `kpi-new` + `kpi-nr` ← `ledger`; `lesagauge` +
`lesaverdict` + `factortable` ← `scorecard`; `scorecard` ← `soilquery` (LE half) and ← `zoi`
(SA surround factors). Because `ledger`'s publish carries the **selection**, the KPI row re-labels and
recomputes for the selected cell through that same binding — no extra wire (§2.3).

**Cycle picking is two-ended.** `ribbon` alternates which endpoint the next click sets (and says
which), with a `from`/`to` pair of `filter` widgets in the header doing the same thing unambiguously.
The first build only ever reassigned the earlier endpoint, which made the TO cycle look frozen —
don't repeat that.

## 3. Data sources

**Read this first.** The crawled `data_sources_ca.md` is an emergency/water/transport inventory
(CAL FIRE 1,187 + Cal OES 993 + Caltrans + DWR). It contains **no statewide crop layer and no
farmland-class layer**: the only service in all 2,339 whose name contains "Agricultural" is a
Caltrans border pest-inspection station list. Both spine datasets are **external**, and one of them
has **moved since the crawl**. The role-tagging below is therefore *context around an externally
supplied core* — see `agland-catalog-ca.json` `meta.coverage_warning`.

| Role | From `data_sources_ca.md` (context) | External (the spine) |
|---|---|---|
| **cropland** | 1 (`CHhighway/Agricultural_Inspection_Stations`) | **DWR Statewide Crop Mapping 2014–2024** (Land IQ) · USDA CDL/CropScape · NASS Quick Stats · Esri/IO Sentinel-2 LULC |
| **farmland-class** | **0** | **DOC Important Farmland 1984–2022** (20 biennial vintages) · `urbanbuiltup` conversion ledger · CIFF GPServers |
| **protection** | **0** | DOC Williamson Act 2021–2025 · DOC Ag Conservation Easements · DOC Ag Planning Grants/SALC · MLRP (no service yet) |
| **soil** | 3 (NEHRP soils, soil texture) | **USDA NRCS Soil Data Access** (LCC + CA Revised Storie Index) · ⚠ Esri Living Atlas soils = **token required** |
| **parcel-cadastre** | 60 (Cal OES fire-response parcel clips; DWR project parcels) | CA Geoportal `UCD_Parcels` (**query disabled**, `PARNO` only) · county assessors · Regrid (paid) |
| **irrigation-water** | 42 (B118 basins, GSAs, GSP areas, critically overdrafted, water districts, well completions, subsidence) | — (the crawl covers this role well) |
| **urbanization** | 7 (`Adjusted_Urban_Area`, urban boundaries) | USGS/Esri Annual NLCD Land Cover |
| **ag-hazard** | 47 (FHSZ, floodplain, burn severity) | — |
| **vegetation-habitat** | 14 (wetlands, habitat, salmon) | CAL FIRE `California_Vegetation_WHR_TYPE` |
| **admin-context** | 160 (counties, boundaries, districts) | — |
| **ag-infrastructure** | 3 | — |
| **climate-context** | 11 | — |
| | **348 role-tagged of 2,339** | **24 feeds, 20 verified-live** |

Licensing/attribution: DOC, DWR and Caltrans services are public-domain California open data
(attribute the agency); USDA SDA and NASS are US public domain; Esri Living Atlas Sentinel-2 LULC
carries Impact Observatory attribution. Everything is reprojected to **EPSG:4326** on ingest.
Full per-feed URLs, field lists and gotchas: `agland-catalog-ca.json` `external[]`.

## 4. Verify each URL first (terminal)

Everything below was run on **2026-07-27**; the `->` lines are the actual measured responses.

```bash
# ─────────────────────────────────────────────────────────────────────────────
# 4.1 THE FIRST TRAP: the DWR crop-mapping URLs MOVED. Every search engine still
#     indexes gis.water.ca.gov/.../Planning/i15_Crop_Mapping_20xx. That folder is gone:
curl -s "https://gis.water.ca.gov/arcgis/rest/services/Planning?f=json" | jq '.services[].name'
#  -> Planning/i15_Parcels_Assessor_Lightbox, i15_Parcels_CVFPB_public, i15_Parcels_SWP_public
#     THREE PARCEL SERVICES. No crop mapping. Resolve the live URL from the AGOL item id instead:
curl -s "https://www.arcgis.com/sharing/rest/search?q=i15_Crop_Mapping&num=25&f=json" | jq -r '.results[]|"\(.title)\t\(.url)"'
#  -> 2014 369be3bf… 2016 11ba0b96… 2018 83f2f377… 2019 1de46194… 2020 576e483b…
#     2021 5fe15fbb… 2022 8b0555ad… 2023 d94e891e… 2024_Provisional 39b63601…
#     https://utility.arcgis.com/usrsvcs/servers/<itemid>/rest/services/Planning/i15_Crop_Mapping_<yr>/MapServer/0

CROP=https://utility.arcgis.com/usrsvcs/servers/d94e891e00364e49a2ed9e9e2e27837d/rest/services/Planning/i15_Crop_Mapping_2023/MapServer/0
curl -s "$CROP/query?where=1%3D1&returnCountOnly=true&f=json"
#  -> {"count":446914}                     2023 FINAL, statewide, field scale
curl -s -o /dev/null -D - -H "Origin: https://example.com" "$CROP?f=json" | grep -i access-control
#  -> Access-Control-Allow-Origin: https://example.com   (CORS-open, keyless, anonymous)

# ─────────────────────────────────────────────────────────────────────────────
# 4.2 THE SECOND TRAP: CLASS1 is '**' on 94% of rows. Do NOT class on it.
curl -s -G "$CROP/query" --data-urlencode "where=1=1" --data-urlencode "outFields=*" \
  --data-urlencode "resultRecordCount=1" --data-urlencode "returnGeometry=false" --data-urlencode "f=json"
#  -> SYMB_CLASS 'T' | CLASS1 '**' | SUBCLASS1 '**' | CLASS2 ' T' | SUBCLASS2 '18'
#     MAIN_CROP 'T18' | ACRES 4.21 | COUNTY 'Sonoma' | UCF_ATT 'S********* T18***00…'
#     CLASS1='**' on 419,240 of 446,914 rows. CLASS2 is SPACE-PADDED (' T'), so TRIM before matching.
#     Class on SYMB_CLASS. Crop code is MAIN_CROP.

# NB: there is no Shape_Area column (it is `Shape.STArea()`); aggregate on ACRES or the groupBy 400s.
curl -s -G "$CROP/query" --data-urlencode "where=1=1" \
  --data-urlencode "groupByFieldsForStatistics=SYMB_CLASS" \
  --data-urlencode 'outStatistics=[{"statisticType":"sum","onStatisticField":"ACRES","outStatisticFieldName":"ac"}]' \
  --data-urlencode "f=json"
#  -> U 4,832,077 · D 2,842,265 · P 1,503,897 · T 896,171 · F 830,460 · I 756,501 · V 750,747
#     G 721,745 · R 529,386 · C 460,989 · X 326,711 · UL 86,004 · YP 60,319   (acres, 2023)

# ─────────────────────────────────────────────────────────────────────────────
# 4.3 THE THIRD TRAP — the most dangerous one: a METHODOLOGY BREAK in the series.
#     Run the same rollup across every year and the Idle class does this:
#        year   I          X          I+X
#        2016          7    948,674     948,680
#        2018         40    994,619     994,659
#        2019          0    818,242     818,242     <- class ABSENT
#        2020          0  1,102,574   1,102,574     <- class ABSENT
#        2021    813,945    600,834   1,414,779     <- class INTRODUCED
#        2022    958,040    503,875   1,461,915
#        2023    756,501    326,711   1,083,212
#        2024    704,959    416,617   1,121,576
#     Charting `I` alone reports a 137,000x increase in idle farmland that did not happen.
#     THE COMPARABLE SERIES IS `I + X`. `UL` also appears only from 2021. 2014 has a different
#     schema and 400s on an ACRES groupBy — treat it as a separate epoch, not the first point.
#     Signals that SURVIVE reconciliation (and are the story):
#       rice   510,795 (2019) -> 260,839 (2022) -> 479,663 (2024)   drought elasticity
#       vine   872,372 (2016) -> 741,590 (2024)                     -15% structural contraction
#       YP     228,346 (2016) ->  74,203 (2024)                     new plantings -67%
#     Statewide total holds at ~14.5M acres every year, so the ledger balances.

# ─────────────────────────────────────────────────────────────────────────────
# 4.4 The statutory record — 20 biennial vintages, 1984 to 2022, each its own FeatureServer.
curl -s "https://gis.conservation.ca.gov/server/rest/services/DLRP?f=json" | jq -r '.services[].name'
#  -> CaliforniaImportantFarmland_1984 … _2022 (20) · _urbanbuiltup · _firstmapped · _countyboundaries
#     CaliforniaWilliamsonActEnrollment_2021…_2025 · Ag_ConservationEasements · Ag_PlanningGrants
#     CIFFAnalysisTool / CIFFPolygon / CIFFPointBuffer / CIFFPolygonBuffer (GPServer, ASYNC)
FMMP=https://gis.conservation.ca.gov/server/rest/services/DLRP/CaliforniaImportantFarmland_2022/FeatureServer/0
curl -s "$FMMP?f=json" | jq -r '.fields[].name'
#  -> county_nam, polygon_ac, polygon_ty, County, upd_year, MetadataLink, PopupInfo
#  -> 2022 acres: Z 14,539,666 · G 13,581,442 · X 6,119,836 · P 3,849,265 · D 2,693,753
#     L 2,630,642 · nv 2,123,671 · S 1,903,023 · U 1,104,065 · W 639,167 · V 345,979 · R 152,214
#     ** DROP polygon_ty='Z' (outside survey area) or 14.5M acres becomes your denominator. **
#     ** D = URBAN AND BUILT-UP here, but D = DECIDUOUS FRUITS & NUTS in the DWR legend above.
#        NEVER share a legend between the two datasets or orchards render as cities. **
#     CEQA §21060.1(a) 'agricultural land' = P + S + U only.

# ─────────────────────────────────────────────────────────────────────────────
# 4.5 THE FOURTH TRAP: the conversion ledger is catastrophically duplicated. Do not SUM it.
URB=https://gis.conservation.ca.gov/server/rest/services/DLRP/CaliforniaImportantFarmland_urbanbuiltup/FeatureServer/0
curl -s "$URB/query?where=1%3D1&returnCountOnly=true&f=json"        #  -> 716,776 polygons
#     Per-polygon provenance: YrChngd (the cycle it became urban), PriorValue (what it was),
#     iftype_first (class when first surveyed), FirstMap (year it entered the survey).
#     Naive SUM(polygon_ac) by cycle gives a 2,256,258-acre spike at YrChngd=2014 (10x any other):
curl -s -G "$URB/query" --data-urlencode "where=YrChngd=2014" --data-urlencode "groupByFieldsForStatistics=County" \
  --data-urlencode 'outStatistics=[{"statisticType":"sum","onStatisticField":"polygon_ac","outStatisticFieldName":"ac"}]' --data-urlencode "f=json"
#  -> Alameda 1,532,533 acres.  ALAMEDA COUNTY HAS ~473,000 LAND ACRES (739 sq mi). Impossible: 3x the county.
curl -s -G "$URB/query" --data-urlencode "where=County='Alameda' AND YrChngd=2014" --data-urlencode "returnCountOnly=true" --data-urlencode "f=json"
#  -> 5,390 rows … but only 1,057 DISTINCT polygon_ac values, summing to 115,986 acres.
#     Multipart polygons are exploded into one row per part with the WHOLE polygon's acreage on each
#     => a 13x overcount. And FirstMap=1984 on 2,088,950 of the 2014 acres => pre-existing urban land
#     re-stamped in a survey rework, not new conversion.
#     GROUND TRUTH: the 2022 snapshot puts Alameda urban at 148,136 acres over 88 polygons.
#  ** THE DEFENSIBLE METHOD IS TO DIFFERENCE THE BIENNIAL SNAPSHOTS (what the state itself reports).
#     Use urbanbuiltup for the ATTRIBUTION of a conversion, never for the total. **

# ─────────────────────────────────────────────────────────────────────────────
# 4.6 Protection — and a dirty domain that silently drops 173,699 acres.
WA=https://gis.conservation.ca.gov/server/rest/services/DLRP/CaliforniaWilliamsonActEnrollment_2025/FeatureServer
curl -s "$WA/0?f=json" | jq -r '.error.message'      #  -> "Layer not found"   ** THE LAYER IS 9, NOT 0 **
curl -s "$WA/9/query?where=1%3D1&returnCountOnly=true&f=json"   #  -> 123,980 contracts
curl -s -G "$WA/9/query" --data-urlencode "where=1=1" --data-urlencode "groupByFieldsForStatistics=WA_Type" \
  --data-urlencode 'outStatistics=[{"statisticType":"sum","onStatisticField":"Acreage","outStatisticFieldName":"ac"}]' --data-urlencode "f=json"
#  -> Nonprime 8,607,673 · Prime 3,952,180 · Mixed 1,910,047 · FSZ 808,058
#     Non-Prime 173,699   <- ** SPELLING VARIANT of Nonprime. Normalise or you lose 173,699 acres. **
#     Nonrenewal 155,080  <- a STATUS, not a type: land in the 9-year wind-down. The single most
#                            predictive open signal that a parcel is leaving agriculture.
#     Carries APN_Number — the one state layer that joins to county assessor parcels.
curl -s "https://gis.conservation.ca.gov/server/rest/services/DLRP/Ag_ConservationEasements/FeatureServer/0/query?where=1%3D1&returnCountOnly=true&f=json"
#  -> 321.  ** POINTS, NOT POLYGONS ** (ACRES, HOLDER, LANDUSE, TERM, Easement_T; ClosingDat is a STRING).
#     The LESA 'Surrounding Protected Resource Land' factor wants area and this cannot supply it —
#     buffer by sqrt(ACRES) as a LABELLED approximation, or use a county easement layer.

# ─────────────────────────────────────────────────────────────────────────────
# 4.7 The LESA Land Evaluation inputs — keyless, CORS-open, and both halves are here.
SDA=https://sdmdataaccess.nrcs.usda.gov/Tabular/post.rest
curl -s -o /dev/null -D - -X POST -H "Origin: https://example.com" -H "Content-Type: application/json" \
  -d '{"format":"JSON","query":"SELECT TOP 1 mukey FROM mapunit"}' "$SDA" | grep -i access-control
#  -> Access-Control-Allow-Origin: *   Access-Control-Allow-Methods: GET,POST,OPTIONS   (browser-callable)

# areasymbol is the SOIL SURVEY AREA, not the county FIPS. 'CA019' returns {} — Fresno is CA653/CA654.
curl -s -X POST -H "Content-Type: application/json" "$SDA" \
  -d '{"format":"JSON+COLUMNNAME","query":"SELECT areasymbol, areaname FROM legend WHERE areaname LIKE '\''%Fresno%'\''"}'
#  -> CA653 Fresno County Western Part · CA654 Eastern Fresno Area · CA750 · CA760   (120 CA survey areas)

# Storie Index lives in cointerp. ** ruledepth=0 IS MANDATORY ** or you get the six sub-rules
# ('Growing season length', 'Chemical properties', …) instead of the score.
cat > q.json <<'EOF'
{"format":"JSON+COLUMNNAME","query":"SELECT TOP 8 mu.mukey, mu.muname, ma.iccdcd, ma.niccdcd, c.compname, c.comppct_r, ci.interphr, ci.interphrc FROM legend l JOIN mapunit mu ON mu.lkey=l.lkey JOIN muaggatt ma ON ma.mukey=mu.mukey JOIN component c ON c.mukey=mu.mukey JOIN cointerp ci ON ci.cokey=c.cokey WHERE l.areasymbol='CA654' AND ci.mrulename='AGR - California Revised Storie Index (CA)' AND ci.ruledepth=0 AND c.majcompflag='Yes'"}
EOF
curl -s -X POST -H "Content-Type: application/json" --data-binary @q.json "$SDA"
#  -> Auberry sandy loam 5-9%   iccdcd 3  niccdcd 3  interphr 0.797 'Grade 2 - Good'
#     Cajon sandy loam          iccdcd 3  niccdcd 7  interphr 0.804 'Grade 1 - Excellent'
#     Calgro complex 0-2%       iccdcd 3  niccdcd 6  interphr 0.263 @60%  AND  0.126 @25%
#     ** interphr is a 0-1 fuzzy value: x100 = the Storie score (0.797 -> 79.7). **
#     ** iccdcd (irrigated LCC) and niccdcd (non-irrigated) DIVERGE HARD (Cajon 3 vs 7) — pick the
#        branch that matches the site and PRINT WHICH YOU USED. **
#     ** A mapunit can carry several major components: ACREAGE-WEIGHT BY comppct_r, which is exactly
#        what the LESA manual requires. **
#     The spatial helper SDA_Get_Mukey_from_intersection_with_WktGeom is NOT on this endpoint
#     ("Invalid object name") and a direct mupolygon.STIntersects query TIMES OUT — resolve mukey
#     from the county mapunit set or a cached SSURGO extract, then join tabularly.

# ─────────────────────────────────────────────────────────────────────────────
# 4.8 Esri Living Atlas soils are NOT an option — named so nobody re-discovers it.
curl -s "https://landscape11.arcgis.com/arcgis/rest/services/USA_Soils_Map_Units/FeatureServer/0/query?where=1%3D1&returnCountOnly=true&f=json"
#  -> {"error":{"code":499,"message":"Token Required"}}   (the service ROOT 200s; the query does not)
#     Same for USA_Farmland_Class ImageServer. Subscriber content. The soils leg must be USDA SDA.

# 4.9 And the parcel leg is genuinely missing from the open data.
curl -s "https://services.gis.ca.gov/arcgis/rest/services/Boundaries/UCD_Parcels/MapServer/0/query?where=1%3D1&returnCountOnly=true&f=json"
#  -> {"error":{"code":400,"message":"Requested operation is not supported by this service."}}
#     Display-only, maxRecordCount 100, and the ENTIRE schema is: OBJECTID, PARNO, Shape_*.
#     No APN format, no owner, no acreage, no land-use code. State the gap; do not fake a parcel.
```

### 4.10 The LESA arithmetic, verified against a real filed study

The California LESA Model (CDC 1997, per SB 850 / Ch. 812 Stats. 1993) — the weights are fixed:

| Component | Factor | Weight |
|---|---|---|
| **Land Evaluation (50%)** | Land Capability Classification | **25%** |
| | Storie Index | **25%** |
| **Site Assessment (50%)** | Project Size | **15%** |
| | Water Resources Availability | **15%** |
| | Surrounding Agricultural Land | **15%** |
| | Surrounding Protected Resource Land | **5%** |

Thresholds (**CDC 1997, Table 9**):

| Total LESA score | Scoring decision |
|---|---|
| **0–39** | Not considered significant |
| **40–59** | Significant **only if** LE **and** SA subscores are each ≥ 20 |
| **60–79** | Significant **unless** either LE **or** SA subscore is < 20 |
| **80–100** | Considered significant |

Worked example, reproduced from a filed Riverside County study (Majestic Freeway Business Center
Phase II, 70.4 acres, May 2023) — the app must reproduce this exactly:

```
LE   LCC 70.0 …………………………………………… LE subtotal 31.7
SA   Project Size                70 × 0.15 = 10.5
     Water Resource Availability 20 × 0.15 =  3.0
     Surrounding Agricultural Land 0 × 0.15 =  0.0
     Protected Resource Land       0 × 0.05 =  0.0   SA subtotal 13.5
     FINAL LESA SCORE 45.2  ->  band 40–59  ->  SA 13.5 < 20  ->  NOT SIGNIFICANT
```

The band alone would have said "maybe"; the sub-test decides it. **A scorecard that prints only the
total is not a LESA scorecard.**

## Guided wizard — **the prompts that assign the app's defaults**

Claude runs this like an ArcGIS Instant-App wizard: ask each group, **apply the default so
"accept all" builds a complete app**, confirm a one-line summary, then run §5. Launch with
`/recipe environment_agriculture-land-use`. Every answer *sets an application default* baked into
`layers.json` / the `AppLayout`. Phrasing from `strata/docs/reference/human-language.md`.

| # | Wizard question | Options → **default** | The default it assigns |
|---|---|---|---|
| 1 · App | Title & as-of date? | free text → **"Agriculture & Land Use — The Ledger"**, today | header title + `strata:notes.asOf` |
| 2 · Region | Which area? | statewide · county · **subbasin/GSA** · custom draw → **San Joaquin Valley (Fresno · Madera · Merced · Tulare · Kings)** | initial extent + `regionfilter` default + which county services load |
| 3 · Cycles | Which pair of survey cycles? | any comparable FMMP pair → **2016 ▸ 2022** | `ribbon` selection, the matrix's from/to vintages, `?cycles=` |
| 4 · Ledger classes | Roll the 15 FMMP codes into how many ledger classes? | full 15 · **6 (Prime · Statewide · Unique · Local · Grazing · Other)** · ag-vs-non-ag | matrix axes + `classmix` segments; `Z` always excluded |
| 5 · Crop layer | Include the DWR field bench? | **yes, 2023 FINAL** · a different year · no | page 3 + `kpi-idle` + `kpi-new`; MapServer proxy URL resolved from the AGOL item |
| 6 · Idle series | Report idle as… | **`I + X` reconciled (labelled)** · raw `I` only (2021+) · omit | `kpi-idle` stat, the ribbon's excluded cycles, the `methodcard` text |
| 7 · Protection | Overlay what's protecting the land? | **Williamson Act 2025 + Ag easements** · Williamson only · none | protection layers + `kpi-nr`; `WA_Type` normalised (`Nonprime`≡`Non-Prime`) |
| 8 · LESA | Enable the scorecard page? | **yes, statutory weights locked** · yes, weights unlocked (what-if) · no | page 2, `lesa-scorecard` props, `lockWeights` |
| 9 · Parcels | Which parcel source for Project Size? | **drawn footprint acreage** · a county assessor FeatureServer (URL) · Regrid (key) | LESA Project Size branch + what the packet prints as its source |
| 10 · Irrigated? | Use irrigated or non-irrigated LCC? | **auto (irrigated where DWR maps a crop)** · always irrigated · always non-irrigated | `iccdcd` vs `niccdcd` branch + the printed provenance |
| 11 · Theme | Light ledger paper or dark review? | **light** · dark · follow system | `ThemeSpec.mode` + paired keyless basemap |
| 12 · Export | What's the deliverable? | **staff-report packet (PDF)** · atlas (one page per parcel) · CSV/GeoJSON only | `/export` composition |

## 5. Prompt-script (run in order)

```
A. /new-app — an "Agriculture & Land Use — The Ledger" open-design app.
   Theme: the §2.2 ThemeSpec (light, primary #3d6b35, fonts.scale compact, radius 3px).
   App-shell: header (title · region ▾ · cycle pair · theme-switch · [Packet ↗]), footer (sources +
   as-of), splash {once:true} carrying the §1 "Honest scope" disclaimer.
   Pages: ledger (fixed) · scorecard (fixed) · field (fixed), behind a page-nav.
   Deps: @strata/processing, @strata/export, @strata/plugin-search, @strata/plugin-statusbar.
   Run: pnpm -C strata build && pnpm dev   (the @strata/* packages resolve to ./dist).

B. /add-data — the verified layers from §3/§4. Set the initial extent to the San Joaquin Valley.
   THE MATRIX PAIR (both cycles, joined on UniqueID — see step D):
   dwr-crop-from  .../i15_Crop_Mapping_2018/MapServer/0   item 83f2f37783a64323acc4c5b68e88c0c8
   dwr-crop-to    .../i15_Crop_Mapping_2023/MapServer/0   item d94e891e00364e49a2ed9e9e2e27837d
                  (base: utility.arcgis.com/usrsvcs/servers/<itemid>/rest/services/Planning/…)
   CONTEXT + THE URBAN LEG:
   fmmp-from   DOC CaliforniaImportantFarmland_2016/FeatureServer/0   where polygon_ty <> 'Z'
   fmmp-current DOC CaliforniaImportantFarmland_2022/FeatureServer/0  where polygon_ty <> 'Z'
   fmmp-urban  DOC CaliforniaImportantFarmland_urbanbuiltup/FeatureServer/0   (attribution only)
   wa-2025     DOC CaliforniaWilliamsonActEnrollment_2025/FeatureServer/**9**
   ag-easements DOC Ag_ConservationEasements/FeatureServer/0        (points)
   gsa / cod / basins / districts  DWR i03_Groundwater_Sustainability_Agencies,
                                   i08_CriticallyOverdraftedBasins, i08_B118_CA_GroundwaterBasins,
                                   i03_WaterDistricts
   veg-whr     CAL FIRE California_Vegetation_WHR_TYPE/MapServer     (context, default off)

C. /symbology + /popup — genuine ESRI drawingInfo/popupInfo on verified fields.
   fmmp-*     uniqueValue on polygon_ty, 15 codes, fills at alpha 40/255. TWO SEPARATE LEGENDS:
              FMMP D = Urban and Built-up; DWR D = Deciduous. Never share a renderer.
   dwr-crop   uniqueValue on TRIM(SYMB_CLASS) via arcade (NOT CLASS1 — it is '**' on 94% of rows).
   wa-2025    uniqueValue on a normalised WA_Type arcade expression collapsing
              'Nonprime' | 'Non-Prime'; Nonrenewal gets its own hatched, high-contrast symbol.
   popupInfo  fmmp-urban: polygon_ac, PriorValue, iftype_first, YrChngd, FirstMap, County
              dwr-crop:   SYMB_CLASS, MAIN_CROP, IRR_TYP1PA, YR_PLANTED, ACRES, COUNTY, HYDRO_RGN
              wa-2025:    APN_Number, WA_Type, Acreage, County_Cit, Year

D. THE MATRIX — an attribute join, NOT a spatial overlay. (Revised after building it; see §0.1.)
   DWR/Land IQ's UniqueID is STABLE ACROSS VINTAGES with identical geometry (verified: field
   1032728 is V in 2022 and YP in 2023 at the same 36.685 ac). So:
     · page both cycles for the county with outFields=UniqueID,SYMB_CLASS,ACRES, returnGeometry=false
       (maxRecordCount 2000, supportsPagination true -> resultOffset paging, 4-wide);
     · join on UniqueID, cross-tab classOf(from) x classOf(to), sum the TO-cycle ACRES.
   That is the whole matrix: ~7.6 s for a 10k-field county, no geometry fetched, and it sidesteps
   the fmmp-urban duplication trap entirely. Fields with no counterpart are EXCLUDED, not counted
   as change. Acreage comes from the TO cycle (Land IQ re-cuts some fields, keeping the id).
   /analyze is still used for:
     buffer       0.25 mi Zone of Influence -> the three LESA SA surround factors.
     pointsWithin ag-easements inside the ZOI (buffered by sqrt(ACRES), labelled approximate).
     aggregate    county / subbasin rollups for regionfilter scoping.
   FMMP keeps the URBAN leg and the farmland-class context — and where a county's crop survey has
   no U/UL class at all (Madera has none), the app must SAY the urban column is unsurveyed, not
   render an empty one. NEVER sum fmmp-urban.polygon_ac (13x multipart duplication, §4.5).

E. /panel statistics — the §2.3 KPI row: kpi-lost, kpi-idle (SYMB_CLASS IN ('I','X')), kpi-new
   (SYMB_CLASS='YP'), kpi-nr (WA_Type='Nonrenewal'), kpi-inview. All via dataSource stat, no wiring.

F. /panel chart + /panel table — classmix (stacked-bar), sizedist (chart histogram on polygon_ac),
   celltable (server-paged, CSV/GeoJSON), factortable, croptable, cropcarto (carto: category on
   SYMB_CLASS and IRR_TYP1PA, histogram on YR_PLANTED).

G. WIF: author AppLayout.connections — the 27 wires in §2.7, plus the four zero-connection
   fromWidget links (ledger -> balance+4 KPIs; scorecard -> gauge+verdict+factortable;
   soilquery -> scorecard; zoi -> scorecard).

H. The three new widgets (§2.6) via an app-local registry override, each shipping behind its
   day-1 fallback so the app is complete either way:
     transition-matrix   fallback: carto rows + stacked-bar + table + sparkline
     lesa-scorecard      fallback: weighted-overlay (statutory weights) + gauge + kpi + table
     comparability-ribbon fallback: horizontal stacked-bar + text callout + filter picker

I. Controls + composed export (+ share deep-link on ?cycles and ?region).
   theme: mode "dark" by default, theme-switch to the light print mode; RE-PAINT the ink map layers
     on every flip (MapLibre paint cannot read CSS variables) and theme MapLibre's own zoom/scale/
     attribution or the cluster stays white on dark.
   layer-panel + basemap as ONE map-anchored control group directly above the zoom cluster, matching
     its chrome (29px, 4px radius, same ring, 1px divider) with ESRI/Calcite glyphs. The layer panel
     is the single source of truth for visibility; the cell-isolate writes the same state.
   status-bar; bookmarks for the subbasins and MLRP regions.
   Retry transient upstream 500s with backoff — gis.water.ca.gov failed 1-in-3 during testing.
```

## 6. Verify (benchmark: DOC CIFF · Esri Agricultural Parcel Analysis · MapBiomas)

✅ = proven by the built demo in `app/` and asserted in `app/test-ledger.mjs` (103/103 green,
2026-07-27). ⬜ = still to prove in the strata-core `<StrataApp>` build.

| Check | Pass |
|---|---|
| Silhouette is the ledger-book matrix; distinct from all five `natural-*` neighbours | ✅ |
| The signature loop (cell → map + table + balance → LESA) works end-to-end | ✅ |
| ≥3 `connections` fire in the strata-core build | ⬜ |
| Every `layerId` + field verified against the service (§4); no field written from memory | ✅ |
| **`SYMB_CLASS`, never `CLASS1`** — the crop layer renders in colour, not one flat grey state | ✅ |
| **Idle reported as `I + X`** always, with the 2020\|2021 break disclosed in the method note | ✅ |
| **No `SUM(polygon_ac)` on `fmmp-urban`**; the Alameda cross-check (148,136 ac / 88 polygons) holds | ✅ |
| **`YrChngd=2014` excluded by construction** and explained, not charted | ✅ |
| **`polygon_ty='Z'` excluded** from every denominator | ✅ |
| **`WA_Type` normalised** — a `Nonprime` filter returns 8,781,372 ac, not 8,607,673 | ✅ |
| Williamson Act bound to **layer 9**, not 0 | ✅ |
| FMMP and DWR legends are separate — no orchard renders as a city | ✅ |
| LESA reproduces the §4.10 worked example: **45.2 → band 40–59 → SA 13.5 < 20 → not significant** | ✅ |
| Every LESA factor prints its **source** and its **branch** (irrigated vs non-irrigated LCC) | ✅ |
| Storie read at **`ruledepth=0`**, acreage-weighted by `comppct_r` | ✅ |
| Transitions are a **UniqueID join**, not a spatial overlay; unmatched fields excluded, not counted | ✅ |
| Acreage taken from the **TO** cycle (Land IQ re-cuts some fields, keeping the id) | ✅ |
| A county with **no urban class** in its survey says so, rather than showing an empty urban column | ✅ |
| The app never says "converted" — it says which survey classed it what, when | ✅ |
| **A selected cell is a toggle** — a second click releases the filter everywhere and closes the scorecard | ✅ |
| **The KPI strip is scoped to the selection** and reverts on clear — never county totals over a filtered matrix | ✅ |
| **Both cycle endpoints are settable** (alternating ribbon + explicit From/To pickers) | ✅ |
| The matrix has its **own scroll track and a floor** — disclosure below it cannot squeeze the protagonist | ✅ |
| Layer + basemap controls are **one group above the zoom cluster**, matching its chrome; MapLibre's own controls themed | ✅ |
| Launches **dark**; the light mode is print/counter. The ink layer is re-painted in JS on every flip | ✅ |
| **Transient upstream 500s are retried**, not surfaced as a dead panel | ✅ |
| `responsive.small` collapses every side-by-side row; the matrix scrolls in its own track | ✅ |
| Basemap keyless; everything EPSG:4326; no token or API key anywhere | ✅ |
| Runs on Strata **and** ArcGIS; read-only degradation path holds (no writes required) | ✅ |
| Packet carries a source line per figure and the scope disclaimer | ✅ (text; PDF is the `/export` step) |

## 7. Harvest (gaps → strata-core)

- **`ledger-book` is a harvest candidate on its first build.** The from→to matrix over two official
  vintages generalises well beyond farmland: loan-book migration between risk grades, zoning
  reclassification, water-right transfers, habitat gain/loss accounting, land-cover change anywhere.
  Second use ⇒ promote to a numbered template in `../APP-TEMPLATE-LIBRARY.md`.
- **`comparability-ribbon` belongs in the core**, not in one recipe. Every long official series has
  methodology breaks, and every recipe built on one currently has no way to disclose them.
- **Release `time-player` back to the Natural Resources & Environment sector** in the template
  assignment table.
- **Gaps to file as strata-core issues:** (a) no first-class matrix/crosstab widget; (b) `carto`
  cannot express a two-dimensional category cross-tab; (c) no built-in "vintage/series" data-source
  kind that carries comparability metadata; (d) `weighted-overlay` has no threshold/verdict
  expression, so a scored model has to render its own verdict; (e) **no `clear`-on-reselect
  convention** — every recipe re-implements "click the active thing again to release the filter";
  (f) **`kpi` cannot scope itself to a selection** without the publisher hand-rolling it, so scoped
  stat cards are per-app work; (g) **map-anchored controls have no shared chrome contract** — matching
  the MapLibre zoom cluster (size, radius, ring, divider) and theming MapLibre's own controls is
  hand-written CSS in every recipe, and it is the difference between "native" and "bolted on";
  (h) **no retry/backoff in the data layer** — public services 500 intermittently and every app
  currently discovers this for itself.

## 8. Sources

- Site: `https://tabaqat.net/solutions` (**Natural Resources & Environment** → Agriculture & Land Use)
- **Regulatory:** Gov. Code §65570 · CEQA Guidelines §15126.2(a), §15360, §15370, §21060.1(a) ·
  California Land Conservation Act (Williamson Act) 1965 · SB 850 (Ch. 812, Stats. 1993) ·
  AB 252 (MLRP) · SGMA (2014)
- **Methodology:** CDC 1997, *California Agricultural Land Evaluation and Site Assessment Model
  Instruction Manual* (weights + Table 9 thresholds) · UC Div. of Agricultural Sciences Spec. Pub.
  3203 (1978), *Storie Index Soil Rating* · USDA Web Soil Survey / SSURGO · Land IQ land-use mapping
  methodology (random forest, field scale, ~98% since 2018)
- **Data (all curl-verified 2026-07-27):** `gis.conservation.ca.gov/server/rest/services/DLRP` ·
  `utility.arcgis.com/usrsvcs/servers/<itemid>/…/i15_Crop_Mapping_<year>` ·
  `sdmdataaccess.nrcs.usda.gov/Tabular/post.rest` · `gis.water.ca.gov/arcgis/rest/services` ·
  `services.gis.ca.gov/arcgis/rest/services` · `nassgeodata.gmu.edu/CropScape` ·
  `quickstats.nass.usda.gov/api`
- **Competitive:** DOC California Important Farmland Finder (`maps.conservation.ca.gov/dlrp/ciff/`) ·
  Esri ArcGIS Solutions *Agricultural Parcel Analysis* (`doc.arcgis.com`) · Esri/Impact Observatory
  Sentinel-2 10m Land Cover Explorer · MapBiomas · Global Forest Watch (WRI/GLAD) · Climate
  FieldView · John Deere Operations Center · BASF xarvio · OneSoil · EOS Crop Monitoring · Taranis ·
  Trimble Ag · Regrid · AcreValue
- **Market/context:** PPIC — *Managing Water and Farmland Transitions in the San Joaquin Valley*,
  *SGMA-Ready Crops as a Low-Water Alternative to Fallowing* · American Farmland Trust — *Farms Under
  Threat: The State of the States* · EDF/DOC — MLRP 2025 progress reporting · Ag Alert /
  Maven's Notebook, May 2026 — "Groundwater law begins reshaping valley"
- **Internal:** `DESIGN-PROPOSAL.md` (the full design deliverable) · `agland-catalog-ca.json` ·
  `build_agland_catalog.py` · `../APP-TEMPLATE-LIBRARY.md` · `../research/esri-solutions-inventory.md` ·
  `strata/recipes/COMPONENT-MANIFEST.md` (§10 freestyle charter) · `strata/docs/guide/app-design.md` ·
  `strata/docs/reference/human-language.md`

---

## Modernization (parity release)

> Applies the Experience-Builder-parity features (Phases 1–7). See `strata/recipes/MODERNIZATION.md`
> + `COMPONENT-MANIFEST.md` §8. Cross-cutting: a structured **`theme`**, app-shell
> (`header`/`footer`/`splash`), and `dataSource.{ sourceId }` linking (widgets sharing a source link
> with **no `connections`**).

- **Structured `ThemeSpec`** (§2.2) — **dark by default**, light ledger paper as the print/counter
  mode, both from one `primary` hex per mode, `fonts.scale: "compact"`, `--strata-radius-md: 3px` and
  `--strata-elevation-1: none` to kill card shadows. The single biggest visual lever, and here it
  carries meaning: debit red / credit green are theme roles (`danger`/`success`), not chart colours.
- **App-shell** — header with region + cycle pair + theme switch + packet button; footer with the
  source line and as-of; `splash {once:true}` carrying the honest-scope disclaimer.
- **DataSource linking** — the design's spine: `ledger` publishes once and five widgets read it,
  `scorecard` publishes once and three read it, with `soilquery`/`zoi` feeding `scorecard`. Nine
  widget updates on **zero** connections — and because the publish carries the selection, the KPI row
  scopes itself to the clicked cell through the same binding.
- **`RestDataSource`** for `agland-catalog-ca.json` and the USDA SDA POST; **`FileDataSource`** for
  uploading a project boundary or a batch site list. **No `StreamDataSource`** — the fastest input in
  this business is a biennial survey.
- **Layout nodes** — `splitter` (resizable rail | map), `panel` (the ledger rail), `window` (the LESA
  drill-in, opened by `showHide`), `views` + `mapState` (the four field-bench tabs), `accordion`
  (phone collapse).
- **Motion** — `animate: "fade"` with a short `animateOptions.stagger` on the matrix rows as they
  fill, so the account visibly settles. Nothing on the map container.
- **Analysis widgets** — `query` (`soilquery`, publishing SSURGO rows as an output), `analysis`
  (`zoi` buffer, publishing the ZOI), `weighted-overlay` (the shipped LESA fallback).
- **Deliberate non-uses, each for a stated reason:** `timeslider` (would animate a methodology
  break), `swipe` (two choropleths is the weak comparison this app replaces), `date-filter` (the axis
  is a survey cycle, not a date), `hexbin`/`hotspot` (would render a population map), `routing`
  (no travel-time question), `elevation`, `embed`/`video`, `StreamDataSource`. Full sweep:
  `DESIGN-PROPOSAL.md` §8.
