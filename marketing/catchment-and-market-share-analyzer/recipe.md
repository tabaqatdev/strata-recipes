# Recipe — Catchment & Market-Share Analyzer (Marketing)

A reproducible path to a **catchment and market-share** application on strata-core that answers the
question every share figure hides: *"how much of this number is the market, and how much is our own
choice of denominator?"* It unions the competing outlet registers into one layer with provenance, draws
six defensible catchments (including a parameter-free proximity catchment computed on the real road
network), and reports the share **as a fraction with a named denominator plus the spread of every other
defensible reading** — then prices the fair-share gap against the client's own book.

> **Scope (honest).** An **analytics and serving layer over open registers plus the client's own sales
> book** — not a system of record, not audited market measurement, not a forecast. **No Huff model is
> run** (its attractiveness term would have to be invented). **No footfall**: every visit column in the
> only open California POI extract dates to **19 Apr – 9 May 2020** and is quarantined. **No
> leakage/surplus**: CDTFA taxable sales are 403-blocked server-side and are reported by permit-holder
> city, not store location. **Dollar demand is a published multiplier** (BLS CEX), printed on screen with
> its series id, never a measured local figure. **Penetration requires the client's book** — without it
> the app reports *position* only, and says so. Deploy **on-prem behind the client's perimeter**; the
> sales book never touches a cloud LLM.

Run the §5 prompt-script on a fresh strata-core project (or via `/new-app`). ESRI Web Map JSON is the map
contract; register ingest is via `/convert` → `/publish` to Strata Serve (no per-feature editing —
the app has **no write path at all**, so it runs identically on Strata and ArcGIS).

---

## 0. Provenance

| | |
|---|---|
| **Source** | `https://tabaqat.net/solutions` — **Marketing** section |
| **Name on site** | Catchment & Market-Share Analyzer |
| **Tagline on site** | "Trade areas, penetration and share by area against competitors" |
| **Researched** | 2026-08-06 |
| **Template** | **`open-design`** ("ratio-bench") — see `DESIGN-PROPOSAL.md` §2.4 for why the library's provisional `nearby-finder` was rejected |
| **Catalog** | `catchment-catalog-ca.json` — 2,340 CA services, 287 role-tagged, 16 external feeds (13 verified-live 2026-08-06). Rebuild with `python3 build_catchment_catalog.py` |

---

## 1. Study — how the market frames this

**The question the buyer asks:** *"We say we hold 18 % of this market. Marketing says 30 %. Which is
right, where exactly are we losing the difference, and what do I tell the board?"* The gap is almost never
a data-quality problem — it is two people dividing by two different denominators and neither writing it
down.

**The finding this recipe is built on (verified, not asserted).** On identical ground — bbox
`33.74,-118.30 → 33.92,-118.05`, Long Beach–Torrance — *"how many grocery competitors are here?"* has four
defensible public answers: **OSM `shop=supermarket` = 113 · SafeGraph `445110` = 323 · SafeGraph `4451` =
433 · Big Box Grocery = anchors only.** A 9-store brand therefore holds **8.0 %, 2.8 % or 2.1 %** — a
**3.8× swing before any analysis runs**.

**Reference solutions (benchmark + coexist, never copy):**
- **Esri — ArcGIS Business Analyst** defines the vocabulary: **simple rings**, **threshold rings**,
  **drive-time**, **threshold drive-time** and **customer-derived** trade areas (the 30/50/70 %
  primary-secondary-tertiary polygons), plus the **Huff** probability model. Esri's own docs are explicit
  that Huff's attractiveness term is an index the analyst supplies (usually floor area) and that both
  exponents must be **calibrated**. Retail MarketPlace adds leakage/surplus on licensed consumer-spending
  data. Market analysis is one of the few verticals ESRI leaves **un-templated** in ArcGIS Solutions.
