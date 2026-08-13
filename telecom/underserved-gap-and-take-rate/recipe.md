# Recipe — Underserved Gap & Take Rate (Utilities, Energy & Telecom)

A reproducible path to a **broadband gap and take-rate** application on strata-core that answers the
question every "underserved" number hides: *"which gap is this — the one nobody can buy from, or the
one nobody buys from?"* It cross-tabulates the two published definitions of the same ground —
**availability** (who *can* buy, the definition public money follows) against **adoption** (who *does*
buy, the definition a build's payback follows) — on one clickable matrix of households, keeps the
blocks that **have no take-rate measurement at all** as a first-class fourth row, and prints the
derivation beside every number.

> **Scope (honest).** An **analytics and serving layer over published regulatory data** — not a system
> of record, not an FCC availability challenge tool, not a build-cost model, not a forecast.
> **The take rate is banded, never precise:** CPUC publishes adoption in six 20-point bands, so the
> app may say "≤ 60 %" and may never say "47 %". **Band 0 means "0 % *or* No Data"** — one code, two
> meanings — and is rendered ambiguously and excluded from headlines. **86 % of California's funded
> gap carries no take-rate record at all** (Fresno: 675 of 786 blocks), so the two layers are
> **outer**-joined and the absence is shown, never dropped. **Availability is provider-reported**
> (`MaxAdDn` is *advertised*); Ookla is offered as a second opinion and is **a sample of speed tests,
> not a census of subscribers**. **The authoritative federal source is unreachable** —
> `broadbandmap.fcc.gov` returns 403 to every non-browser client and its data API returns 401 — so
> this app runs on CPUC's California republication and says so on screen. **No subscriber-level
> data**: actual penetration needs the client's own book, supplied via `FileDataSource`; without it
> the app reports *position*, not *penetration*. **No write path at all**, so it runs identically on
> Strata and ArcGIS. Deploy **on-prem** where a subscriber book is bound.

Run the §5 prompt-script on a fresh strata-core project (or via `/new-app`). ESRI Web Map JSON is the
map contract; ingest is via `/convert` → `/publish` to Strata Serve.

---

## 0. Provenance

| | |
|---|---|
| **Source** | `https://tabaqat.net/solutions` — **Utilities, Energy & Telecom** section |
| **Name on site** | Underserved Gap & Take Rate |
| **Tagline** | "Where the gap is, which kind of gap it is, and who would actually subscribe" |
| **Researched** | 2026-08-09 |
| **Template** | **`open-design`** ("gap-matrix") — see `DESIGN-PROPOSAL.md` §2 for the three candidates and the anti-collision check |
| **Published** | `static/apps/telecom_underserved-gap-and-take-rate/` on strata-website, live at `/solutions/telecom/underserved-gap-and-take-rate`. Registered in `src/pages/apps.rs` (AppMeta) and `src/pages/categories.rs` (the telecom Sol, flipped from Soon to live). Sync the bundle with `./publish.sh` — the site serves a copy, so edits do not reach it otherwise. |
| **Presentation** | `presentation/index.html` — 10-slide deck (keyboard/click nav, print-to-PDF) · `presentation/linkedin-article.md` — ~1,050-word article + teaser post + a claims note listing every figure and its source |
| **Catalog** | `gap-catalog-ca.json` — 2,351 CA services inventoried, 292 role-tagged across 11 roles, 24 external feeds (19 verified-live). Rebuild with `python build_gap_catalog.py` |

---

## 1. Study — how the market frames this

**The question the buyer asks:** *"We can get funded to build in the unserved blocks. But the
households are in the served blocks where nobody subscribes. Which gap are we solving, and how many
customers does each one actually contain?"*

Everyone in this market has been handed a map of **availability** and asked to make a decision about
**revenue**. The industry says so itself: *"a home passed is a unit of infrastructure — it's not a
subscriber, not a satisfied customer and not a return on invested capital."* The US average fiber
take rate on primary passings is **46.5 %**, rising to about **61 %** where a second fiber provider
enters — so **more than half of a build's economics is decided after the build, by a variable the
availability map does not contain.**

### The finding this recipe is built on (verified live 2026-08-09, not asserted)

**California, EOY 2024 — two gaps, same ground:**

| Gap | Definition | Size |
|---|---|---|
| **Funded gap** (availability) | `Unserved` (2,019) + `Priority Unserved < 10/1` (32,027) | **34,046 blocks** of 390,743 — **8.7 %** |
| **Revenue gap** (adoption) | take rate **≤ 60 %** at the federal **100/20** standard | **80,429 blocks · 3,040,820 households** — **23.4 % of California households** |

**Joined block-by-block on `GEOID20`**, both layers fully paged, **outer** join (§4 reproduces it):

| | statewide blocks | statewide hh | Fresno blocks | Fresno hh |
|---|---|---|---|---|
| Funded gap (unserved, *with* an adoption record) | 4,405 | **142,732** | 111 | **3,818** |
| Revenue gap (served, take ≤ 60 % @ 100/20) | 76,146 | **2,904,667** | 2,473 | **66,645** |
| **ratio** | 17× | **20.4×** | 22× | **17.5×** |

> **Two ratios, one rule.** The figures above count **bands 0–3**, i.e. they include band 0
> (“0 % *or* no data”). The **app excludes band 0 by default** — the honest reading, since that band
> is ambiguous — which gives Fresno **16.8×** (2,357 blocks / 64,137 hh). Both are asserted; the app's
> `band 0 excluded / counted` toggle moves between them and prints the difference. Quote **16.8×** as
> the product's number and **17.5×** only alongside the rule that produced it.

**And the row nobody prints.** The availability layer has 390,743 blocks; the adoption layer has
264,137. **Of California's 34,046 unserved / priority-unserved blocks, only 4,405 carry a take-rate
record — 29,641 of them (87.1 %) are absent from the adoption map entirely.** Fresno reads 675 of 786
(86 %), so this is systemic, not local. The take-rate map is blind precisely where the money goes. An
**inner** join would report the funded gap as 4,405 blocks and be wrong by **7.7×**, silently. That
absence is the matrix's fourth row.

**The year-over-year move**, on identical geography (264,137 blocks, 12,993,375 households):
households in the ≤ 60 % bands at 100/20 fell **4,997,745 → 3,040,820**, **−39 %** in one year — in
the first full vintage measured *after* the Affordable Connectivity Program was withdrawn, which
should have pushed the other way. Real movement or a re-reporting artefact; the app shows it and
warns, and does not headline it.

### Reference solutions (benchmark + coexist, never copy)

- **Esri** — [ArcGIS Telecom Management](https://www.esri.com/en-us/industries/telecommunications/overview/telecom-management)
  and the [Bridging the Digital Divide](https://www.esri.com/en-us/industries/telecommunications/digital-divide)
  programme: a **State BEAD Allocations Dashboard**, a Broadband Outreach Solution, Business-Analyst
  enrichment, and
  [network-planning](https://www.esri.com/en-us/industries/telecommunications/digital-divide/arcgis-software-for-telecom-network-planning)
  / [asset-management](https://www.esri.com/en-us/industries/telecommunications/digital-divide/arcgis-software-for-telecom-asset-management)
  tooling — sold as an enterprise agreement at a fixed annual price by organization size.
- **Fiber GIS platforms** (the record systems a telco already owns) —
  **[VETRO FiberMap](https://www.capterra.com/p/193245/VETRO-FiberMap/)** (cloud-native fiber
  documentation, category leader), **[IQGeo](https://slashdot.org/software/comparison/3-GIS-vs-VETRO-FiberMap/)**
  Network Manager / ConnectMaster, **3-GIS**, **Render Networks** (design → permit → construction),
  **Comsof Fiber**, **Bentley OpenComms**. Every one answers *where is my cable*, not *who would buy
  from it*.
- **Availability & funding data** — **CostQuest** (the FCC's Fabric contractor), **LightBox**,
  **Ready.net / Broadband Money**, **Connected Nation**, **Tilson**, **Magellan Advisors**.
- **Measurement** — **Ookla** (Speedtest; the keyless open-data tiles this recipe uses), **M-Lab**,
  **BroadbandNow**.

### Our edge

**Nobody's product is the disagreement.** The plant vendors sell the record; CostQuest sells the
Fabric *and is the contractor for the availability map*, so it cannot be the tool that questions it;
Ookla sells measurement; Esri sells the platform. **The gap between the funded map and the revenue map
is a commercial embarrassment to at least one vendor in every pairing** — which is exactly why an
open, MIT, no-data-to-defend tool can hold both up at once. Second: **the authoritative source is
unreachable and we say so on screen** (§3). Plus the standing edges — AI-authored, runs on **Strata
*or* ArcGIS**, sovereign/on-prem so a subscriber book never leaves the perimeter, cross-widget
interactivity on the first build.

### Standards, programmes & organizations to speak fluently

- **BEAD** — $42.45 B, allocated on **unserved and underserved BSL counts** from the FCC maps. The
  **June 11 2025 Restructuring Policy Notice** removed the fiber preference and forced a
  technology-neutral **"Benefit of the Bargain"** round; by May 2026 **54 of 56** eligible entities had
  Final Proposal approval, with construction from mid-2026.
- **The thresholds.** *Unserved* = no reliable service at **25/3**; *underserved* = 25/3 but below
  **100/20**. California adds its own stricter class, **"Priority Unserved < 10/1"** — which is why a
  California gap count never equals a federal one.
- **FCC BDC · National Broadband Map · the CostQuest Fabric** and its **availability challenge
  process** — widely criticised for overstating availability.
- **RDOF** — the 2020 reverse auction. An award is a *claim* on a location; defaults are common.
- **CASF**, the **Last Mile Federal Funding Account**, and the statewide **Middle-Mile Broadband
  Initiative** — California's own programmes and eligibility map.
- **Take rate · penetration · homes passed · ARPU** — the operator's vocabulary.
- **ACS B28002** — *Presence and Types of Internet Subscriptions in Household*; the survey adoption
  measure. **Now key-gated** (§3).
- **The ACP cliff** — ended **1 June 2024** with 23 M+ households enrolled; **13 %** of recipients had
  cancelled within weeks. The essential context for reading EOY 2024 bands.
- **ESRI Web Map / `drawingInfo` / `popupInfo` JSON** — this repo's rendering contract.

**Honest scope** — see the blockquote at the top; repeated in the app's `splash` and footer.

---

## 2. UI design spec (front-loaded)

### 2.1 Layout (Template: `open-design` — "gap-matrix")

- **Template `open-design`** under the freestyle charter (`strata/recipes/COMPONENT-MANIFEST.md` §10).
  Full derivation, three candidate silhouettes and the anti-collision check: **`DESIGN-PROPOSAL.md`**.
- **Why not a library template:** the sector's six neighbours already hold `insets-grid`,
  `ranked-list`, `chart-board`, `triage-console`, `sidebar-explorer` and an `open-design` voltage
  ladder. More to the point, **the subject of this app is a disagreement between two
  classifications**, and the correct instrument for that is a cross-tab — which no template in the
  library is.
- **The silhouette:** a **4 × 6 matrix of households** is the interface — availability class down
  (plus a fourth row for *no take-rate record*), CPUC's six adoption bands across, **cell area
  proportional to households**. The matrix is simultaneously the navigation, the legend and the
  argument.
- **Signature loop:** **click any cell → the whole app adopts that population** — the map repaints to
  exactly those blocks, the reading recomputes, the provider list and block table re-scope, and the
  URL updates.
- **Wiring floor:** the design authors **14 `connections`** (the guideline's floor is 3) plus 9
  widgets linked by a shared `sourceId`. **The built app ships 41 wired behaviours** — full table in
  `app/README.md`.

**The design skeleton is in `DESIGN-PROPOSAL.md` §3.** What shipped differs from it in four ways,
each recorded here because the difference was learned by building, not by planning:

| Designed | Built | Why |
|---|---|---|
| Scope controls in the header | Header **strip** — `Where · "Adopted" means · Vintage · Band 0` | They were briefly moved into the band, which left a wide gap beside the matrix and wrapped their labels four lines deep. The matrix now owns the band alone and stretches to fill it. |
| Reading rail beside the map only | Rail runs **header → footer**; band, map and table stack beside it | Sandwiched between a full-width band and a full-width table, the rail clipped at ~half a screen and its lower cards never surfaced. |
| One draggable gutter | **Three** — rail width, band height, table height | Persisted to `localStorage`, double-click to reset, clamped so nothing collapses. |
| A `search` widget in the header | **Not built** | Nominatim geocoding adds a second host and a second failure mode for a question always asked by county. Dropped rather than shipped half-wired. |

**As built** (Fresno County, EOY 2024, 100/20 — the real figures):

```
┌ HEADER  Underserved Gap & Take Rate · CPUC EOY 2024 · 13,514 blocks · 298,608 households ───────┐
│  bands are 20 points wide; band 0 = "0% or no data"                                             │
│           Where [Fresno ▾]  "Adopted" [100/20 ▾]  Vintage [EOY 2024 ▾]  [band 0 excluded]       │
│           ·· the federal standard — the only tier that discriminates ··   [Export CSV]  [☀]     │
├ THE MATRIX ── stretches the full width · cell area ∝ households · the app's navigation ─────────┤
│                     0 / ? ⚠     ≤20%      ≤40%      ≤60%      ≤80%      >80%                    │
│  Served              2,508    11,742    16,316    36,079    82,105   146,040                    │
│  Unserved              119       298       103       108         ·        17                    │
│  Priority Unserved   3,038        50        85         ·         ·         ·                    │
│  No take-rate ⚠   ███████ 5,063 blocks — no take-rate published at all ███████                  │
│  hover = preview · click = ADOPT · ← → step · Esc clear                                         │
├═════════════════════════════ drag to resize ════════════════════════════┬───────────────────────┤
│  MAP                                        ┌─┐ + − ⌂ ▤ L ◈ B ☰ G       │  THE ADOPTED CELL     │
│   ▒ blocks shaded by the ACTIVE CELL                                    │   298,608 households  │
│     blue = can't buy · violet = won't buy · amber = nobody measured     │                       │
│   ▨ RDOF · ▧ CASF grants (on) · ━ middle mile (on) · ● anchors          │  FUNDED   REVENUE     │
│  ┌ ROWS ON THE MAP ─────────────────────┐                               │   3,818    64,137     │
│  │ Served              8,340 of 12,728  │                               │   16.8× · different   │
│  │ Unserved               22 of     83  │                               │     places            │
│  │ Priority Unserved      89 of    703  │                               │                       │
│  │ No take-rate record          5,063 ⚠ │                               │  ◔ 85.9 % unmeasured  │
│  └──────────────────────────────────────┘        −119.8, 36.7 · z9 · 4326│  ▸ Board paragraph    │
├═════════════════════════════ drag to resize ════════════════════════════┤  ▸ Derivation         │
│  BLOCK TABLE — geoid · can buy · does buy · hh · pop · block            │  ▸ Providers ▸ Money  │
│                       amber row = no take-rate record                   │  [block detail]       │
└─────────────────────────────────────────────────────────────────────────┴───────────────────────┘
                                                        ▲ drag ─ rail width, persisted             
```

**Map chrome.** MapLibre's native zoom cluster is hidden and replaced with one top-right cluster —
zoom in · zoom out · **fit to the adopted cell** · **L**ayers · **B**asemap · Le**g**end — drawers
opening beside it, never over it. Same vocabulary as `industry_mining-and-concession-compliance`.

**The legend is a control surface, not a caption.** Each availability row toggles or isolates its
class and carries a live count; the `no take-rate record` row is the one most users click first, which
is the point. Isolating a class deliberately does **not** change the adopted cell or the reading, and
a hint line says so.

**Resizable, and remembered.** Three splitters — **rail width · band height · table height** — each
driving one CSS variable, each persisted to `localStorage`, each **double-click to reset**, all
clamped so no region can be collapsed to nothing. Each splitter is also the *only* divider between
the two regions it separates (the band has no bottom border, the table no top border, the rail no
left border), so nothing has a gap or a doubled line.

**Responsive.** Below 1240 px the scope strip wraps onto its own line rather than squeezing the title;
below 1100 px the body collapses to a column and the splitters hide. **The matrix survives intact** —
it scrolls inside its band and never re-flows into a list, because a matrix that has become a list is
no longer an argument. The map drops to 55 vh; the reading rail follows it; the block table goes last.

### 2.2 Theme

Structured `ThemeSpec`, **`mode: "light"`**. **Two hues carry the whole argument: `primary #1d4ed8`
(blue) is "*can* they buy" — availability, infrastructure, the funded gap; `secondary #7c3aed`
(violet) is "*do* they buy" — adoption, take rate, the revenue gap.** No element may use one to mean
the other, so a reader learns the distinction from colour before reading a label. `warning #f59e0b` is
reserved for **absence** rather than danger (the no-record row and the ambiguous band 0), so the
largest cell in the matrix reads as a hole, not an alarm. `success #059669` = the resolved >80 % cell;
`danger #dc2626` held back for a filing contradicted by measurement.

`fonts.scale "compact"` · **mono override on `kpi` and `table`** with `tabular-nums` so household
counts align to one grid · motion **180 ms**, short and mechanical, because every animation here is a
measurement changing.

⚠ **Amber needs two values, not one.** As a fill over a pale basemap `#f59e0b` is right; as *text* on
a white panel it measures **2.15:1**, well under WCAG AA's 4.5 — and amber marks this app's biggest
finding. The built theme therefore splits the token: `--strata-warning` stays the fill (and the gauge
stroke), `--strata-warning-text: #b45309` (**5.02:1**) carries every text use, still unmistakably the
same hue (ΔE 35 from the fill, ΔE 129 from the blue). Dark mode needs no split — `#fbbf24` on the dark
panel is 10.20:1.

**Why light, deliberately:** the artefact is a **counter-document** that leaves as a PDF in a white
grant application or board packet; the matrix depends on **area comparison across 24 very
different-sized cells**, which low-alpha fills over a dark basemap destroy; and every persona
navigates by place name, which a dark basemap under a block choropleth makes unreadable. **Dark stays
first-class** via `theme-switch` (a broadband office does run this on a wall), and an explicitly
chosen basemap survives a theme swap.

**Measured, 2026-08-09** — the choice was originally made on reasoning alone, so it was checked.
WCAG contrast and CIE76 ΔE over the real fill alphas and a sampled CARTO basemap:

| | light | dark |
|---|---|---|
| **blue vs violet on the map** — the load-bearing distinction | **ΔE 17.3** | ΔE 12.5 |
| amber "nobody measured" vs basemap | ΔE 36.3 | **ΔE 49.5** |
| body / muted text on panel | 17.85 / 4.76 | 13.81 / 6.64 |
| amber as TEXT on panel | **2.15 ✗ AA** → fixed to 5.02 | 10.20 ✓ |

Light wins the measurement that matters most: **the two-hue semantic separates 38 % better**
(ΔE 17.3 vs 12.5), and that distinction is the entire argument. Dark wins on amber — both as text
and on the map — which is why it stays first-class rather than decorative.

The measurement also found a real defect: **`#f59e0b` as text on a white panel is 2.15:1, well under
AA's 4.5**, and amber marks this app's single biggest finding (87 % unmeasured). Fixed by splitting
the token — `--strata-warning` stays the bright fill, `--strata-warning-text: #b45309` (5.02:1) is
used for every text use, still unmistakably the same warning hue (ΔE 35 from the fill, ΔE 129 from
the blue). Dark mode needs no split.

Basemap **keyless**: CARTO Positron (light) / Dark Matter (dark) from `OPEN_BASEMAPS`, paired via
`basemapForTheme`. Block fills alpha ≈ 40/255. **EPSG:4326 throughout** — every CPUC service is
`wkid 102100`, so `outSR=4326` on ingest.

**i18n — honest.** EN + AR/RTL is wired and works, but for this audience it is **decorative and the
recipe says so**: California's adoption gap correlates with Spanish-speaking households and
`@strata/i18n` ships `en`/`ar` only. An `es` dictionary is a one-file change and is logged in §7.

### 2.3 KPI cards

Three, all live `kpi`/`gauge` `stat: {field, op}` bound to the shared `reading` source, so they
recompute on every cell adoption **with no `connections`**:

**Households in the adopted cell** (the headline, with its plain-English definition beneath) ·
**Funded gap** (`hh_funded_gap`, blue) · **Revenue gap** (`hh_revenue_gap`, violet) — with the ratio
between them as a callout — and a radial `gauge` for **share of the funded gap with no take-rate
record** (thresholds 25/60, amber), which in Fresno reads **85.9 %**.

Two further metrics — **actual penetration** and **fair-share gap** — render **greyed with an explicit
"needs your subscriber file"** note until a client book is bound via `FileDataSource`.

### 2.4 Charts & table

- **The matrix** — the signature. A new `gap-matrix` widget (§10.3 block in `DESIGN-PROPOSAL.md` §9)
  plotting the 24 cells with area ∝ households; **named fallback: a `table`** over the same `matrix`
  source whose `rowSelect` carries the identical payload, so **the signature loop survives unchanged**.
  **Ship the fallback on day one.** A `stacked-bar` (4 series × 6 bands) is the visual stand-in.
- **Block table** — `AttributeTablePanel` over `blocks`: `geoid` · `status` · `band_label` · `hh` ·
  `provider_n` · `casf_elig` · `rdof`. Sortable, per-column filter, server-paged, row → `zoomTo` +
  `flash`, CSV/GeoJSON export. **Rows with `has_adoption == false` are tinted** — the absence is
  visible in the table too, not only the matrix.
- **One chart only**, in the second-opinion accordion: advertised vs Ookla-measured. **No chart rail** —
  that would collide with `utilities_asset-condition-and-risk-register`.

### 2.5 Capabilities to use (Phases 0–7)

- **`/analyze`** at build time: `spatialJoin` (providers × blocks, subsidy × blocks), `buffer`
  (distance-to-middle-mile bands), `dissolve`/`aggregate` (block → county/place/district roll-ups).
  **`hexbin`/`hotspot` rejected** — density answers "where are they clustered"; the question here is a
  *classification*. **`weighted-overlay` rejected** — it would produce a composite "priority score",
  the invented number this app refuses.
- **WIF `connections`** — 14 (`DESIGN-PROPOSAL.md` §5), signature loop first.
- **DataSource linking** — 9 widgets on shared `sourceId`s (`reading`/`providers`/`subsidy`/
  `measured`); without it the connections table would be ~34 rows.
- **`arcade`** — one `valueExpression` on the block renderer deriving `cell_key` from `status` + band.
- **Deliberately NOT used:** `filter`/`query` builders (a free-form builder lets a user construct a
  gap definition the app cannot then price or defend), `add-data` (pointed at the Drilldown service it
  would try to enumerate **855 layers** — trap G9), `timeslider`/`date-filter` (two vintages is not a
  series), `swipe` (it compares two views of *the same thing*; these vintages differ by a renamed
  field and a possible re-report), `routing`/`isochrones` (a service area is a wire footprint, not a
  drive time), `draw`/`measure`/`near-me` (the unit of analysis is the census block, fixed), `views`
  (the matrix cell *is* the navigation), editing (**no write path at all**). Full sweep:
  `DESIGN-PROPOSAL.md` §8.
- **Composed export** — a board **PDF** (legend + scalebar + north-arrow + the printed derivation +
  the matrix as a table), a per-county **atlas**, a per-cell **feature report**, CSV/GeoJSON, and
  `exportSpec` so the reading round-trips to ArcGIS. **Built today:** CSV of the current scope, with
  the banding caveat in the file's own header comment, plus a **board-ready paragraph** with a copy
  button. The PDF/atlas belong to the `<StrataApp>` path and are not built.
- **Built additions the design did not call for**, each earned by using the thing: the rail's KPIs are
  **navigation** (click *funded gap*, *revenue gap* or the *unmeasured* gauge to map that population);
  a **map click rings the cell you are standing in** so the loop runs both ways; and the layer panel
  names each layer's real state — **off · N in view · none in this view · capped · not mappable**.

---

## 3. Data sources

All EPSG:4326 (reproject on ingest — every CPUC service is `wkid 102100`). Every row below was
`curl`'d on **2026-08-09**; counts are literal `returnCountOnly` / `outStatistics` responses. Full
provenance, schemas and traps: **`gap-catalog-ca.json`** (24 external feeds, 19 verified-live).

**The discovery.** California's public ArcGIS estate — all **2,569 endpoints** in
`data_sources_ca.md` — contains **zero** broadband availability layers, **zero** adoption layers and
**zero** provider registers. The four telecom-adjacent hits are a cellular-licence × county overlay
and public-safety radio towers. But the **California Interactive Broadband Map**
(`broadbandmap.ca.gov`) is a Leaflet app that loads **esri-leaflet**, and its `config.js` points at an
**eighth California ArcGIS server that is not in the catalogue**:

> **`https://cpuc2016.westus.cloudapp.azure.com/arcgis/rest/services/CPUC`** — **36 services**,
> `Map,Query,Data`, **CORS open** (`Access-Control-Allow-Origin` reflects the Origin — browser-direct,
> no proxy). Adoption **and** availability, at two vintages, from one authority, on one join key.

| Role | California (CPUC — the discovered server) | California (public estate) | National / engine |
|---|---|---|---|
| **Adoption — the take rate** | `CPUC_EOY_2024_Broadband_Adoption_100_20/0` — **264,137** blocks, `ACAT_10020` (6 bands) · `..._25_3/0` `ACAT_253_New` · `..._Any/0` `ACAT_ANY` · `CPUC_EOY_2023_..._100_20/0` `ACAT_100_20` (prior vintage) | ⛔ **none** | ⛔ **Census ACS B28002 key-gated** — bare call returns HTML titled *Missing Key* |
| **Availability — the funded gap** | `CPUC_EOY_2024_Fixed_Consumer_Served_Status/0` — **390,743** blocks; `Status` ∈ Served **356,697** / Unserved **2,019** / Priority Unserved <10/1 **32,027** · `..._Downstream_Maximum/0` **362,474** (`MAX_MaxAdDn`) | ⛔ **none** | ⛔ **FCC NBM 403 / data API 401**; only reachable FCC series is **Form 477 June 2021** (retired, over-reporting) |
| **Serviceable locations (BSL)** | `CPUC_EOY_2024_CASF_Infrastructure_Eligibility/0` — **215,804** points; **29,325 eligible** (`CASF_ELIG=1`) | 54 wildfire **parcel** layers (a coarse premise proxy) | ⛔ CostQuest Fabric — licensed |
| **Provider register** | `CPUC_EOY_2024_Provider_Identify/0` — **1,685** polygons, **182 distinct `DBA`**, `TechCode`/`Service_Type`/`MaxAdDn`/`MaxAdUp` | ⛔ none (Cal OES `Cellular_Service_Areas_by_County` = 1,409 cellular *licence* areas, no speed/tech/date) | — |
| **Subsidy footprint** | `CPUC_EOY_2024_Broadband_Grants` (5 layers: 41/**87**/25/8/15) · `CPUC_RDOF_Phase_I_Winners/0` — **264** areas, **26,240 locations, $4,552,860.92** · `CPUC_CDT_Middle_Mile_Network/0` (**1 row**, whole state) | 10 unrelated grant layers | BEAD awards — state-published, changing through 2026 |
| **Demand base** | `..._Political_Boundaries/3` Blocks · `HH_2020`/`HU_2020`/`POP_2020` on every adoption layer | 26 census/demographic layers | **Census TIGERweb** `Tracts_Blocks/MapServer` — keyless, blocks at **2** and **12** |
| **Affordability** | `CPUC_EOY_2024_Median_HH_Income/0` — **25,586** block groups, `MHI_Int`, `PCAP_INC`, `FedPovLvl` | `Disadvantaged_Communities_CES4`, `CA_SVI_2022_tract`, `Low_income_Communities` | ⛔ ACS key-gated |
| **Anchor demand** | `CPUC_EOY_2024_Community_Anchor_Institutions/0` — **100,374** anchors, `CAICAT` | CSCD campuses, CDE schools | — |
| **Reporting frame** | `CPUC_EOY_2024_Political_Boundaries` — **13 layers**: 0 Counties (**58**, `NAME20`) · 4 Places · 5 Urban Areas · **6 Tribal Lands (117)** · **7 Assembly · 8 Senate · 9 Congressional** · 10 School Districts · 12 Zip | 93 frame layers | — |
| **Measured speed — the second opinion** | ⛔ none | ⛔ none | **Ookla Open Data** `ookla-open-data.s3.amazonaws.com/parquet/performance/type=fixed/` — keyless, **current to 2026 Q2**, ~370 MB/quarter (build-time only) |
| **Bulk refresh** | **CPUC Annual Collected Broadband Data** — FGDB/SHP, wireline + fixed wireless, as of 2024-12-31 (posted 2026-04-01) | ⛔ `data.ca.gov` CKAN **403 / Cloudflare 1009** on `q=broadband` | — |

**CORS:** the CPUC server reflects the request Origin; Cal OES/CAL FIRE hosted orgs are `*`; TIGERweb
and the Ookla S3 bucket are open. **Browser-direct, no proxy needed.** Carry CPUC, US Census, Ookla
and OpenStreetMap attribution plus "no warranty", and state on screen that **availability is
provider-reported and adoption is banded**.

### The two gaps, measured

| | blocks | households |
|---|---|---|
| **Statewide funded gap** (Unserved + Priority Unserved) | **34,046** of 390,743 | — |
| **Statewide revenue gap** (take ≤ 60 % @ 100/20) | **80,429** of 264,137 | **3,040,820** of 12,993,375 |
| **Statewide, joined:** funded gap *with* a take-rate record | 4,405 | **142,732** |
| **Statewide, joined:** revenue gap (served, take ≤ 60 %) | 76,146 | **2,904,667** — **20.4×** |
| **Statewide funded-gap blocks with NO adoption record** | **29,641 of 34,046 (87.1 %)** | — |
| **Fresno funded gap** (with an adoption record) | 111 | **3,818** |
| **Fresno revenue gap** (bands 0–3, i.e. counting the ambiguous band 0) | 2,473 | **66,645** — **17.5×** |
| **Fresno revenue gap** (bands 1–3 — the app's default, band 0 excluded) | 2,357 | **64,137** — **16.8×** |
| **Fresno funded-gap blocks with NO adoption record** | **675 of 786 (86 %)** | — |

**Adoption bands, statewide (EOY 2024, households):**

| band | 0 · "0 % or No Data" | 1 · ≤20 % | 2 · ≤40 % | 3 · ≤60 % | 4 · ≤80 % | 5 · >80 % |
|---|---|---|---|---|---|---|
| **@ 100/20** | 220,564 | 332,266 | 826,497 | 1,661,493 | 2,645,340 | 7,307,215 |
| **@ 25/3** | 112,154 | 62,636 | 78,816 | 139,880 | 302,775 | **12,297,114** |

⚠️ **The 25/3 service and the "Any speed" service are the same data under two names** — band-for-band
identical. And at 25/3 the state is **94.6 % saturated**, so only the 100/20 tier discriminates: that
is why it is the app's default.

**Year over year @ 100/20 (households in bands 0–3):** EOY 2023 **4,997,745** → EOY 2024
**3,040,820** (**−39 %**), on identical geography — but the field was renamed and the move lands in
the window when the ACP ended.

### Traps that bite (all reproduced, not inferred)

| # | Trap |
|---|---|
| **G1** | **`Status` is filterable but not selectable.** It drives the Served Status renderer and works in a WHERE (`Status='Unserved'` → 2,019) but is **absent from `fields[]`**, so `outFields=Status` returns **HTTP 400**. Page one query per class. Match the string exactly, spaces and all. |
| **G2** | **Same hidden-field class, asymmetric, on CASF eligibility.** Renderer field `CASF_ELIG` filters fine (1 → 29,325 · 0 → 186,479); displayField `bsl_flag` returns **400**. Test every hidden field before designing a control on it. |
| **G3** | **Band 0 = "0 % *or* No Data"** on every `ACAT_*` field. One code, two meanings. Never colour it as a gap without saying so. |
| **G4** | **`Broadband_Adoption_Any` ≡ `Broadband_Adoption_25_3`** — identical bands. Two names, one measurement. |
| **G5** | **The adoption field is renamed between vintages**: `ACAT_100_20` (2023) → `ACAT_10020` (2024); the 25/3 one carries a `_New` suffix. A hard-coded name returns an empty groupBy **with no error** — which reads as "no change". |
| **G6** | **Three block universes on one server:** 390,743 · 264,137 · 362,474. An inner join silently discards **5,063 blocks in Fresno, 86 % of its funded gap**. **Outer-join, always.** |
| **G7** | **`objectIdField` is `null` on every CPUC layer** — no OID-keyed selection anywhere. Bind read-only; select on `GEOID20`. |
| **G8** | **The block key is `BlockCode`, not `GEOID20`, on `Fixed_Consumer_Downstream_Maximum`** — the only layer that renames it. A `GEOID20` join matches zero rows and reports success. |
| **G9** | **855 layers in `..._Drilldown/MapServer`** (13 `TRANSTECH_*`, 181 `<Provider>_<Tech>_PURPLEDOWN`, 126 providers). Never enumerate it; use `Provider_Identify` (1,685 rows). |
| **G10** | **`documentInfo.Title` is wrong on every service** — all report `CPUC_Fixed_Consumer_Downstream_Maximum_Dec20201`, a stale MXD name, and `editingInfo.lastEditDate` is absent. **Only the service NAME carries the vintage.** |
| **G11** | **Alias disagreement:** on `..._Adoption_Any`, `POP2023` *and* `HH2023` both carry the alias "Households 2020". Trust field names, not aliases; prefer `HH_2020`/`POP_2020`. |
| **G12** | **Everything is `wkid 102100`.** Request `outSR=4326`; never quote `Shape_Length` as distance (it is Web-Mercator metres — see the pipeline recipe's P2). |
| **G13** | **Cal OES `Tower`'s OID is `OID`, not `OBJECTID`**, and `Anticipated Tower` has a **literal space** in the service name needing percent-encoding. These are **CRIS public-safety** towers, not commercial masts — never present them as coverage. |
| **G14** | **`maxRecordCount` 2,000 on every CPUC layer**; an un-paged query returns exactly 2,000 with no error. For a gap count that **understates the gap**. Page with `resultOffset`; check `exceededTransferLimit`. |
| **G15** | **Ookla is ~370 MB of parquet per quarter** — build-time only — and is **a sample of speed tests, not a census of subscribers**, self-selected toward people who suspect their connection is bad. |
| **G17** | **A point layer that never returns points.** `CPUC_EOY_2024_CASF_Infrastructure_Eligibility` declares `geometryType: esriGeometryPoint` and answers count queries correctly (**29,325** eligible), but **`returnGeometry=true` yields features with no geometry under every parameter combination tested** — with/without `outSR`, `outFields=*`, `outFields=OBJECTID`, bbox and no bbox, `f=json` and `f=geojson`. So the 215,804 broadband-serviceable locations — the unit BEAD and CASF money is counted in — are **countable but not mappable** from this service. Mark it undrawable in any layer list; a checkbox that ticks and paints nothing reads as a broken app rather than a withheld coordinate. |
| **G16** | **CAL FIRE names a service per wildfire** (`FS_<INCIDENT>_<YEAR>_<UNIT>_<N>`), so `\btower\b` matches the 2026 *Tower Fire* and `\briver\b` the 2026 *River Fire*. Match the naming signature, not a word blocklist — the next fire called "Fiber" walks straight in. |

---

## 4. Verify each URL first (terminal)

```bash
UA='Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 Chrome/126.0 Safari/537.36'
B=https://cpuc2016.westus.cloudapp.azure.com/arcgis/rest/services/CPUC

# ── THE DISCOVERY: an eighth CA ArcGIS server, found in the state broadband map's config.js ──
curl -s https://www.broadbandmap.ca.gov/config.js | grep -o 'https://cpuc2016[^"]*' | sort -u
curl -s -A "$UA" "$B?f=json" | python -c "import json,sys;print(len(json.load(sys.stdin)['services']),'services')"
                                                                        # → 36 services

# ── CORS: decisive — can a browser reach it at all? ──────────────────────────────────────────
curl -s -I -A "$UA" -H "Origin: https://example.org" \
  "$B/CPUC_EOY_2024_Broadband_Adoption_100_20/MapServer?f=json" | grep -i access-control
                                              # → Access-Control-Allow-Origin: https://example.org

# ── AVAILABILITY: the funded gap. NOTE `Status` filters but is NOT in fields[] (trap G1) ─────
q(){ curl -s -A "$UA" -G "$1/query" --data-urlencode "where=$2" \
     --data-urlencode "returnCountOnly=true" --data-urlencode "f=json"; }
S=$B/CPUC_EOY_2024_Fixed_Consumer_Served_Status/MapServer/0
q "$S" "1=1"                                                    # 390743
q "$S" "Status='Served'"                                        # 356697
q "$S" "Status='Unserved'"                                      #   2019
q "$S" "Status='Priority Unserved < 10 Mbps Down / 1 Mbps Up'"  #  32027
curl -s -A "$UA" -G "$S/query" --data-urlencode "where=1=1" \
     --data-urlencode "outFields=GEOID20,Status" --data-urlencode "f=json" | head -c 120
                            # → {"error":{"code":400 ...}}  ← Status is NOT selectable. G1.

# ── ADOPTION: the take rate, in 20-point bands. Band 0 = "0% OR NO DATA" (trap G3) ───────────
A=$B/CPUC_EOY_2024_Broadband_Adoption_100_20/MapServer/0
curl -s -A "$UA" -G "$A/query" --data-urlencode "where=1=1" \
 --data-urlencode "groupByFieldsForStatistics=ACAT_10020" \
 --data-urlencode 'outStatistics=[{"statisticType":"count","onStatisticField":"GEOID20","outStatisticFieldName":"blocks"},{"statisticType":"sum","onStatisticField":"HH_2020","outStatisticFieldName":"hh"}]' \
 --data-urlencode "orderByFields=ACAT_10020" --data-urlencode "returnGeometry=false" \
 --data-urlencode "f=json"
 # → 0:7838/220564 · 1:9459/332266 · 2:22212/826497 · 3:40920/1661493 · 4:64450/2645340
 #   5:119258/7307215      TOTAL 264,137 blocks / 12,993,375 hh

# ── PROVE the 25/3 and "Any" services are ONE dataset (trap G4) ──────────────────────────────
for s in CPUC_EOY_2024_Broadband_Adoption_25_3:ACAT_253_New \
         CPUC_EOY_2024_Broadband_Adoption_Any:ACAT_ANY; do
  curl -s -A "$UA" -G "$B/${s%%:*}/MapServer/0/query" --data-urlencode "where=1=1" \
   --data-urlencode "groupByFieldsForStatistics=${s##*:}" \
   --data-urlencode 'outStatistics=[{"statisticType":"count","onStatisticField":"GEOID20","outStatisticFieldName":"n"}]' \
   --data-urlencode "orderByFields=${s##*:}" --data-urlencode "f=json"
done   # → both: 3194 / 1879 / 2572 / 4388 / 9423 / 242681   ← identical. One file, two names.

# ── THE PRIOR VINTAGE: note the RENAMED field (trap G5) ──────────────────────────────────────
curl -s -A "$UA" "$B/CPUC_EOY_2023_Broadband_Adoption_100_20/MapServer/0?f=json" \
 | python -c "import json,sys;print([f['name'] for f in json.load(sys.stdin)['fields']])"
                            # → ...'ACAT_100_20'   NOT 'ACAT_10020'. A hard-coded name → empty.

# ── THE HEADLINE JOIN: availability x take rate. Fully paged; one Status query per class. ────
# The join lives in the app's own suite, so there is ONE implementation of it rather than a probe
# script that can drift from the code that ships.
cd app && node test-gap.mjs               # Fresno — the demo county, ~100 s
 # availability 13,514 | adoption 8,451 | joined 8,451 | availability-only 5,063
 # funded gap 786 blocks / 3,818 hh (675 = 85.9% unmeasured) · revenue gap 2,357 / 64,137 · 16.8x
 # (band 0 excluded by default — counting it gives 2,473 / 66,645 / 17.5x, also asserted)

cd app && node test-gap.mjs --statewide   # STATEWIDE — 655k rows, ~330 paged requests, ~25 min
 # availability 390,743 | adoption 264,137 | joined 264,137 | availability-only 126,606
 # FUNDED gap 4,405 blocks / 142,732 hh · REVENUE gap 76,146 blocks / 2,904,667 hh · 20.4x
 # ⚠ 34,046 gap blocks exist but only 4,405 appear here: 29,641 (87.1%) have NO adoption record.
 #   An INNER join would report the funded gap 7.7x too small, and succeed.
 # The run also pins gap.mjs's STATEWIDE constants against the live join, so the recipe's headline
 # figures cannot go stale without a red test.

# ── SUPPORTING LAYERS ────────────────────────────────────────────────────────────────────────
q "$B/CPUC_EOY_2024_CASF_Infrastructure_Eligibility/MapServer/0" "CASF_ELIG=1"   # 29325
q "$B/CPUC_EOY_2024_CASF_Infrastructure_Eligibility/MapServer/0" "bsl_flag=1"    # ERROR 400 (G2)

# ── G17 — the point layer that never returns points. Probe it, do not assume. ────────────────
C=$B/CPUC_EOY_2024_CASF_Infrastructure_Eligibility/MapServer/0
curl -s -A "$UA" "$C?f=json" | python -c "import json,sys;print('declares:',json.load(sys.stdin)['geometryType'])"
                                                              # → declares: esriGeometryPoint
for OF in "CASF_ID" "*" "OBJECTID"; do
  curl -s -A "$UA" -G "$C/query" --data-urlencode "where=CASF_ELIG=1"     --data-urlencode "outFields=$OF" --data-urlencode "returnGeometry=true"     --data-urlencode "outSR=4326" --data-urlencode "resultRecordCount=1" --data-urlencode "f=json"   | python -c "import json,sys;d=json.load(sys.stdin);f=(d.get('features') or [{}])[0];print('  outFields=$OF -> geometry:',f.get('geometry'))"
done            # → all three: geometry: None.  Countable, not mappable.
q "$B/CPUC_EOY_2024_Provider_Identify/MapServer/0" "1=1"                         # 1685 (182 DBA)
q "$B/CPUC_EOY_2024_Community_Anchor_Institutions/MapServer/0" "1=1"             # 100374
q "$B/CPUC_EOY_2024_Median_HH_Income/MapServer/0" "1=1"                          # 25586
q "$B/CPUC_EOY_2024_Political_Boundaries/MapServer/0" "1=1"                      # 58 counties
q "$B/CPUC_EOY_2024_Political_Boundaries/MapServer/6" "1=1"                      # 117 tribal
curl -s -A "$UA" -G "$B/CPUC_RDOF_Phase_I_Winners/MapServer/0/query" \
 --data-urlencode "where=1=1" --data-urlencode 'outStatistics=[{"statisticType":"sum","onStatisticField":"locations","outStatisticFieldName":"loc"},{"statisticType":"sum","onStatisticField":"assigned_support","outStatisticFieldName":"usd"}]' \
 --data-urlencode "f=json"                          # → 26,240 locations · $4,552,860.92

# ── THE 855-LAYER SERVICE — confirm before you ever point /add-data at it (trap G9) ──────────
curl -s -A "$UA" "$B/CPUC_EOY_2024_Drilldown/MapServer?f=json" \
 | python -c "import json,sys;print(len(json.load(sys.stdin)['layers']),'layers')"      # → 855

# ── THE BLOCKED / GATED SOURCES — CONFIRM, never design around a guess ───────────────────────
curl -s -o /dev/null -w "FCC NBM:        %{http_code}\n" -A "$UA" -L https://broadbandmap.fcc.gov/home
                                                            # → 403 (Akamai, even with a browser UA)
curl -s "https://broadbandmap.fcc.gov/api/public/map/downloads/listAvailabilityData/2024-12-31"
                                                            # → {"status":"fail","status_code":401}
curl -s -L "https://api.census.gov/data/2023/acs/acs5?get=NAME,B28002_001E&for=county:019&in=state:06" \
 | head -c 60                                               # → HTML page titled "Missing Key"
curl -s -o /dev/null -w "data.ca.gov:    %{http_code}\n" "https://data.ca.gov/api/3/action/package_search?q=broadband"
                                                            # → 403 (Cloudflare 1009)

# ── THE SECOND OPINION (keyless, and current) ────────────────────────────────────────────────
curl -s "https://ookla-open-data.s3.amazonaws.com/?list-type=2&prefix=parquet/performance/type%3Dfixed/year%3D2026/" \
 | grep -o "<Key>[^<]*</Key>"          # → .../year=2026/quarter=2/2026-04-01_..._fixed_tiles.parquet

# ── CALIFORNIA'S PUBLIC ESTATE: prove the absence that justifies this recipe ─────────────────
python build_gap_catalog.py
 # → library 2351 · role-tagged 292 · ASSERTED EMPTY: availability, adoption,
 #   provider-register, measured-speed — holds
```

**Real field names that drive symbology / joins / KPIs.** Adoption: `GEOID20`, `NAMELSAD20`,
`SQMI_GIS`, `POP_2020`, **`HH_2020`**, `HU_2020`, **`ACAT_10020`** (2024) / **`ACAT_100_20`** (2023) /
**`ACAT_253_New`** / `ACAT_ANY`, `CASF_EL_LOC`. Availability: `GEOID20` + the **unlisted** `Status`.
Downstream max: **`BlockCode`** (not `GEOID20`), `MAX_MaxAdDn`. Providers: `DBA`, `TechCode`,
`Service_Type`, `MaxAdDn`, `MaxAdUp`, `MinDnTier`, `MaxDnTier`, `Busconsm`, `Contact`. CASF BSL:
`CASF_ID` + the **unlisted** `CASF_ELIG`. RDOF: `applicant`, `bidder`, `frn`, `tier`, `latency`,
`county`, `census_id`, `locations`, `assigned_support`. Grants: `PROVNAME`, `DBA`, `FRN`, `PROJNAME`,
`PROJTYPE`, `TECHNOLOGY`, `DOWNSTREAM`, `UPSTREAM`, `HOUSEHOLDS`. Income: `MHI_Int`, `PCAP_INC`,
`FedPovLvl`. Anchors: `ANCHORNAME`, `ADDRESS`, `CAICAT`. Counties: **`NAME20`**. Tribal: `NAME`,
`Type`. Cal OES cellular: `LICENSEE`, `CountyName`.

---

## Guided wizard — **the prompts that assign the app's defaults**

Claude runs this like an ArcGIS Instant-App wizard: ask each group, **apply the default so "accept
all" builds a complete app**, confirm a one-line summary, then run §5. Launch with
`/recipe telecom_underserved-gap-and-take-rate`. Every answer *sets an application default* baked into
`layers.json` / the `AppLayout`.

| # | Wizard question | Options → **default** | The default it assigns |
|---|---|---|---|
| 1 · App | Title & as-of date? | free text → **"Underserved Gap & Take Rate"**, today | header title + `strata:notes.asOf` |
| 2 · Region | Which area? | **Fresno County (CA demo)** · another CA county · a set of counties · statewide · my own extent | initial extent, the `GEOID20 LIKE` prefix, how much is paged live |
| 3 · **Speed tier** | **Which threshold defines "adopted"?** | **100/20 Mbps (federal standard)** · 25/3 Mbps · any available speed | the `ACAT_*` field behind the matrix's columns — *and the app tells the user 25/3 and "any" are the same file, 94.6 % saturated* |
| 4 · Availability classes | Which count as the gap? `[multi]` | Unserved · Priority Unserved <10/1 · both → **both** | the matrix's row grouping and `hh_funded_gap` |
| 5 · The absent row | Show blocks with **no** take-rate record? | **yes, as its own row** · fold into band 0 · hide | whether the fourth matrix row renders — **default yes; hiding it prints a warning** |
| 6 · Band 0 | Treat "0 % or No Data" as a gap? | **no — render ambiguous, exclude from headlines** · yes, count it · hide it | the amber cell rule and every KPI's WHERE |
| 7 · Vintage | Which year, and offer the comparison? | **EOY 2024, with EOY 2023 available** · 2024 only · 2023 only | the `vintage-switch` + the field-rename warning (connection #9) |
| 8 · Frame | Report the gap by? `[multi]` | county · place · tribal land · Assembly · Senate · Congressional · school district → **county + Congressional** | which `Political_Boundaries` layers load and the roll-up in the export |
| 9 · Subsidy | Subtract already-committed money? `[multi]` | CASF approved · RDOF · middle mile → **all three** | the `subsidy` layer and the "already paid for" accordion |
| 10 · Second opinion | Add measured speed (Ookla)? | **no — skip the 370 MB build step** · yes, current quarter · yes, 4-quarter mean | whether `measured` builds and the second-opinion panel renders |
| 11 · Your book | Do you have subscribers by location? | **no → position-only mode** · CSV upload · FeatureServer | whether real penetration + fair-share gap render live or greyed with "needs your subscriber file" |
| 12 · Report/Theme | Board report? · theme/language? | **board PDF + per-county atlas** ; **light / EN** · dark · EN+AR (RTL) | wires `/export report` + atlas; `ThemeSpec` mode + `lang-switch` |

**Then:** Claude echoes *"Fresno County · 100/20 · both gap classes · absent row shown · band 0
ambiguous · EOY 2024 with 2023 · county + Congressional · CASF+RDOF+middle-mile · no Ookla ·
position-only · board PDF · light/EN"* and, on confirmation, runs §5 — so the app opens **fully
configured**.

---

## 5. Prompt-script (run in order)

```
A. /new-app — an "Underserved Gap & Take Rate" open-design app ("gap-matrix", see
   DESIGN-PROPOSAL.md §3-§6). LIGHT ThemeSpec, two-hue semantic system:
   primary #1d4ed8 (CAN they buy), secondary #7c3aed (DO they buy), warning #f59e0b (nobody
   measured), success #059669, danger #dc2626, fonts.scale compact, mono+tabular-nums override
   on kpi and table, motion 180ms. App-shell: header (title + search + county/tier/vintage
   pickers + lang-switch + theme-switch), footer (attribution + share), splash stating the
   thesis in two sentences. Install deps + give me the run command.

B. Build `blocks` — the OUTER join that is the whole design. /convert each CPUC service to
   EPSG:4326 GeoParquet (they are wkid 102100 — request outSR=4326), then LEFT-join
   availability to adoption on GEOID20, keeping every availability block:
     • CPUC_EOY_2024_Fixed_Consumer_Served_Status/0  -> status
       ⚠ `Status` is NOT in fields[] and outFields=Status returns 400 — run one PAGED query per
         class ('Served' / 'Unserved' / 'Priority Unserved < 10 Mbps Down / 1 Mbps Up')
     • CPUC_EOY_2024_Broadband_Adoption_100_20/0     -> band_10020 (ACAT_10020), HH_2020
     • CPUC_EOY_2024_Broadband_Adoption_25_3/0       -> band_253   (ACAT_253_New)
     • CPUC_EOY_2023_Broadband_Adoption_100_20/0     -> band_prev  (ACAT_100_20 — RENAMED field)
   Add `has_adoption` (false for the 5,063 Fresno blocks with no record) and `cell_key`
   = "<row>|<band>" where row ∈ {served, unserved, priority, norecord}. Page EVERYTHING with
   resultOffset and assert exceededTransferLimit is false — maxRecordCount is 2000 and a short
   result is silent truncation that UNDERSTATES the gap. /publish (folder=california,
   service=gap).

C. Build `matrix` — the 24-cell roll-up of `blocks`: row_key, band, blocks_n, hh, hh_pct,
   row_label, band_label, is_active. ASSERT the marginals against §4: statewide 34,046 gap
   blocks and 3,040,820 households in bands 0-3 @100/20. /publish.

D. Build the context layers. /add-data:
     • CPUC_EOY_2024_Provider_Identify/0    (1,685; 182 DBA) -> `providers`
     • CPUC_EOY_2024_Broadband_Grants 0-4 + CPUC_RDOF_Phase_I_Winners/0 +
       CPUC_CDT_Middle_Mile_Network/0 -> union as `subsidy` with a `programme` column
     • CPUC_EOY_2024_CASF_Infrastructure_Eligibility/0 (CASF_ELIG=1 -> 29,325 points)
     • CPUC_EOY_2024_Community_Anchor_Institutions/0, CPUC_EOY_2024_Median_HH_Income/0
     • CPUC_EOY_2024_Political_Boundaries 0 (counties, NAME20) + 6 (tribal) + 7/8/9 (districts)
   /analyze spatialJoin providers x blocks -> provider_n; subsidy x blocks -> rdof/casf flags;
   buffer the middle-mile line -> distance bands. NEVER point /add-data at
   CPUC_EOY_2024_Drilldown — it has 855 layers.

E. /symbology + /popup — genuine ESRI JSON on verified fields only.
   blocks: uniqueValue on `cell_key` (an arcade valueExpression may derive it from status+band)
     — availability hue #1d4ed8, adoption hue #7c3aed, blended per cell; band 0 and
     has_adoption=false in #f59e0b; fill alpha 40/255; NEVER esriSMSPath for the point layers.
   /popup blocks: GEOID20, status, the band LABEL (never the bare integer), HH_2020, providers,
     CASF eligibility, RDOF — and a literal line reading "take rate is published in 20-point
     bands; band 0 means 0% OR no data".

F. /panel statistics as the reading rail: households-in-adopted-cell (headline), funded gap
   (blue), revenue gap (violet), the ratio callout, and a radial gauge for "share of the funded
   gap with no take-rate record" (thresholds 25/60, amber — Fresno reads 85.9%). All as
   kpi/gauge `stat:{field,op}` bound to sourceId `reading` — no connections. Penetration and
   fair-share gap render GREYED with "needs your subscriber file" until a book is bound.

G. The matrix + the rail.
   ① SHIP THE FALLBACK: /panel table over `matrix` (24 rows: row_label, band_label, blocks_n,
      hh, hh_pct; emphasiseField is_active) whose rowSelect carries {field:"cell_key"}. Register
      the app-local `gap-matrix` widget later (DESIGN-PROPOSAL.md §9) — the loop is identical.
   ② carto category widget on blocks.county_name = the county picker.
   ③ accordion: provider list · second-opinion chart (only if Ookla was built) · subsidy list.
   ④ /panel table `blocks` (server-paged, CSV/GeoJSON, rows tinted where has_adoption=false).
   ⑤ interactive legend with live counts, isolate-on-click, and the hint that isolating does not
      change the reading.

H. WIF + controls + export: author AppLayout.connections — the 14 in DESIGN-PROPOSAL.md §5, with
   the SIGNATURE LOOP first (matrix categorySelect -> filter on map + block table + provider
   list + setUrlParam). Map controls navigation/scale/legend/basemapSwitcher positioned
   top-right so the native cluster clears the reading rail; status-bar EPSG:4326. Wire
   /export report (board PDF: legend + scalebar + north-arrow + the PRINTED DERIVATION + the
   matrix as a table), a per-county atlas, /export image, /export layer csv, and a share
   deep-link. Verify responsive.small collapses the splitter and that THE MATRIX SURVIVES
   INTACT (horizontal scroll, never re-flowed to a list).
```

---

## 6. Verify (benchmark to Esri Telecom / VETRO / CostQuest / Ookla)

**Built and driven 2026-08-09** — `app/` (self-contained, MapLibre from CDN; `node server.mjs` →
:8040). **306 assertions green**: `test-gap.mjs` (**147** — the discovered server, both layers against
their own counts, the outer join, the reading, **the traps**, presentation contracts, geometry,
context) and `test-render.mjs` (**159** — the shipped inline module booted against a minimal DOM +
MapLibre stub over a synthetic 10-block county, driving all 41 wired behaviours). The live suite is
**live**: no fixtures, no mocks. The render suite is **fully offline** (`gap.mjs` routes every request
through an injectable `net.fetch`), so it is deterministic and runs in about a second.

**The trap assertions are inverted on purpose** — they pass while the trap is still there. A failure
in `test-gap.mjs` §6 means CPUC fixed something and this §3 needs revisiting; the suite says so on
the way out.

**The interaction surface: 41 wired behaviours** (the design's floor is 3) — full table in
`app/README.md`. The three that matter most:

- **Click any cell → the whole app adopts that population.** Map repaints to exactly those blocks,
  the reading recomputes, the table and provider list re-scope, the URL updates. `←`/`→` step through
  the populated cells **in household order** — one keypress per defensible reading.
- **The no-record row is clickable, and clicking it is the point.** Adopting it selects Fresno's
  5,063 unmeasured blocks and the rail switches its household line to *"these blocks have no
  household count — it is published only on the adoption layer."*
- **`band 0 excluded / counted`.** The toggle that decides whether *"0 % or No Data"* counts as a gap.
  Off by default. Turning it on takes Fresno's revenue gap 64,137 → **66,645** hh and the ratio
  **16.8× → 17.5×**, and prints a warning — so the cost of the ambiguity is visible rather than
  buried in a methodology note.

**Six bugs caught**, both of which read as working: `adopt()` repainted
the map and table but left the reading stale, so the headline household count and both gap KPIs kept
the *previous* cell's numbers; and the vintage field-rename warning was a transient status message
that load progress overwrote before anyone could read it (now a persistent `#notice`); and **the map
was empty on open**, because with nothing adopted `paint()` summed all 13,514 blocks, hit the draw cap
and cleared the source — fixed by caching the county's geometry once (~2.5 MB / ~6 s), which also made
every later cell adoption a **4 ms** client-side filter. The live suite also caught a documentation
error — the server publishes **36** services, not the 37 first recorded.

| Check | Pass |
|---|---|
| An eighth CA ArcGIS server discovered, enumerated (36 services) and CORS-verified | ✅ verified 2026-08-09 |
| California's public estate proven empty of broadband data (2,569 endpoints swept; 4 telecom-adjacent, 0 availability, 0 adoption) | ✅ asserted in `build_gap_catalog.py`, build fails if it changes |
| Both gap definitions counted statewide: 34,046 blocks vs 80,429 blocks / 3,040,820 hh | ✅ live |
| The two gaps joined block-by-block and shown to diverge — statewide **20.4×**, Fresno **16.8×** (17.5× counting band 0) | ✅ driven — `test-gap.mjs` and `--statewide` (655k rows paged) |
| **87.1 % of the funded gap has no take-rate record**, and the outer join keeps it | ✅ 29,641 of 34,046 statewide; 675 of 786 in Fresno — systemic, not local |
| `Status` filterable-but-not-selectable (G1) and `CASF_ELIG`/`bsl_flag` asymmetry (G2) reproduced | ✅ both return the documented 400 |
| 25/3 and "Any" proven to be one dataset (G4); the vintage field rename proven (G5) | ✅ band-for-band identical / `ACAT_100_20` ≠ `ACAT_10020` |
| Every blocked source CONFIRMED, not assumed: FCC 403 + API 401, ACS "Missing Key", data.ca.gov 403 | ✅ all four |
| A keyless, current second opinion exists (Ookla 2026 Q2) | ✅ listed |
| Every `layerId` + field verified against the live service (§4); no field written from memory | ✅ |
| Statewide join, all 58 counties, 655k rows fully paged | ✅ completed — 20.4×, and it *raised* the unmeasured share from Fresno's 86 % to 87.1 % |
| Application built and driven | ✅ `app/` — 306 assertions green (147 live + 159 offline) |
| Scope controls in the header strip; the matrix stretches to fill the band — no dead space beside it | ✅ driven |
| All three regions resizable (rail · band · table), persisted, double-click to reset, clamped | ✅ 10 assertions |
| No gaps or doubled borders — each splitter is the only divider between its two regions | ✅ asserted |
| The reading rail runs the full page height **from directly under the header**; band + map + table stack beside it | ✅ driven |
| The statewide reference constants are pinned against the live 655k-row join | ✅ 6 equalities under `--statewide` |
| Light-vs-dark decided by MEASUREMENT, not taste; the amber AA failure it exposed is fixed | ✅ contrast + ΔE pinned in the suite |
| The county is on the map **on open**, and adopting a cell needs no refetch | ✅ 13,514 blocks drawn; cell adoption 4 ms |
| Map chrome matches the catchment app's vocabulary (cluster · one drawer · live tile thumbnails) | ✅ 14 assertions |
| `Esc` dismisses the splash in the capture phase, without clearing the selection under it | ✅ driven |
| The rail's KPIs are navigation — funded / revenue / unmeasured each map their population | ✅ driven — 786 / 2,357 / 675 blocks |
| The matrix renders as a clickable surface; one live cell per populated cell key | ✅ driven |
| The signature loop end-to-end: cell → map + rail + table + providers + URL | ✅ driven — `served\|3` → 1 block / 200 hh, URL followed |
| The **no-record row** is adoptable and reports "not counted", never 0 | ✅ driven — 5,063 blocks in Fresno |
| Band-0 toggle changes the reading and warns: 64,137 → 66,645 hh, 16.8× → 17.5× | ✅ driven |
| Legend isolate changes the map and **not** the reading (the hint's promise) | ✅ asserted byte-for-byte |
| Vintage switch warns about the renamed field **persistently** | ✅ driven — the toast bug is fixed and pinned |
| Popup names both axes in words, never a band integer; escapes hostile values | ✅ driven incl. an XSS payload |
| Every request goes to one host; the 855-layer Drilldown is never touched | ✅ asserted |
| Every geometry request carries `outSR=4326` + `maxAllowableOffset` (draw, not measure) | ✅ asserted |
| The layer panel REPRESENTS the map: the primary blocks layer is listed, counted and toggleable | ✅ driven — this is why it read as "all hidden" |
| Context layers distinguish **off** / **none in this view** / **capped** / **not mappable** | ✅ driven, all four states |
| G17: the CASF point layer never returns geometry, and the app says so instead of painting nothing | ✅ 5 assertions across every parameter combination |
| Basemaps keyless; two hues never cross semantics in either theme | ✅ 17 assertions |
| Path-traversal guard holds against encoded payloads | ✅ `../`, `%2e%2e%2f`, `..%5c` all 404 |
| Responsive collapse; matrix survives intact (scrolls, never re-flows) | ✅ CSS asserted — **not** visually confirmed (no headless browser in this checkout) |
| Composed PDF / atlas export | ⛏ **not built** — belongs to the `<StrataApp>` path |
| Ookla second opinion | ⛏ **not built** — build-time input (~370 MB/quarter); the panel explains why it is empty |
| AR/RTL binding | ⛏ **not built** — and declared decorative for this audience (§2.2) |

**On-par-or-better:** matches the availability vocabulary of the FCC/CPUC estate and the gap framing
Esri's BEAD dashboard uses, and **exceeds all of them on the one axis none of them can compete on —
showing the funded gap and the revenue gap on the same axes, including the part of the funded gap that
nobody has measured** — plus the AI-authored build, the sovereign/on-prem posture, cross-widget
interactivity and a one-click board report, MIT, on Strata or ArcGIS. **Honestly less than:** VETRO /
IQGeo / 3-GIS on plant records and splice-level design (this app has no OSP model at all); CostQuest
on location-level Fabric precision; Ookla on measurement depth; Esri on enterprise integration and
Business-Analyst enrichment. We ship two published classifications, cross-tabulated, with the
derivation printed — and we say so on screen.

---

## 7. Harvest (gaps → strata-core)

Log as strata-core issues:

- **`gap-matrix` as a core widget** (`DESIGN-PROPOSAL.md` §9) — a clickable cross-tab where cell area
  encodes a magnitude. Promote after a second use. The generic form — *"cross-tabulate two
  classifications of the same features; each cell is a selectable population"* — would serve any
  recipe where two authorities disagree about a categorisation.
- **An `es` (Spanish) dictionary for `@strata/i18n`.** The package ships `en`/`ar` only. For a
  California digital-equity tool, Spanish is the language that matters, and shipping AR while calling
  the app an equity tool is theatre. One file, high value, and it unblocks the whole US public-sector
  catalog.
- **A `banded-measure` renderer helper.** Three recipes now hand-roll "this integer is a bucket, here
  is its label, and one of the buckets means *unknown*". `drawingInfo` can express it, but the
  ambiguous-bucket rule (render it differently, exclude it from headlines) is a policy the app has to
  reimplement each time.
- **Outer-join as a first-class option** in `/analyze`. `spatialJoin`/attribute joins default to inner,
  and **inner-joining two authorities' block lists silently deleted 86 % of this app's subject**. The
  correct thing should be the easy thing — cf. the catchment recipe's identical plea for areal
  apportionment.
- **A "hidden field" probe in `/add-data`.** Two fields on this server drive renderers and filter in a
  WHERE while being absent from `fields[]`, and one of those two 400s on `outFields`. A one-line
  capability probe at add-data time would have saved an afternoon, and the same class of trap has now
  appeared on four services across three recipes.
- **A layer-count guard.** `CPUC_EOY_2024_Drilldown` has **855 layers**; `/add-data` and `LayerPanel`
  should refuse (or paginate) above a threshold rather than trying.
- **Per-widget "assumption" affordance** — a first-class way to attach a printed derivation + source
  to any KPI. This is now the **fourth** recipe to hand-roll it (`education_campus-operations`,
  `real-estate_site-selection`, `marketing_catchment-and-market-share-analyzer`, this one). It is a
  pattern, not a one-off.
- **`buttonClick` emitters** (Phase-2 pending) — the tier/vintage switches want real segmented
  controls; `menu` works but carries list semantics the design does not want.
- **A resizable-region primitive for `AppLayout`.** `splitter` exists as a container kind, but the
  built app needed three splitters driving CSS variables, persisted, clamped and double-click-to-reset
  — roughly 60 lines that every console-shaped recipe will rewrite. It belongs in the layout engine
  with `sizes`/`minSizes` already in the schema.
- **A layer-state vocabulary for `LayerPanel`.** A layer can be *off*, *on and empty here*, *capped at
  maxRecordCount*, or *undrawable because the service publishes no geometry* (trap G17). Today all
  four render as an unticked or ticked box, which is how "everything is hidden" gets reported. The
  panel should be able to say which.

Editing is not applicable — this app has **no write path by design**.

---

## 8. Sources

- **Programme & policy:**
  [CRS R48666 — The BEAD Program: Issues for the 119th Congress](https://www.congress.gov/crs-product/R48666) ·
  [FBA — BEAD Funding Moves Forward (Apr 2026)](https://fiberbroadband.org/2026/04/02/bead-funding-moves-forward/) ·
  [ALEC — "Benefit of the Bargain" restructuring](https://alec.org/article/broadband-update-a-benefit-of-the-bargain-for-the-bead-program/) ·
  [CPUC BEAD Program](https://www.cpuc.ca.gov/beadprogram) ·
  [CRS IF12637 — The End of the Affordable Connectivity Program](https://www.congress.gov/crs-product/IF12637) ·
  [Pew — States Reckon With Lapse of the ACP](https://www.pew.org/en/research-and-analysis/articles/2024/09/20/states-reckon-with-lapse-of-the-broadband-affordable-connectivity-program)
- **The mapping and its critics:**
  [FCC BDC — How to Submit an Availability Challenge](https://help.bdc.fcc.gov/hc/en-us/articles/10476040597787-How-to-Submit-an-Availability-Challenge) ·
  [What's on the National Broadband Map](https://help.bdc.fcc.gov/hc/en-us/articles/13532984820379-What-s-on-the-National-Broadband-Map) ·
  [NRECA — FCC Maps challenge process overview (PDF)](https://www.cooperative.com/topics/telecommunications-broadband/documents/fcc-map-update-overview.pdf) ·
  [ILSR — "Deadline to Challenge FCC Maps Highlights Failed Process"](https://ilsr.org/article/community-broadband-networks/fcc-mapping-challenge-deadline-highlights-failed-process/)
- **Take rate & the economics:**
  [Adtran — "Mind the gap: why homes passed is no longer enough"](https://www.blog.adtran.com/en/mind-the-gap-why-homes-passed-is-no-longer-enough) ·
  [FBA — 2025 North American FTTH Deployment & Market Update](https://fiberbroadband.org/resources/2025-north-american-ftth-deployment-and-market-update/) ·
  [Percepture — What is fiber take rate?](https://percepture.com/telecom-insights/what-is-fiber-take-rate/) ·
  [PwC — US consumer fiber shakeout or step change? 2026 outlook](https://www.pwc.com/us/en/industries/tmt/library/consumer-fiber-shakeout-or-step-change.html) ·
  [EY — How US FTTH providers can navigate an evolving market](https://www.ey.com/en_us/insights/telecommunications/how-us-ftth-providers-can-navigate-an-evolving-market) ·
  [Brattle/FBA — Economic Benefits of Fiber Deployment (PDF)](https://fiberbroadband.org/wp-content/uploads/2024/11/2024.11.20-Benefits-of-Fiber-Deployment-Brattle-FINAL.pdf)
- **The adoption gap:**
  [PPI — Closing the Broadband Adoption Gap](https://www.progressivepolicy.org/closing-the-broadband-adoption-gap/) ·
  [Benton — Lessons From Broadband Adoption Trends](https://www.benton.org/blog/lessons-broadband-adoption-trends) ·
  [Census Reporter — Table B28002](https://censusreporter.org/tables/B28002/) ·
  [Brookings — Broadband Adoption Rates and Gaps (PDF)](https://www.brookings.edu/wp-content/uploads/2016/07/broadband-tomer-kane-12315.pdf)
- **Esri (nominative use):**
  [ArcGIS Telecom Management](https://www.esri.com/en-us/industries/telecommunications/overview/telecom-management) ·
  [Bridging the Digital Divide](https://www.esri.com/en-us/industries/telecommunications/digital-divide) ·
  [ArcGIS for telecom network planning](https://www.esri.com/en-us/industries/telecommunications/digital-divide/arcgis-software-for-telecom-network-planning) ·
  [ArcGIS for telecom asset management](https://www.esri.com/en-us/industries/telecommunications/digital-divide/arcgis-software-for-telecom-asset-management) ·
  [Telecom network planning & buildout with ArcGIS Solutions (StoryMap)](https://storymaps.arcgis.com/stories/eca696d2abf04a038f5c71ed7e390f76)
- **Non-Esri platforms:**
  [VETRO FiberMap (Capterra)](https://www.capterra.com/p/193245/VETRO-FiberMap/) ·
  [3-GIS vs VETRO FiberMap](https://slashdot.org/software/comparison/3-GIS-vs-VETRO-FiberMap/) ·
  [ISE — Comprehensive broadband availability & data mapping](https://www.isemag.com/cande-netdev-ops-gis-open-source-networks/article/55129176/comprehensive-broadband-availability-data-mapping)
- **California data:**
  [California Interactive Broadband Map](https://www.broadbandmap.ca.gov/) *(the Leaflet app whose `config.js` exposed the CPUC ArcGIS server)* ·
  [CPUC Annual Collected Broadband Data](https://www.cpuc.ca.gov/industries-and-topics/internet-and-phone/broadband-mapping-program/cpuc-annual-collected-broadband-data) ·
  [CPUC Broadband Mapping Program](https://www.cpuc.ca.gov/industries-and-topics/internet-and-phone/broadband-mapping-program) ·
  [CPUC CASF project development resources](https://www.cpuc.ca.gov/industries-and-topics/internet-and-phone/california-advanced-services-fund/project-development-resources---data-and-maps) ·
  [Last Mile FFA public map](https://www.cpuc.ca.gov/industries-and-topics/internet-and-phone/broadband-implementation-for-california/last-mile-federal-funding-account/ffa-public-map)
- **Data endpoints (all curl-verified 2026-08-09 — see §4):** CPUC ArcGIS
  `cpuc2016.westus.cloudapp.azure.com/arcgis/rest/services/CPUC` (36 services) ·
  Cal OES hosted org `services.arcgis.com/BLN4oKB0N1YSgvY8` ·
  [Ookla Open Data](https://ookla-open-data.s3.amazonaws.com/) ·
  [Census TIGERweb](https://tigerweb.geo.census.gov/arcgis/rest/services) ·
  [Census API key signup](https://api.census.gov/data/key_signup.html) (now required) ·
  [FCC Open Data](https://opendata.fcc.gov/) (Form 477, June 2021 — stale) ·
  [FCC National Broadband Map](https://broadbandmap.fcc.gov/) (403 server-side)
- **Internal:** `DESIGN-PROPOSAL.md` (this recipe's full design record) · `gap-catalog-ca.json` +
  `build_gap_catalog.py` · `app/test-gap.mjs` (the join, and the trap suite) · `../APP-TEMPLATE-LIBRARY.md` · `../DESIGN-CONTEXT.md` ·
  `../../data_sources/data_sources_ca.md` ·
  `../marketing_catchment-and-market-share-analyzer/RECIPE.md` (format exemplar; its ACS and
  data.ca.gov findings are reconfirmed here) · `strata/recipes/COMPONENT-MANIFEST.md` (§10 freestyle
  charter) · `strata/docs/guide/app-design.md` · `strata/docs/reference/human-language.md`

---

## Modernization (parity release)

> Applies the Experience-Builder-parity features (Phases 1–7). See `strata/recipes/MODERNIZATION.md` +
> `COMPONENT-MANIFEST.md` §8. Cross-cutting: a structured **`theme`**, app-shell
> (`header`/`footer`/`splash`), and `dataSource.{ sourceId }` linking (widgets sharing a source link
> with **no `connections`**).

- **DataSource linking is the backbone, not a nicety.** Nine widgets — three KPIs, the gauge, the
  ratio callout, the derivation block, the adopted-cell label, the subsidy list and the legend — bind
  to `sourceId` `reading`/`subsidy` and relink on every cell adoption with **zero** wiring. Without it
  this app would need roughly 20 more connections.
- **`recordsChange` + `fromWidget` as the app's spine.** The matrix publishes the adopted population
  as the `reading` output source; six widgets consume it. The signature loop is therefore *one*
  trigger feeding both a `filter` fan-out and an output source.
- **Structured `ThemeSpec` doing semantic work.** Two hues encode the two questions
  (`primary` = can they buy, `secondary` = do they buy) and `warning` is redefined as *absence*
  rather than danger — a theme carrying an argument, not a palette. `overrides.kpi`/`overrides.table`
  put counts in mono with `tabular-nums` so 24 cell values align to one grid.
- **App-shell + `splash`.** The splash states the thesis in two sentences before the first render;
  this app's argument has to land before its numbers do.
- **`accordion`** collapses three secondary panels (providers · measured · money committed) into one
  rail slot, so the reading stays above the fold.
- **`splitter`** once (map ↔ reading rail) with `responsive.small` collapse — and the explicit rule
  that **the matrix survives the collapse intact**.
- **`arcade`** one `valueExpression` deriving `cell_key` from `status` + band on the live renderer, so
  the tier and vintage switches repaint without a rebuild.
- **`FileDataSource`** is the client-book path: a CSV of subscribers by location turns real
  penetration and the fair-share gap from greyed to live, with no schema change.
- **Freestyle charter §10.2** invoked exactly once, for `gap-matrix`, with a named `table` fallback
  that preserves the signature loop.