- **Foot-traffic platforms** — **Placer.ai** (the **True Trade Area**: a polygon enclosing a chosen
  share, commonly 70 %, of observed visitors' home locations; plus captured-vs-potential market),
  **Advan Research**, **Unacast**, **StreetLight**, **Gravy**.
- **Site-selection / network platforms** — **Kalibrate** (Market Intelligence), **Buxton** (now part of
  Audiense), **SiteZeus** (relaunched as "Atlas", May 2026), **GrowthFactor**, **SiteSeer**, **CARTO**,
  **Alteryx**.
- **Geodemographic classifiers** — **CACI Acorn** (6/18/62), **Experian Mosaic** (18/68), **Esri
  Tapestry** (60/12). Reference by name; never clone a taxonomy.
- **FMCG measurement** — **Circana** (formerly IRI/NPD) and **Nielsen** own **fair share** and the **Fair
  Share Index**, which is the discipline's own name for the arithmetic this app performs.

**Our edge:** every one of those platforms **sells a denominator** — a panel, a POI file, a
classification — and none can show the number under a rival's register without undermining its product.
SafeGraph's own published comparison reports **100 % coverage / 95.6 % fill for SafeGraph vs 26 % / 39.8 %
for OSM** in a controlled Dollar General test; that is a vendor benchmarking itself (cite it, do not adopt
it), but it concedes the point: **registers disagree materially, and the disagreement is the buyer's
risk.** An open, MIT, on-prem tool is structurally advantaged here because **we have no register to
defend** — we can print all four and the spread between them. Plus the standing edges: AI-authored, runs
on **Strata *or* ArcGIS**, sovereign/on-prem so the sales book never leaves the perimeter.

**Standards, specifications & organizations to speak fluently:**
- **NAICS 2017/2022** (US Census Bureau) — the category boundary. `445110` vs `4451` vs `445` is the
  single most consequential definitional choice in the app.
- **Fair share · Fair Share Index · Category Development Index** (Circana/Nielsen) — FSI = share of
  tactic ÷ share of dollars × 100.
- **Market Penetration Index (MPI)** — the hotel-sector form of the same ratio; the version finance
  teams already recognize.
- **Huff (1963)** and **Nakanishi–Cooper MCI (1974)** — the gravity/attractiveness lineage. Name it;
  explain why we do not run it uncalibrated.
- **Reilly's law of retail gravitation** — the breakpoint between two centres; ancestor of the
  "closer to me" catchment we *do* ship.
- **Census geography** — block group / tract / place / county; `GEOID` is the join key.
- **BLS Consumer Expenditure Survey** — the published per-consumer-unit spend multiplier.
- **ODbL** — OpenStreetMap's share-alike licence, governing any derived layer we publish.
- **ESRI Web Map / `drawingInfo` / `popupInfo` JSON** — this repo's rendering contract.

**Honest scope** — see the blockquote at the top; repeated in the app's `splash` and footer.

---

## 2. UI design spec (front-loaded)

### 2.1 Layout (Template: `open-design` — "ratio-bench")

- **Template `open-design`** under the freestyle charter (`strata/recipes/COMPONENT-MANIFEST.md` §10).
  Full derivation, three candidate silhouettes and the anti-collision check: **`DESIGN-PROPOSAL.md`**.
- **Why not the library's `nearby-finder`:** (a) it is *also* assigned to
  `real-estate_tenant-mix-and-catchment`, whose scaffold names this recipe as its sibling — two catchment
  apps with the same ring gesture on adjacent catalog cards is the exact failure the library prevents;
  (b) **the ring is the thing this app exists to question**. `nearby-finder` is **freed for Marketing
  use**; the ring survives, correctly demoted, as one of six selectable catchment definitions.
- **The silhouette:** the **share fraction itself** is the interface — a live numerator over a live
  denominator, with the choices that construct it on a bench above and the **spread of every other
  defensible reading** as a band beneath.
- **A third bench axis was added when it was built — the category.** The design specified two choices
  (who counts, where); building it showed the thesis is not a grocery quirk, so *what you sell* became
  the first row: **Grocery · Restaurants · Fuel**, each with its own competing registers. The registers
  disagree by **12.4×**, **4.5×** and **2.3×** respectively on identical ground — see §3.
- **Signature loop:** **click any tick on the spread band → the whole app adopts that definition** (map
  repaints, fraction recomputes, brand table re-ranks, URL updates).
- **Wiring floor:** 12 authored `connections` (the guideline's floor is 3), plus 7 widgets linked by a
  shared `sourceId` with no wiring at all. **The built app ships 23 wired behaviours** — full table in
  `app/README.md`.

```
┌ HEADER ─ Catchment & Market-Share Analyzer · Grocery · Long Beach–Torrance ····· "ours"=Vons ·······┐
│                       [search a place] [Analyse this view] [Add drive times] [Export] [⛶] [Light]   │
├ THE BENCH ── two tight rows, 77 px ─────────────────────────────────────────────────────────────────┤
│ YOU SELL [Grocery][Restaurants][Fuel] │ WHO COUNTS·THE DENOMINATOR [OSM 113][445110 323][4451 433][…]│
│ WHERE·THE AREA [3 mi][5 mi][10 min][15 min][Closer to us][City]   …one line on what it ignores       │
├──────────────────────────────────────┬──────────────────────────────────────────────────────────────┤
│  ┌────────────────────────────────┐  │                        MAP                    ┌─┐ + − ⌂      │
│  │            8.0 %               │  │   ● ours (teal) · ○ theirs (slate)            │ │ ▤ L layers │
│  │        9 ──── of ──── 113      │  │   ▒ tracts shaded by ADVANTAGE                │ │ ◈ B basemap│
│  └────────────────────────────────┘  │       teal ← closer to us ·· amber contested   └─┘ ☰ G legend│
│  [amber caution if self-selecting]   │   ━━ catchment outline, drawn (not assumed)                  │
│  SPREAD  ▏▏▏  ▏      ▏      ●        │  ┌ LEGEND — every row is a control ───────┐                  │
│   hover=preview · click=adopt · ←→    │  │ ours — Vons          9  ← click: hide  │                  │
│  households 36,524 · demand $227 M   │  │ competitors        104                 │                  │
│  [ ] follow the map view             │  │ ours by >3 km        0  ← click: isolate│                  │
│  positioned 34 % · captured — · gap —│  │ contested            0                 │                  │
│                                      │  └────────────────────────────────────────┘        4326 ▸    │
├══════════════════════════════════════╪══════════════════════════════════════════════════════════════┤
│ DERIVATION — printed, with sources   │ BRAND TABLE — brand · outlets · share · rank · SAME GROUND ·  │
│  (same width as the reading panel —  │   other register · identity   ·  row → set as ours / zoom     │
│   one unbroken seam, one --col-w)    │                                                              │
└──────────────────────────────────────┴──────────────────────────────────────────────────────────────┘
     ▲ drag either vertical gutter — both drive one width, so the seam never breaks
```

**Map chrome (built).** One control vocabulary, **on the map** — MapLibre's own zoom cluster is hidden
and replaced by a single top-right cluster: zoom in · zoom out · **fit to this reading's outlets** ·
Layers · Basemap · Legend, with `L`/`B`/`G` shortcuts. Drawers open beside the cluster, never over it;
each **basemap option carries a live tile thumbnail of the current market in that style**. Same pattern
as `industry_mining-and-concession-compliance`.

**The legend is a control surface, not a caption.** Outlet rows toggle their layer (in sync with the
Layers drawer); each **advantage band isolates** that band on the map (`Esc` clears). It filters rather
than fades, so a hidden tract cannot be clicked through. Every row carries a live count, so the legend
doubles as the distribution of the reading — and isolating a band deliberately does **not** change the
reading, which the hint line states.

**Responsive.** Below 1100 px the body collapses to a column, the map drops to 55 vh and the gutters
hide (nothing is side-by-side to resize). The **fraction card and the spread band survive the collapse
intact** — everything else may scroll.

### 2.2 Theme

Structured `ThemeSpec`, **`mode: "light"`** *(revised 2026-08-06 — the proposal originally specified
dark; building it and rendering both modes overturned that)*. Light wins on three counts: the diverging
advantage ramp keeps its semantics in light and turns into **saturated alarm-orange on a dark basemap**;
place names on Dark Matter become unreadable under a low-alpha fill, and a business reader navigates by
place name; and the artefact is a **counter document** — a fraction with a printed derivation that
leaves as a CSV or a PNG in a white deck. **Dark stays first-class** via `theme-switch` (better on a wall
display or in a dim room), and an explicitly chosen basemap now survives a theme swap. Differentiation
from the two light Marketing siblings is carried by the **silhouette**, which is what the design
guideline asks for anyway. Full reasoning: `DESIGN-PROPOSAL.md` §6.

`primary #2dd4bf` (instrument teal — *ours*, and nothing else) · `secondary #94a3b8` (theirs, deliberately
inert — a competitor is not a threat colour) · `warning #f59e0b` (the contested band and the spread) ·
`danger #f43f5e` (the fair-share gap) · app bg `#0b1220` · panel `#111a2b` · `fonts.scale "compact"` ·
**mono override on `kpi`** so the fraction and the spread ticks align to one grid · motion `180 ms`,
short and mechanical, because every animation here is a measurement changing.

Basemap **keyless**: CARTO Dark Matter (dark) / Positron (light) from `OPEN_BASEMAPS`. Polygon fills
alpha ≈ 40/255. **EN + AR/RTL is real, not decorative** — the Saudi binding serves genuine EN/AR layer
pairs, so `lang-switch` swaps `layerId`.

### 2.3 KPI cards

Seven, all live `kpi`/`gauge` `stat: {field, op}` bound to shared `sourceId`s (`readings`, `demand`) so
they update on every bench change **with no `connections`**:

**Share %** (the fraction's headline) · **ours** (`mine_n`) · **in the market** (`all_n`) · **households in
catchment** (`hh`) · **category demand $/yr** (`hh_usd`) · **positioned to win %** (radial `gauge`,
thresholds 25/50) · **actually captured %** and **fair-share gap (pts)** — the last two greyed with an
explicit *"needs your sales file"* note when no client book is bound.

### 2.4 Charts & table

- **The spread band** — the signature. A new `spread-band` widget (§10.3 block in `DESIGN-PROPOSAL.md`
  §9) plotting the 24 precomputed readings with the active one marked; **named fallback: a `chart`
  `kind:"bar"`** over the same `readings` source, which emits `categorySelect` natively so the signature
  loop survives unchanged. **Ship the fallback on day one.**
- **Brand table** — `AttributeTablePanel` over `brand_rollup`: `brand` · `outlets_n` · `share_pct` ·
  `rank_n` · **`share_alt` ("under NAICS 445110")**. That last column is the thesis in one cell. Sortable,
  per-column filter, row → `zoomTo` + `flash`, CSV/GeoJSON export.
- **No second chart.** A chart rail would make this a chart-board and collide with
  `financial-services_concentration-and-exposure-analyzer`.

### 2.5 Capabilities to use

- **`/analyze`** at build time: `centroids` on block groups; `nearest` twice per demand unit (nearest
  ours, nearest competitor) → `adv_s`; `buffer` only to draw the ring catchments. **`voronoi` rejected**
  (straight-line, ignores the road network, and `competitor-and-whitespace-map` owns it);
  **`hexbin`/`hotspot` rejected** (density answers "where are they clustered", not "whose is it").
- **Routing** — `@strata/plugin-routing` **OSRM `/table`** is the engine (verified keyless); computed at
  build time, tiled by city, cached as columns. `fetchIsochrone` is **opt-in only** (needs self-hosted
  Valhalla or keyed ORS).
- **WIF `connections`** — 12 (§2.1 loop + register chips → filter map/table/spread + `setUrlParam`
  deep-link + table `rowSelect` → `zoomTo`/`flash` + map `extentChange` → `showStatistics` +
  `featureSelect` → `viewInTable`).
- **`views` + `mapState`** — the catchment tabs. Each tab's `mapState.definitionExpression` **is** a
  catchment definition, applied by the ViewsNode itself (no dependence on the pending `viewChange`
  emitter).
- **Deliberately NOT used:** `filter`/`query` (a free-form builder would let a user construct an
  indefensible denominator the app cannot price), `date-filter`/`timeslider` (registers are
  single-vintage; animating them would imply a market change that is really a data-source change),
  `layer-panel`/`add-data`/`analysis`/`weighted-overlay`/`draw`/`near-me` (each invites a user-authored
  catchment at runtime — the freedom this app deliberately withholds), `button`/`controller`
  (`buttonClick` is a **Phase-2 pending emitter**). Full sweep: `DESIGN-PROPOSAL.md` §8.
- **Composed export** — a board **PDF** (legend + scalebar + north-arrow + the printed derivation + the
  spread table), a per-city **atlas**, CSV/GeoJSON, and `exportSpec` so the reading round-trips to ArcGIS.

---

## 3. Data sources

All EPSG:4326 (reproject on ingest). Every California row below was `curl`'d on **2026-08-06**; the Saudi
rows were read from the connected Strata Serve the same day. Full provenance, counts and traps:
**`catchment-catalog-ca.json`** (16 external feeds, 13 verified-live).

| Role | California (open) | Saudi Arabia (client Strata Serve) | National / engine |
|---|---|---|---|
| **Supply register — the denominator** | Cal OES `Retail/FeatureServer/0` (**82,905**; `4451` = **13,363**, `445110` = **10,292**) · `Restaurants` (**85,109**) · `Hospitality` (**6,922**) · `Big_Box_Grocery_Stores/0` (**1,014**) · `CA_GasStations/0` (**9,617**) | `tabaqat/locations-and-points-of-interest` layer **14** Commercial Centres EN (**2,981**, source MODON) / **15** AR; also Banks, ATM Locations, Gas Stations, Hotels, High Scale POI | **OpenStreetMap via Overpass** — keyless, **current to the hour** (`timestamp_osm_base 2026-08-06T07:15:01Z`), carries `brand:wikidata` |
| **Own network — the numerator** | client-supplied (CSV → `FileDataSource`, or a published FeatureServer). Public template ships **synthetic** own-outlets | client-supplied | — |
| **Demand base** | Cal OES `CA_CensusTracts_2023/0` — **9,107 tracts, 164 fields** (`TOTHH_CY`, `MEDDI_CY`, `HINC0_CY`…`HINC200_CY`, `_FY` projections) | `tabaqat/census-and-statistics` | **Census TIGERweb** `Tracts_Blocks/MapServer` — keyless geometry, **block groups at layers 1/5/8/11** |
| **Demand value ($)** | Caltrans `CHhqcore/Economic_Opportunity/0` (8,035 tracts — H+T burden haircut) | — | **BLS CEX** `CXUFOODHOMELB0101M` = **$6,224/CU (2024)**, $6,053 (2023), $5,703 (2022) — keyless |
| **Travel cost** | Caltrans `CHhighway/Traffic_AADT/0` (**13,919** rows ≈ 6,960 stations — pass-by exposure only, *not* a catchment input) | — | **OSRM `/table`** (`router.project-osrm.org`) — verified real driving seconds |
| **Access mode** | Caltrans `CHrailroad/CA_HQ_Transit_Areas/0` (**20,558**) — flags where a *driving* catchment understates the contested area | — | — |
| **Catchment frame** | CAL FIRE `California_Incorporated_City_Boundaries/0` (**483**) · Geoportal `Boundaries/CA_Counties/FeatureServer/0` (field **`County`**, not `POLYGON_NM`) | `tabaqat/boundaries` | — |
| **Reconciliation (aspirational)** | **CDTFA taxable sales** — `data.ca.gov` CKAN **403 / Cloudflare 1009**, re-verified 2026-08-06. City-level only, by **permit-holder city not store location** | — | — |

**CORS:** Cal OES + CAL FIRE hosted orgs are `*`; Caltrans and the state geoportal reflect the request
Origin; Overpass, OSRM and BLS are open. Browser-direct, no proxy needed. Carry OSM (**ODbL, share-alike**),
Cal OES/Esri-enrichment, BLS and OSRM attribution plus "no warranty".

**The thesis holds in every category (all counts live, same bbox, 2026-08-06).** Building the app
added two more categories to the original grocery finding:

| Category | Registers (narrowest → widest) | Disagreement |
|---|---|---|
| **Grocery** | Big Box 35 · OSM 113 · NAICS 445110 323 · NAICS 4451 433 | **12.4×** |
| **Restaurants** | OSM 664 · NAICS 722511 1,787 · NAICS 722 2,985 | **4.5×** |
| **Fuel** | OSM 173 · CEC stations 277 · NAICS 4471 402 | **2.3×** |

⚠ **NAICS 722 is 2,985 rows and the service caps a page at 2,000.** An un-paged fetch returns exactly
2,000 without complaint — a truncated denominator **inflates your share**. `fetchRegister` pages, and a
regression assertion pins the 2,985.

**Two catalogue corrections established by this research** — both recorded in
`catchment-catalog-ca.json`:

1. **`api.census.gov` is now key-gated.** The sibling `real-estate_site-selection` catalog recorded it on
   2026-07-28 as "keyless for low volume but a bare call redirected (302)". On **2026-08-06** that 302
   resolves to an HTML page titled **"Missing Key"** — for `acs/acs5` *and* `cbp`, with and without `-L`.
   The open redistributable demand fallback is gone unless the deployer registers a key, so the
   **licensed** Cal OES enrichment is load-bearing; and **County Business Patterns is unavailable**,
   removing the one authoritative public establishment count that could have adjudicated between the POI
   registers. *That gap is the honest reason this app reports a spread instead of picking a winner.*
2. **`Retail`, `Restaurants` and `Hospitality` are one file.** All three report layer name
   **`CA_051220_WeeklyPatterns`**, the same 46 fields and the same `OBJECTID_1` quirk — category slices of
   a single SafeGraph extract totalling **174,936 CA POI**. "NAICS 722 returns 0 rows" is true of the
   *Retail* slice only; food service lives in `Restaurants` (**84,016** rows: 50,957 `722511` full-service,
   16,638 `722513` limited-service).

**The catalogue finding that justifies the design.** Of **2,340** services inventoried from
`data_sources_ca.md`, **287** carry a catchment role and exactly **5 are supply registers** — all Cal OES,
three of them slices of one 2020 file. *California's entire public ArcGIS estate contains one competitive
register.* There is no second opinion inside the state's own holdings; the alternatives must be imported
(OSM) or bought — which is exactly why a buyer cannot check the number they are given.

---

## 4. Verify each URL first (terminal)

```bash
# ── The finding the whole app rests on: four defensible answers on identical ground ──────────
B=https://services.arcgis.com/BLN4oKB0N1YSgvY8/arcgis/rest/services
BBOX='{"xmin":-118.30,"ymin":33.74,"xmax":-118.05,"ymax":33.92,"spatialReference":{"wkid":4326}}'
sg(){ curl -s -G "$B/Retail/FeatureServer/0/query" --data-urlencode "where=$1" \
      --data-urlencode "geometry=$BBOX" --data-urlencode "geometryType=esriGeometryEnvelope" \
      --data-urlencode "inSR=4326" --data-urlencode "spatialRel=esriSpatialRelIntersects" \
      --data-urlencode "returnCountOnly=true" --data-urlencode "f=json"; }
sg "naics_code='445110'"                                                  # → 323
sg "naics_code LIKE '4451%'"                                              # → 433
curl -s -X POST https://overpass-api.de/api/interpreter --data-urlencode \
 'data=[out:json][timeout:110];(nwr["shop"="supermarket"](33.74,-118.30,33.92,-118.05););out tags center;' \
 | python3 -c "import json,sys;e=json.load(sys.stdin)['elements'];print(len(e),'osm;',sum(1 for x in e if x['tags'].get('brand')),'branded')"
                                                                          # → 113 osm; 80 branded

# ── Statewide register sizes ─────────────────────────────────────────────────────────────────
q(){ curl -s -G "$1/query" --data-urlencode "where=${2:-1=1}" \
     --data-urlencode "returnCountOnly=true" --data-urlencode "f=json"; }
q "$B/Retail/FeatureServer/0"                       # 82905
q "$B/Retail/FeatureServer/0" "naics_code LIKE '4451%'"   # 13363
q "$B/Retail/FeatureServer/0" "naics_code='445110'"       # 10292
q "$B/Restaurants/FeatureServer/0" "naics_code LIKE '722%'"  # 84016  ← food service DOES exist
q "$B/Hospitality/FeatureServer/0"                  # 6922
q "$B/Big_Box_Grocery_Stores/FeatureServer/0"       # 1014
q "$B/CA_GasStations/FeatureServer/0"               # 9617

# ── PROVE the three POI services are one file (same layer name, same OID quirk) ──────────────
for s in Retail Restaurants Hospitality; do
  curl -s "$B/$s/FeatureServer/0?f=json" | python3 -c \
   "import json,sys;d=json.load(sys.stdin);print('$s',d['name'],d['objectIdField'],len(d['fields']))"
done            # → all three: CA_051220_WeeklyPatterns  OBJECTID_1  46

# ── Demand base + the missing spend column ───────────────────────────────────────────────────
curl -s "$B/CA_CensusTracts_2023/FeatureServer/0?f=json" | python3 -c "
import json,sys,re; d=json.load(sys.stdin); n=[f['name'] for f in d['fields']]
print('tracts fields:',len(n)); print('spend cols:',[x for x in n if re.search(r'RET|SPEN|SALES|DEMAND|LEAK',x,re.I)])"
                # → 164 fields · spend cols: []   ← there is NO consumer-spending column. Use BLS.
q "$B/CA_CensusTracts_2023/FeatureServer/0"          # 9107

# ── The $ multiplier (keyless) ───────────────────────────────────────────────────────────────
curl -s https://api.bls.gov/publicAPI/v1/timeseries/data/CXUFOODHOMELB0101M \
 | python3 -c "import json,sys;s=json.load(sys.stdin)['Results']['series'][0]['data'];print([(r['year'],r['value']) for r in s])"
                # → [('2024','6224'),('2023','6053'),('2022','5703')]

# ── The routing engine (keyless) ─────────────────────────────────────────────────────────────
curl -s "https://router.project-osrm.org/table/v1/driving/-118.2437,34.0522;-118.35,34.06;-118.15,34.10?sources=0&annotations=duration" \
 | python3 -c "import json,sys;print(json.load(sys.stdin)['durations'])"      # → [[0, 938.9, 800.3]]

# ── The two blocked/gated feeds — CONFIRM, do not design around a guess ──────────────────────
curl -s -o /dev/null -w "census acs: %{http_code}\n" -L "https://api.census.gov/data/2023/acs/acs5?get=NAME,B11001_001E&for=tract:*&in=state:06&in=county:037"
                # → 200 but the BODY is an HTML page titled "Missing Key"  ← key now required
curl -s -o /dev/null -w "data.ca.gov: %{http_code}\n" "https://data.ca.gov/api/3/action/package_search?q=taxable+sales"
                # → 403 (Cloudflare 1009)

# ── Frames ───────────────────────────────────────────────────────────────────────────────────
q "https://services1.arcgis.com/jUJYIo9tSA7EHvfZ/arcgis/rest/services/California_Incorporated_City_Boundaries/FeatureServer/0"   # 483
```

**Real field names that drive symbology / joins / KPIs.** SafeGraph slices: `safegraph_place_id`,
`location_name`, `brands`, `top_category`, `sub_category`, **`naics_code` (a STRING)**, `street_address`,
`city`, `postal_code` — **`objectIdField` is `OBJECTID_1`, not `OBJECTID`** (both columns exist; OID-keyed
selection on the wrong one silently addresses the wrong rows). Enriched tracts: `ID`, `COUNTY_NAME`,
`TOTHH_CY`, `TOTPOP_CY`, `MEDHINC_CY`, `MEDDI_CY`, `HINC0_CY`…`HINC200_CY`, `TOTHH_FY`. OSM: `shop`,
`name`, `brand`, **`brand:wikidata`**, `operator`. Cities: `CITY`, `COUNTY`. Counties: **`County`**
(mixed case) — not `POLYGON_NM`. Strata Serve Saudi: `nameen`/`namear`, `subtypeen`/`subtypear`,
`addressen`/`addressar`, `fkregionid`.

**Traps that bite (all observed, not assumed).**

| # | Trap |
|---|---|
| C1 | **`objectIdField` = `OBJECTID_1`** on all three SafeGraph slices, though an `OBJECTID` column also exists |
| C2 | **`naics_code` is `esriFieldTypeString`** — `naics_code = 445110` errors; use `LIKE '4451%'` or cast |
| C3 | **Every visit column is 19 Apr – 9 May 2020** (`Visits04_19_2020`…`Visits5_9_2020`, inconsistently zero-padded). Quarantine or drop — never a footfall claim |
| C4 | **`Retail` has no food service** (NAICS 722 → 0); it lives in `Restaurants`. Three services, one file |
| C5 | **The enriched tract layer has no spending column** — 164 fields, zero matching `RET\|SPEN\|SALES`. Any $ figure must come from BLS CEX and be printed as an assumption |
| C6 | **BLS CEX is per CONSUMER UNIT, nationally** — CU ≠ household, and it is not a California figure. v1 API caps at 25 queries/day: cache as a build constant |
| C7 | **OSRM is the public demo** — no SLA, driving-only, and a matrix is O(sources×destinations). Build-time, tiled, cached; self-host for production |
| C8 | **Overpass 504s on a full-county bbox** — tile by city |
| C9 | **OSM brand coverage ≈ 71 %** — state brand shares over *tagged* outlets, never over all |
| C10 | **`api.census.gov` now requires a key**; **`data.ca.gov` returns 403/1009** |
| C11 | **Tracts must be areally apportioned** into a catchment, never counted whole, or a straddling tract donates all its households to one side |
| C12 | **Caltrans AADT fields are Strings and every station is duplicated 2×**; geometry is wkid 2875 — request `outSR=4326` |
| C13 | **Strata Serve Saudi layers are `esriGeometryMultipoint`**, `displayField` is `namear`, and EN/AR are **separate layers** — `lang-switch` must swap `layerId` |
| C14 | **`CA_Counties` name column is `County`**, not `POLYGON_NM` — bind the wrong one and every group-by silently splits |

---

## Guided wizard — **the prompts that assign the app's defaults**

Claude runs this like an ArcGIS Instant-App wizard: ask each group, **apply the default so "accept all"
builds a complete app**, confirm a one-line summary, then run §5. Launch with
`/recipe marketing_catchment-and-market-share-analyzer`. Every answer *sets an application default* baked
into `layers.json` / the `AppLayout`.

| # | Wizard question | Options → **default** | The default it assigns |
|---|---|---|---|
| 1 · App | Title & as-of date? | free text → **"Catchment & Market-Share Analyzer"**, today | header title + `strata:notes.asOf` |
| 2 · Market | Which market? | **Long Beach–Torrance (CA demo)** · a CA county/city · Riyadh (Strata Serve) · my own extent | initial extent, which registers load, which frame layer |
| 3 · Category | What are you selling? | **Grocery** · convenience · fuel · restaurants · banking · commercial centres | the NAICS/`shop` rules behind each register chip |
| 4 · **Default register** | **Which register is the denominator on first render?** | **OSM (current, keyless)** · SafeGraph 445110 · SafeGraph 4451 · Big box · my own list | the chip selected at load — *and the app tells the user this is a choice* |
| 5 · Own network | Your outlets? | **sample synthetic** · CSV upload · my FeatureServer URL | the `owner='mine'` rows + the teal renderer |
| 6 · Sales book | Do you have sales or customers by store? | **no → position-only mode** · CSV upload · FeatureServer | whether `captured_pct`/`gap_pts` render live or greyed with "needs your sales file" |
| 7 · Catchments | Which definitions on the bench? `[multi]` | 3 mi · 5 mi · 10 min · 15 min · closer-to-me · city → **all six** | the `views` tabs and the size of the spread (registers × catchments) |
| 8 · Demand weight | Weight demand by? | **households** · households × BLS CEX $ · disposable income | the `hh`/`hh_usd` KPI pair and the derivation block |
| 9 · Routing | Travel-time engine? | **public OSRM (demo, rate-limited)** · self-hosted OSRM endpoint · straight-line only | whether `t_s_*` columns are network or haversine — stamped on screen either way |
| 10 · Census key | Have a Census API key? | **no → licensed Cal OES enrichment** · paste key → open ACS | which demand source binds, and which caveat prints |
| 11 · Report/Theme | Board report? · theme/language? | **board PDF + per-city atlas** ; **dark / EN** · light · EN+AR (RTL) | wires `/export report` + atlas; `ThemeSpec` mode + `lang-switch` |

**Then:** Claude echoes *"Long Beach–Torrance · grocery · OSM denominator · sample own network ·
position-only (no sales book) · all six catchments · households × BLS CEX · public OSRM · Cal OES
enrichment · board PDF · dark/EN"* and, on confirmation, runs §5 — so the app opens **fully configured**.

---

## 5. Prompt-script (run in order)

```
A. /new-app — a "Catchment & Market-Share Analyzer" open-design app ("ratio-bench", see
   DESIGN-PROPOSAL.md §3–§6). Dark ThemeSpec: primary #2dd4bf, secondary #94a3b8, warning #f59e0b,
   danger #f43f5e, app-bg #0b1220, fonts.scale compact, mono override on kpi, motion 180ms.
   App-shell: header (title + category/market pickers + lang-switch + theme-switch), footer
   (attribution + share), splash stating the thesis. Install deps + give me the run command.

B. Build `outlets` — the union of the registers WITH PROVENANCE. /convert each into EPSG:4326
   GeoParquet, adding a `register` + `register_label` + `vintage` column to every row:
     • OSM via Overpass, tiled by city (shop=supermarket), keep name/brand/brand:wikidata
     • Cal OES Retail naics_code='445110'   (mind OBJECTID_1, and naics_code is a STRING)
     • Cal OES Retail naics_code LIKE '4451%'
     • Cal OES Big_Box_Grocery_Stores       (City_1, not City; maxRecordCount 1000 — page it)
   DROP every Visits* / total_visits / daily_average column at convert time. Mark the client's own
   rows owner='mine' (sample synthetic if none supplied), all others owner='theirs'.
   /publish to Strata Serve (folder=california, service=catchment).

C. Build `demand`. /add-data Census TIGERweb block groups (layer 1, ACS-2025 vintage — RECORD the
   layer id) for the market; join Cal OES CA_CensusTracts_2023 TOTHH_CY/MEDDI_CY by GEOID and
   AREALLY APPORTION tracts to block groups. /analyze centroids → one point per block group.
   Then the travel columns, at BUILD TIME, tiled by city and cached:
     • OSRM /table from each block-group centroid to every outlet → t_s_mine (min over owner='mine')
       and t_s_best_comp (min over owner='theirs'); adv_s = t_s_mine − t_s_best_comp
     • haversine d_km_mine via /analyze nearest (the straight-line fallback + the ring catchments)
     • hh_usd = hh × 6224   (BLS CXUFOODHOMELB0101M, 2024 — store the series id and vintage as
       columns, not as a code constant, so the derivation block can print them)
   /publish.

D. Build `readings` + `brand_rollup`. For each of the 4 registers × 6 catchments compute
   mine_n, all_n, share_pct, hh, hh_usd, positioned_pct (share of catchment demand with adv_s<0),
   and — only if a sales book was supplied — captured_pct, gap_pts, gap_usd. Emit reading_label
   ("OSM supermarket · 10-min drive") and is_active. brand_rollup = outlets grouped by brand with
   share_pct, rank_n and share_alt (the same brand's share under NAICS 445110). /publish both.

E. /symbology + /popup — genuine ESRI JSON on verified fields only.
   outlets: uniqueValue on `owner` — mine esriSMSCircle #2dd4bf size 8, theirs esriSMSCircle
   #94a3b8 size 6 (NEVER esriSMSPath). demand: classBreaks on `adv_s`, diverging teal→amber→grey,
   fill alpha 40/255, break at 0 labelled "the draw line". frame: hollow outline.
   /popup outlets: title {name}; show brand, register_label, vintage, address — and NOT any Visits
   column. /popup demand: households, demand $, drive time to ours, to nearest competitor, advantage.

F. /panel statistics as the reading column: share_pct (headline), mine_n / all_n (the fraction),
   hh, hh_usd, positioned_pct (radial gauge, thresholds 25/50), captured_pct + gap_pts (greyed with
   "needs your sales file" when no book). All as kpi/gauge `stat:{field,op}` bound to sourceId
   `readings`/`demand` — no connections.

G. The bench + the spread.
   ① carto category widget on outlets.register_label (live counts) = the register chips.
   ② a `views` node, nav:"tabs", one view per catchment, each carrying
      mapState.definitionExpression on `demand`:
        3 mi → d_km_mine <= 4.828 · 5 mi → d_km_mine <= 8.047 · 10 min → t_s_mine <= 600
        15 min → t_s_mine <= 900 · closer-to-me → adv_s < 0 · city → city_name = '<city>'
      plus a one-paragraph `text` in each view saying what that definition ignores.
   ③ the spread: ship the `chart` kind:"bar" FALLBACK over `readings` (valueField share_pct,
      sorted asc, active reading emphasised). Register the app-local `spread-band` widget later.
   /panel table brand_rollup (brand, outlets_n, share_pct, rank_n, share_alt).

H. WIF + controls + export: author AppLayout.connections — the 12 in DESIGN-PROPOSAL.md §5, with
   the SIGNATURE LOOP first (spread categorySelect → filter on map + brand table + setUrlParam).
   Map controls navigation/scale/legend/basemapSwitcher/measure, position top-right so the native
   cluster clears the reading column; status-bar EPSG:4326. Wire /export report (board PDF:
   legend + scalebar + north-arrow + the PRINTED DERIVATION block + the spread table), a per-city
   atlas, /export image, /export layer csv, and a share deep-link. Verify responsive.small
   collapses the splitter and that the fraction card + spread band survive intact.
```

---

## 6. Verify (benchmark to Esri Business Analyst / Placer / Circana)

**Built and driven 2026-08-06** — `app/` (self-contained, MapLibre from CDN; `node server.mjs` → :8032).
**200+ assertions green** across `test-bench.mjs` (up to 114 — the arithmetic + the live services, per
category) and `test-render.mjs` (104 — the shipped page's wiring, its **23-behaviour interaction
surface**, and a headless drive of the render pipeline). **145 are deterministic** (offline subsets:
61 + 84). The live total moves by a few between runs: Overpass is a shared endpoint with no SLA, and
when it is down its registers downgrade to `⚠` warnings rather than failures — so quote **200+**, never
a fixed figure. Driven in headless Chrome via puppeteer-core: boot → both
readings → the signature loop → every interaction → clean console, zero failed requests.

**The interaction surface (23 wired behaviours; the design's floor is 3)** — full table in
`app/README.md`. The three that matter most, all verified live:

- **Set as ours.** Any brand in the table can be designated yours and the entire app rebuilds from it.
  Until you can say which operator is yours, the app is answering someone else's question — and the
  default is only ever a guess that *changes with the register* (7-Eleven under NAICS 4451, Vons under
  OSM). Switching also **discards any measured drive matrix**, because it was measured from the old
  outlets.
- **Hover a spread tick to preview it**, click to adopt, **← / → to step through the readings in share
  order** — one keypress per defensible answer.
- **Follow the map view.** Demand figures recompute for the visible extent (`extentChange →
  showStatistics`); zooming took *Households in view* from **400,381 (282 tracts) → 199,992 (143)**.
  The reading does not change and the label reads "in view", so the two can never be confused.
  **Analyse this view** goes further and re-runs the whole analysis for the current viewport —
  refusing a bbox above ~3,000 km², because Overpass will 504.

| Check | Pass |
|---|---|
| Four registers load from one normalised `outlets` shape; the register chip changes the market population 113 → 323 → 433 in place | ✅ live |
| The fraction card reads `6 of 9 = 66.7 %` (OSM · closer) and `63 of 404 = 15.6 %` (NAICS 4451 · 3 mi), recomputing on every bench change | ✅ live |
| The spread band shows a tick per resolvable reading (13 / 7 depending on register availability); **clicking a tick adopts that definition** across map + table + URL | ✅ driven — `66.7 % → 3.4 %`, URL followed |
| Six catchments each apply a real filter; "closer to us" is parameter-free and **self-selecting — the app cautions on the number itself** | ✅ |
| Advantage choropleth renders diverging with the break at **0 = the draw line**; fills at alpha ≈ 40/255; out-of-catchment tracts faded, not hidden | ✅ |
| Brand table's `same ground · other register` column shows the same brand under a second register **on identical tracts** | ✅ **Vons 1.7 % vs 7.3 % — 4.3×** |
| Without a sales book, capture and gap render **greyed with "needs your sales file"** — every other number still works | ✅ |
| Derivation prints the register rule + vintage, the catchment rule, the shared footprint, `TOTHH_CY` + source, and **BLS series id + vintage + the consumer-unit caveat** | ✅ 7 lines, asserted to quote the same numbers as the card |
| **No `Visits*` column reaches the client** — never requested, and a popup blocklist is exercised with a row that deliberately contains one | ✅ |
| Every field verified against the live service (§4); `naics_code` compared as a String; OID is `OBJECTID_1` on the SafeGraph slices | ✅ |
| Responsive collapse at 1100 px; fraction card + spread band survive intact | ✅ |
| Basemap keyless; EPSG:4326 throughout; **no write path anywhere**; path-traversal guard holds against encoded payloads | ✅ |
| Composed PDF / atlas export | ⛏ **not built** — `/export` belongs to the `<StrataApp>` path (see below) |
| Saudi binding swaps `layerId` on `lang-switch` with `dir="rtl"` | ⛏ **not built** — endpoints verified (§3), binding not wired in this app |

> **Scope of the build.** The `<StrataApp>` / `AppLayout` path is fully authored in §2 and
> `DESIGN-PROPOSAL.md` §4–§5 but is **unrendered in this checkout** — `strata/node_modules` is absent,
> so nothing can build the React engine. `app/` is the same design executed self-contained, which is
> the house pattern for a built recipe here. The two ⛏ rows above are the parts that live only on the
> `AppLayout` path.

**On-par-or-better:** matches Business Analyst's ring / drive-time trade-area vocabulary and Circana's
fair-share arithmetic, and **exceeds all of them on the one axis none of them can compete on — showing
the number under a rival's denominator** — plus the AI-authored build, the sovereign/on-prem posture,
cross-widget interactivity and the one-click board report, MIT, on Strata or ArcGIS. **Honestly less
than:** Business Analyst's calibrated Huff trade areas, Retail MarketPlace leakage/surplus and
15,000-variable enrichment; Placer's observed foot traffic and True Trade Area; Circana's measured
panel. We ship distance, measured drive time, proximity advantage and a **published** spend multiplier —
and we say so on screen.

---

## 7. Harvest (gaps → strata-core)

Log as strata-core issues:

- **A named `advantage` / `allocation` op** in `@strata/processing`. Today "nearest ours vs nearest
  theirs" is two `nearest` runs plus an app-side subtraction per demand unit. It deserves to be one
  registry tool (`allocate(demandFc, outletsFc, {ownerField, costFn})` → `adv`, `winner`) because it is
  the parameter-free heart of every catchment app and it is currently reinvented per recipe.
- **An OSRM `/table` matrix helper** in `@strata/plugin-routing` — tiling, caching and back-off are
  boilerplate every catchment/isochrone recipe rewrites. `fetchIsochrone` exists but needs a self-hosted
  Valhalla or a keyed ORS; a keyless matrix path is the one that actually ships.
- **`spread-band`** as a core widget (see `DESIGN-PROPOSAL.md` §9) — promote after a second use.
- **Areal apportionment** as a first-class join option (`spatialJoin(..., {apportion: 'area', fields})`).
  Counting whole tracts into a catchment is the most common silent error in this whole discipline and we
  should make the correct thing the easy thing.
- **A `ratio` KPI variant** — `kpi` with `stat: {numerator:{field,op}, denominator:{field,op}}` so a
  share is one widget with one provenance line rather than three widgets in a card.
- **`buttonClick` emitters** (Phase-2 pending) — the bench wants real segmented controls; `carto`
  category chips are a good stand-in but carry list semantics the design does not want.
- **Per-widget "assumption" affordance** — a first-class way to attach a printed derivation + source to
  any KPI. Three recipes now hand-roll it (`education_campus-operations`'s derivation card,
  `real-estate_site-selection`'s flip margin, this one's derivation block); it is a pattern, not a
  one-off.

Editing is not applicable — this app has **no write path by design**.

---

## 8. Sources

- **Esri (nominative use):**
  [Trade areas — Business Analyst](https://desktop.arcgis.com/en/arcmap/latest/extensions/business-analyst/trade-areas.htm) ·
  [Customer-derived trade areas: threshold methods](https://www.esri.com/arcgis-blog/products/bus-analyst/business/threshold-rings-drive-time-polygons-arcgis-business-analyst-pro) ·
  [Advanced customer-derived trade areas](https://www.esri.com/arcgis-blog/products/bus-analyst/analytics/advanced-cdta-options-business-analyst-pro) ·
  [How Huff Model works (ArcGIS Pro)](https://pro.arcgis.com/en/pro-app/3.5/tool-reference/business-analyst/understanding-huff-model.htm) ·
  [How the Original Huff Model works](https://desktop.arcgis.com/en/arcmap/10.3/tools/business-analyst-toolbox/how-original-huff-model-works.htm) ·
  [Choosing a POI data source in Business Analyst](https://www.esri.com/arcgis-blog/products/bus-analyst/business/choosing-poi-data-source-arcgis-business-analyst)
- **Non-Esri platforms:**
  [Placer.ai — Trade Area Analysis guide](https://www.placer.ai/guides/trade-area-analysis) ·
  [Placer True Trade Area API](https://docs.placer.ai/reference/post_v1-reports-true-trade-area) ·
  [Kalibrate — retail trade area analysis guide](https://kalibrate.com/insights/blog/the-ultimate-guide-to-retail-trade-area-analysis/) ·
  [SiteSeer — choosing the right trade area](https://www.siteseer.com/choosing-the-right-trade-area/) ·
  [GrowthFactor — trade area definition & methods](https://www.growthfactor.ai/resources/blog/what-is-a-trade-area) ·
  [GrowthFactor — retail site selection software comparison](https://www.growthfactor.ai/resources/blog/retail-site-selection-software) ·
  [SiteZeus vs Kalibrate](https://sitezeus.com/compare/kalibrate/)
- **Segmentation vocabularies:**
  [CACI Acorn](https://acorn.caci.co.uk/) · [Experian Mosaic](https://www.experian.co.uk/business/platforms/mosaic) ·
  [Mosaic (geodemography) — overview](https://en.wikipedia.org/wiki/Mosaic_(geodemography))
- **Fair share / market share method:**
  [Circana — fair-share measures](https://www.circana.com/post/5-ways-to-unleash-brand-growth-using-fair-share-measures) ·
  [Fair share gap analysis](https://www.cpgdatainsights.com/answer-business-questions/opportunity-gap-analysis/) ·
  [Category Development Index & Fair Share Index](https://blog.cmkg.org/blog/category-development-index-calculation_fair-share-index-calculation) ·
  [Market Penetration Index (MPI) and hotel fair share](https://insights.shijigroup.com/hotel-benchmarking-kpis-fair-share-and-more/)
- **The denominator problem, on the record:**
  [SafeGraph vs OpenStreetMap — the vendor's own comparison](https://www.safegraph.com/blog/comparing-safegraph-and-openstreetmap/) ·
  [OSM POI data for modelling urban change (PLOS One)](https://journals.plos.org/plosone/article?id=10.1371%2Fjournal.pone.0212606) ·
  [Conflating POI data: a systematic review of matching methods](https://www.sciencedirect.com/science/article/abs/pii/S0198971523000406)
- **Data endpoints (all curl-verified 2026-08-06 — see §4):** Cal OES hosted org
  `services.arcgis.com/BLN4oKB0N1YSgvY8` · CAL FIRE hosted org `services1.arcgis.com/jUJYIo9tSA7EHvfZ` ·
  Caltrans `caltrans-gis.dot.ca.gov` · CA State Geoportal `services.gis.ca.gov` ·
  [Overpass API](https://overpass-api.de/api/interpreter) · [Project OSRM](https://router.project-osrm.org) ·
  [BLS Public Data API](https://api.bls.gov/publicAPI/v1/timeseries/data/CXUFOODHOMELB0101M) ·
  [Census TIGERweb](https://tigerweb.geo.census.gov/arcgis/rest/services) ·
  [Census API key signup](https://api.census.gov/data/key_signup.html) (now required) ·
  [CDTFA data portal](https://cdtfa.ca.gov/dataportal/) (403 server-side)
- **Internal:** `DESIGN-PROPOSAL.md` (this recipe's full design record) · `catchment-catalog-ca.json` +
  `build_catchment_catalog.py` · `../APP-TEMPLATE-LIBRARY.md` · `../DESIGN-CONTEXT.md` ·
  `../../data_sources/data_sources_ca.md` · `../real-estate_site-selection/RECIPE.md` (format exemplar
  and the CDTFA/Huff rejections this recipe reconfirms) · `strata/recipes/COMPONENT-MANIFEST.md` (§10
  freestyle charter) · `strata/docs/guide/app-design.md`

---

## Modernization (parity release)

> Applies the Experience-Builder-parity features (Phases 1–7). See `strata/recipes/MODERNIZATION.md` +
> `COMPONENT-MANIFEST.md` §8. Cross-cutting: a structured **`theme`**, app-shell
> (`header`/`footer`/`splash`), and `dataSource.{ sourceId }` linking (widgets sharing a source link with
> **no `connections`**).

- **DataSource linking is the backbone, not a nicety.** Seven widgets — the fraction's three `kpi`s, the
  households/dollar pair, the `gauge` and the gap — bind to `sourceId` `readings`/`demand` and relink on
  every bench change with **zero** wiring. Without it this app would need ~20 more connections.
- **`views` + `mapState` as a control surface.** The catchment tabs are not navigation; each tab's
  `mapState.definitionExpression` **is** a catchment definition. This is the Phase-7 primitive doing
  analytical work, and it sidesteps the pending `viewChange` emitter entirely.
- **App-shell + `splash`.** The splash states the thesis in two sentences before the first render — this
  app's argument has to land before the numbers do.
- **Structured `ThemeSpec` with a widget override.** `overrides.kpi` puts the numerals in mono so the
  fraction and the spread ticks align to one grid — one config key doing real typographic work.
- **`FileDataSource`** is the client-book path: a CSV of sales by store turns `captured_pct` and
  `gap_pts` from greyed to live, with no schema change.
- **`splitter`** twice (reading↔map, derivation↔table) with `responsive.small` collapse.
- **Freestyle charter §10.2** invoked exactly once, for `spread-band`, with a named `chart` fallback that
  preserves the signature loop.
