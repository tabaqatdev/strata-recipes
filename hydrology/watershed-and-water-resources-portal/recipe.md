# Recipe — Water Resources (Natural Resources & Environment)

A reproducible path to a **watershed availability and quality** application on strata-core that answers
the question every water-supply number hides: *"is this figure measured, or is it modelled?"* It puts
California's three water books — **what has been promised** (60,569 points of diversion), **what is
measured** (2,597 stream gages, 1,196 of them switched off) and **what has been assessed for quality**
(4,170 river segments against 1,603,153 never assessed) — on one nested watershed navigation, and
prints, at every level of the hierarchy, which of the three the state can actually answer at that grain.

> **Scope (honest).** An **analytics and serving layer over published regulatory data** — not a system
> of record, not a hydrologic model, not a water-budget engine, not a substitute for a Water Supply
> Assessment. **The app reports how MANY claims exist on a watershed and never how much water they
> claim** — and the reason is stronger than this recipe first recorded. The eWRIMS Points of Diversion
> layer **is** reachable (its host's WAF is *intermittent*, not standing), it carries **63,990**
> diversions keyed to `HUC_12`, and it has **39 fields, none of which is a quantity of water**. The
> volume is therefore missing from the published GIS record itself, not merely hidden behind a
> firewall; the state's open-data route (`data.cnra.ca.gov`) additionally answers **403**. So the
> over-allocation ratio the literature reports (~5× mean annual runoff, Grantham & Viers 2014) is
> **cited and not reproduced**. **The claim register is preferred-but-flaky**: when eWRIMS answers,
> diversions are *published* at HUC12 and nothing in the app is an estimate; when the WAF is in the
> way it falls back to the HUC10 density layer and *apportions*, and the grain notice names which
> register answered. **"Measured" is a definition, not a fact** — active flow gage vs
> real-time vs any-record-including-dead moves the headline from 37.4 % to 12.6 %, and the app makes
> the reader choose and prints the difference. **`listingstatus='Not Listed'` does not mean clean** —
> it means assessed and not listed, against an assessed set that is a fraction of a percent of the
> network. **Groundwater does not nest inside watersheds** (515 B118 subbasins vs 4,469 HUC12s) and the
> app shows the mismatch rather than papering over it. **The SB 19 figures are 2022-vintage**, compared
> against that plan's own inventory. **No write path at all**, so it runs identically on Strata and
> ArcGIS. Deploy **on-prem** where an agency binds its own diversion or gage records.

Run the §5 prompt-script on a fresh strata-core project (or via `/new-app`). ESRI Web Map JSON is the
map contract; ingest is via `/convert` → `/publish` to Strata Serve.

---

## 0. Provenance

| | |
|---|---|
| **Source** | `https://tabaqat.net/solutions` — **Natural Resources & Environment** section |
| **Name on site** | Water Resources |
| **Tagline on site** | "Watersheds, quality and availability for resource and utility planning" |
| **Scaffolded** | 2026-07-22 |
| **Researched** | **2026-08-10** — this pass replaces the scaffold in full |
| **Template** | **`open-design`** ("basin-descent") — three candidates, the anti-collision check and the two rejections are in **`DESIGN-PROPOSAL.md`** §2. **The scaffold's provisional `scoreboard` assignment is rejected** (rule 1 of `../DESIGN-REQUEST-PROMPT.md`, and `scoreboard` is already held *within this sector* by `emergency-management_live-dashboard`) |
| **Built** | `app/` — self-contained, MapLibre from CDN; `node server.mjs` → :8041. **335 assertions green**: `test-basin.mjs` (128, live), `test-render.mjs` (118, offline), `drive.mjs` (89, real headless Chrome). See `app/README.md` for the 51 wired behaviours and the twelve corrections building it produced. |
| **Presentation** | `presentation/index.html` — 10-slide deck (keyboard/click nav, print-to-PDF), carrying the descent as its signature object · `presentation/linkedin-article.md` — ~1,050-word article + teaser post + a claims note listing every figure and its source |
| **Catalog** | `watershed-catalog-ca.json` — **2,532** CA endpoints inventoried, **342** role-tagged across 11 roles, **16** external feeds (13 verified-live). Rebuild with `python build_watershed_catalog.py` |
| **Overlaps** | `utilities_sampling-and-water-quality-map` (compliance sampling — that one is the *operator's* view of sample results; this one is the **watershed availability/quality** framing) · `hydrology_groundwater-level-and-abstraction` (the aquifer side) · `hydrology_rainfall-and-streamflow-monitor` (the live-telemetry side; this app is about the gages that **are not** there) |

---

## 1. Study — how the market frames this

**The question the buyer asks:** *"I have to commit to a water supply figure for this watershed and
defend it for twenty years. How much of that number did somebody actually measure?"*

### The finding this recipe is built on (verified live 2026-08-10, not asserted)

California answers the availability question on a network it largely does not have.

| | |
|---|---|
| HUC12 watersheds in the state's own SB 19 frame | **4,469** |
| …containing an **active gage that measures flow** | **559 — 12.5 %** |
| **…with no active flow gage** | **3,910 — 87.5 %** |
| The state's own `huc12_prop_needgage` field, statewide mean | **0.868** — *a different column, the same answer* |
| Stream length **never gaged** | **239,521 km of 262,259 km — 91.3 %** |
| Gage sites in the inventory that are **Inactive** | **1,196 of 2,597 — 46.1 %** |
| Gage sites reporting **real-time flow** | **724 — 27.9 %** |
| Gage sites operated by **USGS** | **2,078 — 80.0 %** |

**Two independent routes agree.** An attribute join of the 2,597-site gage inventory to the 4,469
watersheds gives 87.5 % ungaged; the Water Board's own per-watershed `huc12_prop_needgage` column
averages 0.868. Neither was derived from the other.

**And the number that makes it a business problem rather than a science one** — joined watershed by
watershed on HUC10 (§4 reproduces it):

| | watersheds | points of diversion |
|---|---|---|
| Licensed diversions statewide | 1,128 | **60,569** |
| In watersheds with **no active flow gage** | **557** | **22,676 — 37.4 %** |
| In watersheds with **no gage record at all, ever** | 284 | 7,646 — **12.6 %** |
| **The retreat** — diversions that once sat in a gaged watershed and no longer do | | **24.8 points** |

> **More than a third of California's licensed diversions are taken from watersheds nobody currently
> measures — and two thirds of that gap is not a network that was never built, but a network that was
> switched off.**

**The state named the problem itself and has not closed it.** **SB 19 (2019)** directed DWR and the
State Water Board to prioritise the gaging gap; the 2022 plan tiered **486 Tier-1** watersheds and
recommended reactivating **161** of ~900 inactive gages. Today **382 of those 486 Tier-1 watersheds —
78.6 % — still have no active flow gage.** The federal picture is the same: of **4,756** sites that
qualify for the USGS Federal Priority Streamgage network, only **3,436 are active**, and only 25–30 %
of those are fully USGS-funded.

**The quality book is smaller still.** The **2026 Integrated Report** (Clean Water Act §303(d)/§305(b))
assesses **4,170** river/stream segments — **2,235 listed impaired (53.6 %)** — and **905** lakes,
**581 listed (64.2 %)**. Against that, the Water Board publishes **`NHD_Not_Assessed`: 1,603,153
reaches**. So "Not Listed" means *assessed and not listed*, and any portal that paints it green is
asserting something the record does not support.

**And there is no single "watershed".** California reports water on **at least six incompatible
partitions**, none of which nest: CalWater planning watersheds (**7,008**), federal HUC12 (**4,469**),
HUC10 (**1,128**), CalWater hydrologic units (**203**), B118 groundwater subbasins (**515**), DWR
hydrologic regions (**12**) and Regional Water Quality Control Board regions (**9**). A planner asking
"how much water is here" is answered by two agencies on two unrelated geometries — surface water by
topography, groundwater by alluvium. **That mismatch is not a data-quality defect to be joined away; it
is the structural reason a Californian watershed balance sheet cannot be assembled from the state's own
publications, and this app renders it rather than hiding it.**

### Reference solutions (benchmark + coexist, never copy)

- **Esri** — [GIS for Water Resources / watershed management](https://www.esri.com/en-us/industries/water-resources/overview)
  and **[Arc Hydro](https://www.esri.com/en-us/industries/water-resources/arc-hydro)**, the de-facto
  standard for building the foundational water dataset (flow direction, accumulation, watershed and
  stream-network delineation), plus
  [GIS for Water](https://www.esri.com/en-us/industries/water) and the
  [water-utility solutions](https://www.esri.com/en-us/industries/water-utilities/overview).
  Sold as an enterprise agreement. Arc Hydro's product **is** the delineation — its incentive is to make
  the watershed look complete.
- **Time-series / monitoring platforms** (the systems an agency already owns) —
  **[AQUARIUS](https://aquaticinformatics.com/)** (Aquatic Informatics, a Danaher company) and
  **KISTERS [WISKI](https://www.kisters.eu/product/wiski/) / [Hydstra](https://www.kisters.com.au/product/hydstra/)**:
  hydrometric network management, rating curves, QA of continuous records, agency data portals. Both
  are excellent, and both answer *what did my gages record* — a question that presupposes gages.
- **Modelling** — **[Aquaveo](https://aquaveo.com/software)** (GMS/SMS/WMS, and SaaS web viewers),
  **[WEAP](https://www.weap21.org/)** (SEI, integrated water-resources planning and allocation),
  **DHI MIKE**, **HEC-HMS/RAS**. These are the tools that **fill** the ungaged 87.5 % with regression
  and simulation — legitimately, and invisibly to the person reading the output.
- **Public reference services** — **[USGS StreamStats](https://streamstats.usgs.gov/)** (delineate a
  basin, get regression-estimated flow statistics — *the* instrument for ungaged sites, and it is
  explicit that it is regression), **USGS NWIS / Water Dashboard**, **EPA How's My Waterway**,
  **[USGS water-resources software catalog](https://water.usgs.gov/water-resources/water-resources-software-catalog/)**,
  and the EU's **WISE / Water Framework Directive** reporting portals — the closest non-US analogue to
  the Integrated Report.
- **California's own** — the SWRCB [SB 19 Stream Gaging Plan](https://www.waterboards.ca.gov/waterrights/water_issues/programs/stream_gaging_plan/)
  and Gage Analysis Tool, [Water Availability Information](https://www.waterboards.ca.gov/waterrights/water_issues/programs/water_availability/),
  the [Fully Appropriated Streams](https://www.waterboards.ca.gov/waterrights/water_issues/programs/fully_appropriated_streams/)
  programme, DWR's SGMA Data Viewer, and the CNRA/California Water Data Consortium open-data effort.

### Our edge

**Nobody's product is the absence.** Aquatic Informatics and KISTERS sell the platform that manages gage
data — their product gets *worse* the more prominently you show the gages that do not exist. Aquaveo,
DHI and WEAP sell the simulation that fills the gap — they cannot foreground that the gap is being
filled by simulation. Esri sells the platform and Arc Hydro sells the delineation. **Every incumbent
has a measurement or a model to defend; we have neither**, which is precisely why an open, MIT,
on-prem tool can put the ungaged 87.5 % in the middle of the screen and still be the honest one in the
room. Second: **the authoritative claim register is unreachable and we say so on screen** (§3). Plus
the standing edges — AI-authored, runs on **Strata *or* ArcGIS**, sovereign/on-prem so an agency's own
diversion book never leaves the perimeter, and cross-widget interactivity on the first build.

### Standards, programmes & organizations to speak fluently

- **SB 19 (2019)** — the Stream Gaging Prioritization Plan; DWR + State Water Board, in consultation
  with CDFW, DOC and the Central Valley Flood Protection Board. 2022 technical report + appendices +
  digital data. Five prioritization categories: **ecosystem · water supply · water quality · flood and
  public safety · reference.**
- **Clean Water Act §303(d) / §305(b)** — the **Integrated Report** listing cycle. California's **2026**
  cycle is final. `integrated_report_category` (1–5) is the assessment category; **Category 5** is the
  §303(d) list proper. A listing obliges a **TMDL**.
- **Porter–Cologne Water Quality Control Act** — the state statute, and the **Basin Plans** that assign
  **beneficial uses** (`MUN`, `AGR`, `COLD`, `WARM`, `REC1/2`, `SPWN`, `MIGR`, `BIOL`, `RARE`, `COMM`,
  `SHELL`, `AQUA`). A use is what the water is legally *for*, and it constrains diversion independently
  of hydrology.
- **Water Code §1200 et seq.** — appropriative water rights, **eWRIMS**, and **Water Right Order 98-08**,
  which declares **Fully Appropriated Stream Systems**: reaches that cannot accept new appropriations.
- **Water Availability Analysis (WAA)** — how the Division of Water Rights decides whether water is
  available to appropriate. It rests on **unimpaired flow**, which in an ungaged watershed is a
  *regression estimate*. The reform literature explicitly flags re-evaluating unimpaired-flow
  methodology **in ungaged watersheds** — the exact seam this app exposes.
- **Water Code §10910 (SB 610) / §10912** — **Water Supply Assessments** for large developments;
  **§10610 et seq.** — **Urban Water Management Plans**, every five years. These are the documents this
  app's export is written into.
- **SGMA (2014)** — Sustainable Groundwater Management Act; **Bulletin 118** basins, GSAs, GSPs, basin
  prioritization, critically overdrafted basins, probation.
- **SAFER** (SB 200) — Safe and Affordable Funding for Equity and Resilience; failing/at-risk public
  water systems. **HR 2 / AB 685** — the Human Right to Water.
- **NHD / NHDPlus / WBD (HUC8-10-12)** — the federal hydrography and watershed-boundary datasets;
  **CalWater 2.2.1** — California's own, older, incompatible partition.
- **ESRI Web Map / `drawingInfo` / `popupInfo` JSON** — this repo's rendering contract.

**Honest scope** — see the blockquote at the top; repeated in the app's `splash` and footer.

---

## 2. UI design spec (front-loaded)

### 2.1 Layout (Template: `open-design` — "basin-descent")

- **Template `open-design`** under the freestyle charter (`strata/recipes/COMPONENT-MANIFEST.md` §10).
  Full derivation, three candidate silhouettes, both rejections and the anti-collision check:
  **`DESIGN-PROPOSAL.md`** §2.
- **Why not a library template.** The sector's neighbours already hold `time-player` (flood early
  warning), `chart-board`, `zone-lookup`, `launchpad`, `media-pager`, `scroll-story`, `compare-swipe`,
  `sidebar-explorer` and four open-designs — and **`scoreboard`, the scaffold's provisional pick, is
  held inside this same sector by `emergency-management_live-dashboard`**. It is also wrong on its
  merits: an extent-driven stat wall implies the statistic is a property of the viewport, when here it
  is a property of a **named, nested administrative unit**.
- **The silhouette.** Navigation is the **watershed hierarchy descended one level at a time** —
  HUC8 → HUC10 → HUC12 — drawn as **three stacked bands of blocks whose width is drainage area**.
  Clicking a block re-draws the band beneath it with that watershed's children and scopes the whole app.
  The ink is **one continuous ratio — the share of drainage area that drains past a working gage** —
  from measured teal to unmeasured amber, hatched where claims exist and measurement does not.
- **The signature accent — records drop out as you descend.** At HUC8 all three books answer. At HUC10
  the diversion register answers and the gaging tier does not. At HUC12 the gaging record answers and
  the claim count can only be **apportioned**. At a single reach only the quality record exists, and for
  1.6 M of 1.6 M reaches it says *not assessed*. **A persistent grain notice names, at every level,
  which records can still answer and which have become estimates.** The app makes the reader watch its
  own knowledge thin out as they zoom in — a finding rendered as an interaction, not asserted in a
  footnote.
- **Signature loop:** **click any block → the whole app adopts that watershed** — map repaints and flies,
  the three books recompute, the grain notice updates, the table re-scopes, the URL updates.
- **Wiring floor:** the design authors **14 `connections`** (the guideline's floor is 3) plus **9
  widgets linked by a shared `sourceId`** with no wiring at all.
- **Anti-collision:** every navigation in the library is **flat** — a vertical ladder (aviation), a
  from×to matrix (agriculture), a voltage rail (utilities), an elimination cascade (mining), a 2-D
  cross-tab (telecom), a ranked slate (real estate), a threshold rack (education), a linear-referenced
  strip (energy), a recourse ladder (local government), a share fraction (marketing). **This one is
  nested, and the nesting is the argument.** Nearest cousins named honestly in `DESIGN-PROPOSAL.md` §2.

**The ASCII skeleton, real figures (Russian River HUC8 1801011), is in `DESIGN-PROPOSAL.md` §3.**

**Map chrome.** MapLibre's native zoom cluster is hidden and replaced with one top-right cluster —
zoom in · zoom out · **fit to the adopted watershed** · **L**ayers · **B**asemap · Le**g**end — drawers
opening beside it, never over it. Same vocabulary as `industry_mining-and-concession-compliance` and
`telecom_underserved-gap-and-take-rate`.

**The legend is a control surface, not a caption.** Each paint class isolates or toggles and carries a
live count; the *claims present, no measurement* hatch is the class most users click first, which is the
point. Isolating a class deliberately does **not** change the adopted watershed or the reading, and a
hint line says so.

**Responsive.** Below 1240 px the scope strip wraps onto its own line; below 1100 px the body collapses
to a column and the splitters hide. **The descent survives intact** — it scrolls horizontally inside its
band and is never re-flowed into a list, because a hierarchy that has become a list is no longer a
hierarchy. The grain notice stays pinned beneath it. Map drops to 55 vh; the three books follow; the
table goes last.

### 2.2 Theme

Structured `ThemeSpec`, **`mode: "light"`**. **Three hues carry the whole argument: `primary #0e7490`
(teal) is MEASURED — we have evidence; `secondary #7c3aed` (violet) is PROMISED — claims and diversions;
`warning` (amber) is UNMEASURED / NEVER ASSESSED — absence, not danger.** No element may use one to mean
another, so a reader learns the distinction from colour before reading a label. `danger #b91c1c` is
reserved for a **real regulatory finding** (a §303(d) listing), never for absence; `success #15803d` is
assessed-and-not-listed.

`fonts.scale "compact"` · **mono override on `kpi` and `table`** with `tabular-nums` so watershed counts
align to one grid · motion **180 ms**, short and mechanical, because every animation here is a
measurement changing.

**Measured, 2026-08-10** — WCAG 2.1 contrast and CIE76 ΔE, not asserted:

| token | on white | on dark | AA |
|---|---|---|---|
| `primary` **measured** `#0e7490` | **5.36** | 9.82 | ✓ / ✓ |
| `secondary` **promised** `#7c3aed` | **5.70** | 6.52 | ✓ / ✓ |
| `warning` **unmeasured** `#b45309` | **5.02** | 10.63 | ✓ / ✓ |
| `danger` **listed** `#b91c1c` | 6.47 | 6.41 | ✓ / ✓ |
| body / muted text | 17.74 / 7.56 | 14.33 / 6.99 | ✓ / ✓ |
| **`#f59e0b` amber as TEXT on white** | **2.15** | 8.26 | **✗ AA** |

ΔE76 between map fills: measured-vs-promised **98.7** · measured-vs-unmeasured **109.2** ·
promised-vs-unmeasured 162.3. The three load-bearing hues are unmistakable from each other in both modes.

⚠ **Amber needs two values, not one — and this reproduces, independently, the split
`telecom_underserved-gap-and-take-rate` was forced into.** As a **fill** over a pale basemap `#f59e0b`
is right; as **text** on a white panel it is **2.15:1**, well under AA's 4.5, and amber marks this app's
largest finding. So `--strata-warning-fill: #f59e0b` carries every map fill and gauge stroke, and
`--strata-warning: #b45309` (**5.02:1**) carries every text use. Dark mode needs no split. **Two recipes
hitting the same wall is a pattern, not a coincidence** — logged in §7.

**Why light, deliberately:** the artefact is a **counter-document** that leaves as a PDF into an Urban
Water Management Plan, a Water Supply Assessment or a board packet; the descent depends on **comparing
many differently-sized blocks**, which low-alpha fills over a dark basemap destroy; and every persona
navigates by **watershed name**, which a dark basemap under a choropleth makes unreadable. **Dark stays
first-class** via `theme-switch` (a Regional Board does run this on a wall), and an explicitly chosen
basemap survives a theme swap.

Basemap **keyless**: the full `OPEN_BASEMAPS` set — CARTO Positron · Voyager · Dark Matter ·
OpenStreetMap · **OpenTopoMap** (terrain, because a watershed *is* a shape in the topography) —
paired to the theme via `basemapForTheme`, and **an explicitly chosen basemap survives a theme
swap**, which is now asserted in the browser rather than merely promised. The picker follows the
sibling apps: each option carries a **live tile of the area on screen** rather than a colour swatch
(a swatch cannot tell Positron from Voyager, which is the choice the list exists to make), plus a
**Follow the theme** row — and the basemap *in force* ticks whether it was chosen or derived, so a
list of five with nothing ticked can never happen. Watershed fills alpha ≈ 40/255. **EPSG:4326 throughout** — SWRCB hosted layers are
`wkid 3857` on the wire, so `outSR=4326` on ingest.

**i18n — honest.** EN + AR/RTL is wired and works, but for this audience it is **decorative and the
recipe says so**: the languages that matter in California water are English and Spanish, and
`@strata/i18n` ships `en`/`ar` only. An `es` dictionary is a one-file change and is logged in §7 — the
second recipe to ask for it.

### 2.3 KPI cards

Four `kpi` plus one `gauge`, all live `stat: {field, op}` bound to the shared `reading` source, so they
recompute on every watershed adoption **with no `connections`**:

**PROMISED** — points of diversion in scope (violet), with *"count only — the volume is not public"*
beneath it · **MEASURED** — active flow gages (teal), with the SB 19 tier and the recommended
reactivation site id · **ASSESSED** — segments listed / assessed (red / green) · **reaches never
assessed** (amber) · and a radial **`gauge` for the share of drainage area draining past a working
gage** (thresholds 25/60, inverted, amber), which for the Middle Russian River reads **11 %**.

Two further metrics — **face-value allocation** and **allocation-to-supply ratio** — render **greyed
with an explicit "the volume record is not reachable — see *What this cannot show*"** until an agency
binds its own eWRIMS extract via `FileDataSource`.

### 2.4 Charts & table

- **The descent** — the signature. A new `basin-descent` widget (§10.3 block in `DESIGN-PROPOSAL.md`
  §9) plotting nested watershed blocks with width ∝ drainage area; **named fallback: a `table`** over
  the same `watersheds` source, grouped by level, whose `rowSelect` carries the identical payload, so
  **the signature loop survives unchanged**. **Ship the fallback on day one.**
- **One `stacked-bar` only** — the three books to scale, which is the single most persuasive image in
  the app (4,170 assessed against 1,603,153 not). **No chart rail** — that would collide with
  `emergency-management_hazard-impact-analyzer`'s `chart-board` in the same sector.
- **Watershed table** — `AttributeTablePanel` over `watersheds`: `huc` · `name` · `claims_n` ·
  `gaged_area_share` · `listed_n` · `assessed_n` · `tier`. Sortable, per-column filter, virtualized,
  row → `zoomTo` + `flash`, CSV/GeoJSON export. **Rows with claims and no active gage are amber-tinted**,
  and **apportioned claim counts render in italic** — the estimate is visible in the table too, not only
  in the grain notice.

### 2.5 Capabilities to use (Phases 0–7)

- **`/analyze`** at build time: `spatialJoin` (gages × HUC12; IR segments × HUC12; B118 × HUC12 for the
  overlap %), `dissolve`/`aggregate` (HUC12 → HUC10 → HUC8 roll-ups of drainage area and claims).
  **`hexbin`/`hotspot` rejected** — density answers "where are they clustered"; the question here is
  *coverage of a named unit*. **`buffer` rejected** — a buffer around a stream is not a watershed.
  **`weighted-overlay` refused on principle** — it would produce a composite "watershed health score",
  the invented number this app exists to refuse.
- **WIF `connections`** — 14 (`DESIGN-PROPOSAL.md` §5), signature loop first.
- **DataSource linking** — 9 widgets on shared `sourceId`s (`reading`/`watersheds`/`benuses`/`b118`);
  without it the connections table would be ~24 rows.
- **`arcade`** — one `valueExpression` deriving the descent's paint class from `gaged_area_share`, so
  the *measured means* toggle repaints without a rebuild.
- **`FileDataSource`** — the client path: an agency's own gage inventory or eWRIMS extract turns the
  greyed volume KPIs live with no schema change.
- **Deliberately NOT used:** `filter`/`query` builders (a free-form builder lets a user construct a
  "water available" definition the app cannot then defend — the three named definitions are the whole
  choice set), `timeslider`/`date-filter` (two IR cycles and one SB 19 vintage is not a series, **and**
  the period-of-record strings will not parse — see the rejected `retreat-clock`), `swipe` (it compares
  two views of the same thing; the two IR cycles differ by draft status, not by measurement),
  `routing`/`isochrones`/`near-me` (a watershed is fixed by topography; a drive time means nothing to
  it), `draw` (a hand-sketched catchment is exactly the invented boundary this app argues against),
  `elevation` (a longitudinal profile is the energy recipe's instrument), `views` (the descent block *is*
  the navigation), editing (**no write path at all**). Full sweep: `DESIGN-PROPOSAL.md` §8.
- **Composed export** — a board **PDF** (legend + scalebar + north-arrow + the printed derivation + the
  three books as a table), a **per-hydrologic-region atlas** (12 pages), a per-watershed **feature
  report** sized to drop into a WSA appendix, CSV/GeoJSON, and `exportSpec` so the reading round-trips
  to ArcGIS.

---

## 3. Data sources

All EPSG:4326 (reproject on ingest — SWRCB hosted layers are `wkid 3857`). Every row below was `curl`'d
on **2026-08-10**; counts are literal `returnCountOnly` / `outStatistics` responses. Full provenance,
schemas and traps: **`watershed-catalog-ca.json`** (16 external feeds, 13 verified-live).

### The discovery — a tenth California server, and the largest water estate in the state

California's crawled public estate — all **2,532** endpoints in `data_sources_ca.md` — contains, when
swept for this recipe's eleven water roles, exactly **two** `measurement` services (one DWR Hydstra
station layer, listed twice as FeatureServer and MapServer) and **two** `claim-register` services (one
DWR adjudicated-areas layer, likewise). **No stream-gage network. No diversion register. No Integrated
Report.** But the **State Water Resources Control Board** runs an ArcGIS server the catalogue knew only
as a single unannotated URL buried in one recipe's feed list, and never enumerated:

> **`https://gispublic.waterboards.ca.gov/portalserver/rest/services`** — **28 folders + 86 root
> services + a `Hosted` folder carrying 569 FeatureServers**: water rights, water quality, groundwater
> quality (GAMA), drinking water, the Integrated Report and the SB 19 stream-gage record, on one host.
> **CORS is open** (`Access-Control-Allow-Origin` reflects the request Origin, `Allow-Credentials:
> true`) — browser-direct, no proxy.

⛔ **And the sibling path on the same host is WAF-blocked.** Everything under
`gispublic.waterboards.ca.gov/**/arcgis/**` — including every eWRIMS URL a search engine will hand you —
returns an **Imperva/Incapsula HTML challenge page under HTTP 200**. Not a 403: a naive client reports
success and a JSON parser reports a syntax error. **Use `/portalserver/`.**

| Role | California — SWRCB (the discovered server) | California — the crawled estate | National / engine |
|---|---|---|---|
| **Watershed frame** | `Stream_Gage_Prioritization_Analysis_by_HUC12_Watershed_(View_2)/0` — **4,469** HUC12 (`huc12`, `huc12_name`, `huc10`, `tier`, `primary_benefit`, `huc12_prop_needgage`, `dac_type`) · `Hydrology/CalWater_Boundaries` — 6 levels, **7,008** planning watersheds, **203** hydrologic units | DWR `i03_Hydrologic_Regions` (**12**) · Caltrans `DEA_Hydrologic_Unit_Codes`, `DEA_CalWater_Boundaries` | USGS **WBD/NHDPlus** |
| **Measurement — the evidence** | `California_Stream_Gages_(View1)/0` — **2,597** sites; Inactive **1,196**, real-time flow **724**, USGS-operated **2,078** · `gagegap_flowlines_sb19_(View)/0` — **139,837** flowlines, `gagegap_status` never-gaged **124,928 (91.3 % of km)** | ⛔ **DWR `i08_Stations_Monitoring_Continuous_Hydstra_Period` only** — a station index, not a network | **USGS NWIS** `waterservices.usgs.gov/nwis/iv` — keyless, CORS `*`, 516 CA sites |
| **Claim register — the promise** | **`Water_Rights/Points_of_Diversion/0` (eWRIMS) — 63,990 PODs keyed to `HUC_12`/`HUC_8`**, the app's primary register when the WAF lets it through · `DensityOfDiversions/0` (`PODsPerHUC10`) — **1,128** HUC10, **60,569** PODs, the always-available fallback · `FASS_Streams/MapServer` — **55** year-round / **304** all fully-appropriated reaches | ⛔ DWR `i03_Adjudicated_Areas` only | — |
| **Claim volumes** | ⛔ **Not published at all.** The eWRIMS layer is reachable and has **39 fields, none a quantity of water** — no face value, no acre-feet, no rate. This is a gap in the *record*, not only in the *access*. | ⛔ none | ⛔ **`data.cnra.ca.gov` and `data.ca.gov` CKAN both 403** |
| **Quality assessment** | `Final_2026_Rivers_and_Streams/0` — **4,170** segments, **2,235 Listed** · `Final_2026_Lakes_and_Reservoirs/0` — **905**, **581 Listed** · `NHD_Not_Assessed/0` — **1,603,153** reaches never assessed | 23 role-tagged (Cal OES `Impaired_Watercourse_(303D_Listed)`, `Basin_Plan_Poly_*`, CAL FIRE `Watercourses_303d`) | EPA ATTAINS |
| **Legal use constraint** | `Basin_Plan/California_Basin_Plan_Beneficial_Uses` — **2,618** polygons + **4,676** lines (`MUN`/`COLD`/`SPWN`/`REC1`/…) | Cal OES `Basin_Plan_Region_*` (9 regional copies) | — |
| **Groundwater — the other partition** | `Probationary_Basins` (2) · `Inadequate_Basins` · `GAMA` folder (**60** services) · `Water_Quality_Risk_2026_ARM` | **DWR `i08_B118_CA_GroundwaterBasins` — 515 subbasins** · `i08_CriticallyOverdraftedBasins` (⚠ query INTERMITTENT — 21 subbasins when it answers) · `i03_Groundwater_Sustainability_Agencies` · 72 role-tagged groundwater services | — |
| **Demand base** | `Drinking_Water/California_Drinking_Water_System_Area_Boundaries/0` (**SABL, 4,981**, OID `OBJECTID_1`) · `Public_Water_Systems_SAFER_Status_ARGQ` (**4,993**) | Cal OES republication (OID `FID`, truncated names) · DWR `i03_WaterDistricts` | — |
| **Supply / storage** | `Precipitation_Stations_CDEC_ARGQ`, `CA_Average_Annual_Precipitation_1981_to_2010` | CA Geoportal `AtmosphereClimate/CDEC_Stations` · Cal OES `CA_Reservoirs_Final`, `DWR_Reservoir_Storage_View`, `CA_Drought_Monitor_view` | NRCS SNOTEL, NOAA |
| **Reporting frame** | `California_Regional_Water_Quality_Control_Board_boundaries` — **9** | CA Geoportal `CA_Counties` (58) · 150 role-tagged frames | Census TIGERweb |

**CORS:** the SWRCB `portalserver` and DWR both reflect the request Origin; Cal OES / CAL FIRE hosted
orgs are `*`; USGS NWIS is `*`. **Browser-direct, no proxy needed.** Carry State Water Board, DWR, USGS
and OpenStreetMap attribution plus "no warranty", and state on screen that **availability is modelled
wherever no gage exists** and that **claim counts are not volumes**.

### The finding, measured

| | |
|---|---|
| HUC12 watersheds (SB 19 frame) | **4,469** |
| …with an active flow gage / real-time flow gage | **559 (12.5 %)** / 512 (11.5 %) |
| …**ungaged** | **3,910 (87.5 %)** |
| Independent corroboration — mean `huc12_prop_needgage` | **0.868** |
| HUC key hygiene — String, full-width, zero nulls, across all 8,194 rows | ✅ no padding step needed (W4) |
| Tier-1 priority watersheds / still ungaged | **486** / **382 (78.6 %)** |
| Tier-2 priority watersheds / still ungaged | 426 / 331 (77.7 %) |
| Stream length never gaged | **239,521 km of 262,259 km (91.3 %)** |
| Flowlines: never gaged / well-gaged / **inactive gage** / almost | 124,928 / 8,023 / **5,400** / 1,486 |
| Gages: Inactive / Active-High-Quality / Active-Limited / eliminate | **1,196** / 662 / 359 / 286 |
| Points of diversion statewide | **60,569** across 912 of 1,128 HUC10s |
| **PODs in watersheds with no active flow gage** | **22,676 (37.4 %) in 557 watersheds** |
| PODs in watersheds with no gage record at all | 7,646 (12.6 %) in 284 watersheds |
| **The retreat (the difference)** | **24.8 points** |
| IR 2026 rivers assessed / listed | 4,170 / **2,235 (53.6 %)** |
| IR 2026 lakes assessed / listed | 905 / 581 (64.2 %) |
| **NHD reaches never assessed** | **1,603,153** |
| Fully Appropriated Stream reaches (year-round / all) | 55 / **304** |

**Gage coverage collapses upstream** (`gagegap_status` by Strahler order): order 1 **99 %** never gaged ·
order 2 97 % · order 3 84 % · order 4 59 % · order 5 40 % · order 6 42 % · order 7 **19 %**.

**The six partitions, none of which nest:** CalWater planning watersheds **7,008** · HUC12 **4,469** ·
HUC10 **1,128** · CalWater hydrologic units **203** · B118 groundwater subbasins **515** · DWR
hydrologic regions **12** · Regional Water Quality Control Boards **9**.

### Traps that bite (all reproduced, not inferred)

| # | Trap |
|---|---|
| **W1** | **A WAF that answers HTTP 200 — and does so *intermittently*.** `gispublic.waterboards.ca.gov/**/arcgis/**` returns an Imperva/Incapsula HTML challenge page **with a 200 status**, so `curl -w '%{http_code}'` reports success and only the JSON parser fails. **Check the body, not the status code.** ⚠ **And probe it more than once:** research met the challenge on every attempt and recorded the path as permanently blocked; hours later five consecutive probes returned clean JSON, CORS-open. Together with W11 this is the **second** endpoint in this recipe wrongly declared dead from a single sample. The app therefore treats eWRIMS as a *preferred but flaky* source and degrades to the HUC10 register, naming which one answered on screen. |
| **W2** | **`flow_yn='Y'` does not mean the gage reports flow.** 2,325 of 2,597 sites (89.5 %) carry it — it means *capable of*. Only **724 (27.9 %)** have `flow_realtime='Y'`, and 1,196 are `gage_status='Inactive'`. Reading `flow_yn` as coverage overstates the live network by **3.2×**. |
| **W3** | **`tier: null` means *not prioritised*, not *adequately gaged*.** 3,557 of 4,469 watersheds (79.6 %) have no tier. Rendering null as "fine" inverts the finding. |
| **W4** | **The HUC padding trap does NOT exist here — checked, because it usually does.** All three layers store `huc8`/`huc10`/`huc12` as **String**, every value is full-width (8/10/12) and **there are no nulls** — verified across all 8,194 rows (2,597 gages + 4,469 watersheds + 1,128 HUC10s). So no zero-padding step is needed, and one added "defensively" is dead code. **The real adjacent risk is the opposite one:** every scope in this app is a **prefix match** (`huc12 LIKE '18010110%'`), so the columns must stay **strings** — a client that casts a HUC to a number to "clean" it destroys scoping and, for the 14/15/16/17/18 California regions, does so without ever producing a visibly wrong value. |
| **W5** | **A POD count is not an allocation.** One 964-POD watershed may be claimed for less water than one 12-POD watershed with a single project right. The layer may say *how many claims*; it may never say *how much water*. The volume column is W1-blocked. |
| **W6** | **Stream order 9 reads 89 % never gaged and is not a headline.** Order 9 here is Delta/estuarine and tidal channel, where a discharge gage is not the instrument. Exclude or label it, or the app will appear to claim the state does not measure the Sacramento. |
| **W7** | **`FASS_Streams/MapServer/0` has `objectIdField: null`** and a four-column schema (`OBJECTID, SHED_ID, SHAPE, SHAPE_Length`). The one binding statement California makes about surface-water availability carries **no stream name, no season, no order date**. Bind read-only on `SHED_ID`; treat the absence as content. |
| **W8** | **`NHD_Not_Assessed` is 1,603,153 rows against `maxRecordCount` 2,000.** Never page it client-side — that is an 800-request mistake that reads as the app hanging. Use it as a rendered backdrop or a `returnCountOnly` denominator only. Its OID is **`fid`**, not `OBJECTID`. |
| **W9** | **`listingstatus='Not Listed'` ≠ clean.** It means assessed and not listed, against an assessed set of 4,170 in a network of 1.6 M reaches. Never paint it green without the denominator beside it. |
| **W10** | **MapServer service roots 400 on `/query`.** `CalWater_Boundaries/MapServer/query` → `{'code':400,'Invalid URL'}`; sublayers must be queried at `/MapServer/{n}/query`. That 400 reads like a dead service and is not one. |
| **W11** | **`i08_CriticallyOverdraftedBasins` is INTERMITTENT, and the first reading was wrong.** Its query endpoint returned `{'code':500,'Error performing query operation'}` on three consecutive probes during research and answered `{'count':21}` on three consecutive probes an hour later — so the honest contract is *flaky*, not *broken*, and the recipe's first draft ("do not put a KPI on it") over-claimed from a single sample. **Probe an endpoint more than once before recording it as dead**, and give any KPI bound to it a degradation path rather than a blank. `test-basin.mjs` §8 passes on either outcome and fails only on a wrong count. |
| **W12** | **Three OID conventions in one app.** SWRCB SABL is **`OBJECTID_1`** (a separate `OBJECTID` also exists and diverges); `NHD_Not_Assessed` is **`fid`**; `FASS_Streams/0` is **`null`**. The house rule (`OBJECTID`) holds on none of them. |
| **W13** | **`operator` is free text.** 'DWR' (181) and 'CA Dept of Water Resources' (48) and five more spellings are one agency; a group-by fragments it and understates DWR while USGS's 2,078 stands correctly at 80 %. |
| **W14** | **Multiple Integrated Report vintages on one server** (`Final_2026_*`, `Draft_2024_*`, `Proposed_2024_*`, `IR2020_CA_303d_DRAFT`). Bind the `Final_2026_` layers or two readers cite different lists. `wbid` carries **no HUC**, so every watershed roll-up needs a spatial join. |
| **W15** | **569 FeatureServers in the `Hosted` folder.** Never point `/add-data` at the folder root — the same class as CPUC's 855-layer Drilldown. Curate; offer a URL box for the rest. |
| **W16** | **Three vintages of B118** (`_2003`, `_2016`, unsuffixed). Pin the vintage in the layer title or two people compute different basin totals. (`environment_agriculture-land-use` recorded this first.) |
| **W17** | **`data.cnra.ca.gov` 403s exactly like `data.ca.gov`.** The state's designated open-data route to water-rights volumes is closed to non-browser clients — the **third** recipe to hit this standing Cloudflare block, and the first to confirm CNRA's own portal is included. Policy, not an outage. |
| **W18** | **The catalogue's own noise trap, reconfirmed.** CAL FIRE names one service **per wildfire** (`FS_RIVER_2026_CAKRN_000005`, `FS_LAKE_2025_…`), so a `\briver\b`/`\blake\b` sweep for hydrology pulls in fire perimeters; `Water_Drafting_Sites` and `Helicopter_Water_Drops` are firefighting logistics. `build_watershed_catalog.py` rejects them by naming signature and logs each rejection. |
| **W19** | **Our own tagger bit us.** The bare substring `fass` (for Fully Appropriated Stream Systems) matches **`CalifAssemSenDists`** and **`ReadyForWildfire_SelfAssessment`** — two services with no connection to water rights. Fixed to `\bfass\b`; recorded because the next recipe writing a role regex will make the same mistake. |
| **W20** | **An emptiness check that lies.** On the eWRIMS POD layer, `HUC_12 IS NOT NULL` matches all **63,990** rows and `HUC_12 <> ''` matches **0** — while every value is a real 12-digit code. The empty-string comparison returns a confident zero on a fully populated column, so a client that filters "blank" rows this way silently discards the entire register and reports success. Test `IS NOT NULL` and check the length client-side. |

---

## 4. Verify each URL first (terminal)

```bash
UA='Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 Chrome/126.0 Safari/537.36'
P=https://gispublic.waterboards.ca.gov/portalserver/rest/services
H=$P/Hosted

# ── THE DISCOVERY: a tenth CA ArcGIS server, absent from data_sources_ca.md's roots table ────
curl -s -A "$UA" "$P?f=json" | python -c "
import json,sys; d=json.load(sys.stdin)
print(len(d['folders']),'folders,',len(d['services']),'root services')"          # → 28 folders, 86 services
curl -s -A "$UA" "$P/Hosted?f=json" | python -c "
import json,sys; print(len(json.load(sys.stdin)['services']),'hosted services')" # → 569

# ── CORS: decisive — can a browser reach it at all? ──────────────────────────────────────────
curl -s -I -A "$UA" -H "Origin: https://example.org" "$P?f=json" | grep -i access-control
                                              # → Access-Control-Allow-Origin: https://example.org

# ── W1 — THE WAF THAT ANSWERS 200. Confirm, never assume. ────────────────────────────────────
curl -s -o /dev/null -w "eWRIMS POD status: %{http_code}\n" -A "$UA" \
  "https://gispublic.waterboards.ca.gov/arcgis/rest/services/Water_Rights/Points_of_Diversion/FeatureServer?f=json"
                                                                    # → 200  ← and it is a LIE
curl -s -A "$UA" \
  "https://gispublic.waterboards.ca.gov/arcgis/rest/services/Water_Rights/Points_of_Diversion/FeatureServer?f=json" \
  | head -c 80                                # → <html ...>_Incapsula_Resource...  ← the real answer

# ── MEASUREMENT: the gage inventory, and how little of it is switched on ─────────────────────
G=$H/California_Stream_Gages_\(View1\)/FeatureServer/0
gb(){ curl -s -A "$UA" -G "$1/query" --data-urlencode "where=1=1" \
  --data-urlencode "groupByFieldsForStatistics=$2" \
  --data-urlencode 'outStatistics=[{"statisticType":"count","onStatisticField":"objectid","outStatisticFieldName":"n"}]' \
  --data-urlencode "returnGeometry=false" --data-urlencode "f=json"; }
gb "$G" gage_status    # → Inactive 1196 | Active-High Quality 662 | Active-Limited Use 359 | ... TOTAL 2597
gb "$G" flow_realtime  # → N 1862 | Y 724 (27.9%)  ← the live network
gb "$G" flow_yn        # → Y 2325 (89.5%)          ← W2: 'capable of', not 'reports'
gb "$G" operator       # → USGS 2078 (80.0%)       ← federally operated

# ── THE FRAME + THE STATE'S OWN ADMISSION ────────────────────────────────────────────────────
S=$H/Stream_Gage_Prioritization_Analysis_by_HUC12_Watershed_\(View_2\)/FeatureServer/0
curl -s -A "$UA" -G "$S/query" --data-urlencode "where=1=1" \
  --data-urlencode "returnCountOnly=true" --data-urlencode "f=json"              # → 4469
gb "$S" tier           # → null 3557 (79.6%) | 1: 486 | 2: 426   ← W3
curl -s -A "$UA" -G "$S/query" --data-urlencode "where=1=1" \
  --data-urlencode 'outStatistics=[{"statisticType":"avg","onStatisticField":"huc12_prop_needgage","outStatisticFieldName":"a"}]' \
  --data-urlencode "f=json"     # → 0.868  ← the state's OWN column corroborating 87.5% independently

# ── THE NETWORK, PAINTED BY WHETHER ANYONE MEASURES IT ───────────────────────────────────────
F=$H/gagegap_flowlines_sb19_\(View\)/FeatureServer/0
curl -s -A "$UA" -G "$F/query" --data-urlencode "where=1=1" \
  --data-urlencode "groupByFieldsForStatistics=gagegap_status" \
  --data-urlencode 'outStatistics=[{"statisticType":"count","onStatisticField":"objectid","outStatisticFieldName":"n"},{"statisticType":"sum","onStatisticField":"length_km","outStatisticFieldName":"km"}]' \
  --data-urlencode "f=json"
  # → never gaged 124,928 / 239,521 km (91.3%) | well-gaged 8,023 | inactive gage 5,400 | almost 1,486

# ── THE CLAIM REGISTER (counts only — W5) ────────────────────────────────────────────────────
D=$H/DensityOfDiversions/FeatureServer/0
curl -s -A "$UA" -G "$D/query" --data-urlencode "where=1=1" \
  --data-urlencode 'outStatistics=[{"statisticType":"sum","onStatisticField":"pod_count","outStatisticFieldName":"pods"},{"statisticType":"count","onStatisticField":"objectid","outStatisticFieldName":"n"}]' \
  --data-urlencode "f=json"                              # → 60,569 PODs across 1,128 HUC10

# ── THE HEADLINE JOIN: claims x measurement. Fully paged, zero-padded (W4). ──────────────────
# Lives in the app's own suite so there is ONE implementation, not a probe that can drift.
cd app && node test-basin.mjs
 # HUC10 1,128 | gages 2,597 | active-flow watersheds 361 | ungaged 767 (68.0%)
 # PODs in UNGAGED watersheds 22,676 of 60,569 = 37.4%  (557 watersheds still divert)
 # against ANY gage record incl. inactive: 7,646 = 12.6%   →  THE RETREAT = 24.8 points
 # HUC12: 559 of 4,469 have an active flow gage (12.5%);  Tier 1: 382 of 486 ungaged (78.6%)

# ── QUALITY: the smallest book, and the denominator that makes it readable ───────────────────
c(){ curl -s -A "$UA" -G "$1/query" --data-urlencode "where=${2:-1=1}" \
     --data-urlencode "returnCountOnly=true" --data-urlencode "f=json"; echo; }
c $H/Final_2026_Rivers_and_Streams/FeatureServer/0                       # 4170
c $H/Final_2026_Rivers_and_Streams/FeatureServer/0 "listingstatus='Listed'"   # 2235 (53.6%)
c $H/Final_2026_Lakes_and_Reservoirs/FeatureServer/0                     # 905
c $H/Final_2026_Lakes_and_Reservoirs/FeatureServer/0 "listingstatus='Listed'" # 581 (64.2%)
c $H/NHD_Not_Assessed/FeatureServer/0                                    # 1,603,153  ← W8, W9

# ── THE LEGAL INSTRUMENTS ────────────────────────────────────────────────────────────────────
c $P/FASS_Streams/MapServer/0     # 55 year-round      ← W7: objectIdField is null
c $P/FASS_Streams/MapServer/1     # 304 all
c $P/Basin_Plan/California_Basin_Plan_Beneficial_Uses/MapServer/0   # 2618 use polygons
c $P/Basin_Plan/California_Basin_Plan_Beneficial_Uses/MapServer/1   # 4676 use lines
c $P/Hydrology/CalWater_Boundaries/MapServer/0    # 7008 planning watersheds   ← W10 on the root
c $P/Hydrology/CalWater_Boundaries/MapServer/5    # 203 hydrologic units

# ── THE OTHER PARTITION (it does not nest — that IS the point) ───────────────────────────────
W=https://gis.water.ca.gov/arcgis/rest/services
c $W/Geoscientific/i08_B118_CA_GroundwaterBasins/FeatureServer/0     # 515 subbasins
c $W/Boundaries/i03_Hydrologic_Regions/FeatureServer/0               # 12 regions
# W11 — PROBE IT THREE TIMES. One sample recorded this as dead; it is merely flaky.
for i in 1 2 3; do c $W/Geoscientific/i08_CriticallyOverdraftedBasins/FeatureServer/0; done
  # → {"count":21} x3 on 2026-08-10 17:0x, {"code":500} x3 an hour earlier.

# ── THE BLOCKED SOURCES — CONFIRM, never design around a guess ───────────────────────────────
curl -s -o /dev/null -w "data.cnra.ca.gov: %{http_code}\n" -A "$UA" \
  "https://data.cnra.ca.gov/api/3/action/package_search?q=eWRIMS"          # → 403   ← W17
curl -s -o /dev/null -w "data.ca.gov:      %{http_code}\n" -A "$UA" \
  "https://data.ca.gov/api/3/action/package_search?q=water+rights"         # → 403
curl -s -o /dev/null -w "waterrightsmaps:  %{http_code}\n" -A "$UA" \
  "https://waterrightsmaps.waterboards.ca.gov/gisapp/rest/services?f=json" # → 503

# ── CALIFORNIA'S CRAWLED ESTATE: prove the absence that justifies this recipe ────────────────
python build_watershed_catalog.py
 # → library 2532 · role-tagged 342 · measurement 2 · claim-register 2
 #   ASSERTED EMPTY: statewide-diversion-volume  holds
 #   ASSERTED EMPTY: statewide-water-right-register  holds
```

**Real field names that drive symbology / joins / KPIs.** Gages: `siteid`, `sitename`, **`gage_status`**,
`operator`, **`flow_yn`**, **`flow_realtime`**, `flow_por`, `huc8`/`huc10`/`huc12` (**unpadded**),
`gagegap_status`, `tier`, `primary_benefit`, `sb19_action_recommended`, `site_id_reactivate`,
`totdasqkm`. Watersheds: **`huc12`**, `huc12_name`, `huc10`, `huc10_name`, **`huc12_prop_needgage`**,
`huc12_gg_combined`, `ecosystem_rank`/`waterquality_rank`/`watersupply_rank`/`flood_rank`,
**`tier`**, `dac_type`. Flowlines: `comid`, `gnis_name`, **`gagegap_status`**, **`drainage_area_sqkm`**,
`length_km`, **`streamorder`**, `huc12`. Diversions: **`pod_count`**, `podsperacre`, `huc10`, `name`,
`areaacres`. Integrated Report: `wbname`, `wbid`, `wbtype`, **`listingstatus`**,
`integrated_report_category`, `newpollutantslisted`, `hyperlink`. FAS: **`SHED_ID`** only. SABL:
**`OBJECTID_1`**, `SABL_PWSID`, `WATER_SYSTEM_NAME`, `POPULATION`, `COUNTY`. B118: `Basin_Subbasin_Name`,
`Basin_Subbasin_Number`, `Area_SqMiles`, `Date_Data_Applies_To`.

---

## Guided wizard — **the prompts that assign the app's defaults**

Claude runs this like an ArcGIS Instant-App wizard: ask each group, **apply the default so "accept all"
builds a complete app**, confirm a one-line summary, then run §5. Launch with
`/recipe hydrology_watershed-and-water-resources-portal`. Every answer *sets an application default*
baked into `layers.json` / the `AppLayout`. Phrasing from `strata/docs/reference/human-language.md`.

| # | Wizard question | Options → **default** | The default it assigns |
|---|---|---|---|
| 1 · App | Title & as-of date? | free text → **"Water Resources"**, today | header title + `strata:notes.asOf` |
| 2 · Region | Which area? | **Russian River HUC8 (CA demo)** · another HUC8 · a DWR hydrologic region · statewide · my own extent | initial extent, the `huc` prefix, how much is paged live |
| 3 · **"Measured" means** | **What counts as a measured watershed?** | **an active gage that measures flow** · a real-time flow gage · any gage record, including inactive ones | the descent's paint field and every KPI's WHERE — *and the app prints that this choice moves the headline from 37.4 % to 12.6 %* |
| 4 · Descent levels | Which levels of the hierarchy? | **HUC8 → HUC10 → HUC12** · HUC10 → HUC12 only · add CalWater planning watersheds | the descent bands, and where the grain notice fires |
| 5 · The grain notice | Show which records can answer at each level? | **yes, persistent under the descent** · on hover only · off | **default yes; turning it off prints a warning** — it is the app's signature |
| 6 · Apportioned claims | Show claim counts at HUC12, where they are not published? | **yes, apportioned by area and labelled italic** · leave blank below HUC10 | whether `claims_n` renders at the finest level, and how |
| 7 · Quality cycle | Which Integrated Report? | **IR 2026 (final)** · IR 2024 (draft), with the draft warning | the `impairment` layer and the listing counts |
| 8 · Quality denominator | Show the never-assessed reach count beside "Not Listed"? | **yes — 1,603,153** · listing counts only | whether the `never-assessed` KPI and the three-books bar render |
| 9 · Legal overlays | Which constraints? `[multi]` | Fully Appropriated Streams · Basin Plan beneficial uses · both → **both** | the `fas` and `benuses` layers and the accordion |
| 10 · Groundwater | Show the B118 partition that does not nest? | **yes, as an explicit mismatch panel** · as a plain overlay · off | the groundwater accordion and the overlap-% computation |
| 11 · Live cross-check | Add USGS NWIS real-time discharge? | **yes, for gages in scope only** · statewide · no | whether `RestDataSource` mounts and the live column renders |
| 12 · Your records | Do you have your own gage or diversion data? | **no → published-record mode** · CSV upload · FeatureServer | whether the volume KPIs render live or greyed with "the volume record is not reachable" |
| 13 · Report/Theme | Board report? · theme/language? | **board PDF + per-region atlas** ; **light / EN** · dark · EN+AR (RTL) | wires `/export report` + atlas; `ThemeSpec` mode + `lang-switch` |

**Then:** Claude echoes *"Russian River HUC8 · measured = active flow gage · HUC8→HUC10→HUC12 · grain
notice on · claims apportioned at HUC12 · IR 2026 · never-assessed shown · FAS + beneficial uses ·
groundwater mismatch panel · NWIS for gages in scope · published-record mode · board PDF · light/EN"*
and, on confirmation, runs §5 — so the app opens **fully configured**.

---

## 5. Prompt-script (run in order)

```
A. /new-app — a "Water Resources" open-design app ("basin-descent", see DESIGN-PROPOSAL.md §3-§6).
   LIGHT ThemeSpec, three-hue semantic system: primary #0e7490 (MEASURED), secondary #7c3aed
   (PROMISED), warning #b45309 text / #f59e0b fill (UNMEASURED — absence, not danger), danger
   #b91c1c (LISTED impaired), success #15803d, fonts.scale compact, mono+tabular-nums override on
   kpi and table, motion 180ms. App-shell: header (title + region picker + "measured means" +
   IR cycle + lang-switch + theme-switch), footer (attribution + "availability is modelled wherever
   no gage exists"), splash stating the thesis in two sentences. Install deps + run command.

B. Build `watersheds` — the frame the whole app hangs on. /convert to EPSG:4326 GeoParquet
   (SWRCB hosted layers are wkid 3857 — request outSR=4326):
     • Stream_Gage_Prioritization_Analysis_by_HUC12_Watershed_(View_2)/0  -> 4,469 HUC12
       carry huc12, huc12_name, huc10, huc10_name, tier, primary_benefit, huc12_prop_needgage, dac_type
     • Keep every HUC column a STRING (W4). They arrive correctly padded, so no cleanup is
       needed — but every scope in this app is a prefix match (huc12 LIKE '18010110%'), which a
       numeric cast would silently destroy.
   /analyze dissolve HUC12 -> HUC10 -> HUC8 to build the three descent levels, summing
   drainage area. /publish (folder=california, service=watershed).

C. Build `gaged_area_share` — the ink on the descent, and the number nobody else reports.
   /add-data gagegap_flowlines_sb19_(View)/0 (139,837 flowlines; comid, drainage_area_sqkm,
   gagegap_status, streamorder, huc12). Per watershed:
       gaged_area_share = SUM(drainage_area_sqkm WHERE gagegap_status IN ('well-gaged',
                              'almost well-gaged')) / SUM(drainage_area_sqkm)
   ⚠ EXCLUDE streamorder = 9 (W6 — Delta/estuarine channel; including it makes the app appear to
     claim the Sacramento is unmeasured). Record the exclusion in the derivation panel with a toggle.
   ASSERT against §4: never-gaged 124,928 flowlines / 239,521 km (91.3%).

D. Build `reading` — the three books. /add-data:
     • California_Stream_Gages_(View1)/0 (2,597) -> `gages`; derive gages_active_flow =
       gage_status LIKE 'Active%' AND flow_yn='Y'.  ⚠ NOT flow_yn alone (W2).
     • DensityOfDiversions/0 (1,128 HUC10, 60,569 PODs) -> `claims`. Publish claims_n at HUC10;
       at HUC12 apportion by area and FLAG IT (`claims_apportioned = true`) — the grain notice and
       the table's italic both read that flag.  ⚠ NEVER derive a volume from a count (W5).
     • Final_2026_Rivers_and_Streams/0 (4,170) + Final_2026_Lakes_and_Reservoirs/0 (905) ->
       `impairment`; spatial-join to HUC12 (wbid carries no HUC — W14).
     • NHD_Not_Assessed/0 -> `unassessed` COUNT ONLY, never paged (1.6M rows vs maxRecordCount
       2,000 — W8). OID is `fid`.
   ASSERT the marginals against §4: 22,676 PODs (37.4%) in watersheds with no active flow gage;
   7,646 (12.6%) against any-gage-ever; the retreat = 24.8 points; 382 of 486 Tier-1 ungaged.

E. Build the constraint + context layers. /add-data:
     • FASS_Streams/MapServer/0 (55 year-round) + /1 (304 all) -> `fas`
       ⚠ objectIdField is NULL and the schema is 4 columns (W7) — bind read-only on SHED_ID and
         render the missing season/date as CONTENT in the popup, not as a blank field.
     • Basin_Plan/California_Basin_Plan_Beneficial_Uses/MapServer 0 (2,618) + 1 (4,676) -> `benuses`
     • DWR i08_B118_CA_GroundwaterBasins/0 (515) -> `b118`; /analyze spatialJoin b118 x HUC12 to
       compute overlap_pct — the mismatch is the panel's content, not a defect to fix.
     • SABL Drinking_Water/.../FeatureServer/0 (4,981, OID OBJECTID_1 — W12) -> `pws`
   NEVER point /add-data at the SWRCB Hosted folder root — it has 569 FeatureServers (W15).

F. /symbology + /popup — genuine ESRI JSON on verified fields only.
   watersheds: classBreaks on `gaged_area_share` (an arcade valueExpression may derive the class),
     teal #0e7490 -> amber fill #f59e0b; fill alpha 40/255; a hatched overlay where
     claims_n > 0 AND gaged_area_share = 0.
   flowlines: uniqueValue on `gagegap_status` (well-gaged teal · inactive gage amber-dashed ·
     never gaged pale amber). gages: esriSMSCircle — NEVER esriSMSPath — filled where active,
     hollow where inactive. fas: heavy violet casing.
   /popup watersheds: name, HUC, claims (with "count only — not a volume"), active flow gages,
     gaged_area_share as a PERCENT WITH ITS DENOMINATOR, SB 19 tier (and "not prioritised" where
     tier is null — never "OK"), listed/assessed segments, and a literal line reading
     "Not Listed means assessed and not listed; 1,603,153 reaches are not assessed at all."

G. The descent + the three books.
   ① SHIP THE FALLBACK: /panel table over `watersheds` grouped by level (level, name, claims_n,
      gaged_area_share, tier) whose rowSelect carries {field:"<level field>", value:"<huc>"}.
      Register the app-local `basin-descent` widget later (DESIGN-PROPOSAL.md §9) — the loop is
      identical, only the picture changes.
   ② /panel statistics as the three books: PROMISED (violet), MEASURED (teal), ASSESSED
      (red/green), reaches-never-assessed (amber), + a radial gauge for gaged_area_share
      (thresholds 25/60, inverted). All kpi/gauge `stat:{field,op}` on sourceId `reading` — no
      connections. Volume KPIs render GREYED with "the volume record is not reachable".
   ③ ONE stacked-bar: the three books to scale (4,170 assessed vs 1,603,153 not). No chart rail.
   ④ carto category widget on huc8_name = the region picker.
   ⑤ accordion: beneficial uses · groundwater mismatch · WHAT THIS CANNOT SHOW · derivation ·
      board paragraph.
   ⑥ /panel table `watersheds` (virtualized, CSV/GeoJSON, amber-tinted where claims-without-gage,
      ITALIC where claims are apportioned).
   ⑦ interactive legend with live counts, isolate-on-click, and the hint that isolating does not
      change the reading.

H. WIF + controls + export: author AppLayout.connections — the 14 in DESIGN-PROPOSAL.md §5, with
   the SIGNATURE LOOP first (descent categorySelect -> filter + zoomTo on map, filter on table,
   setUrlParam, and the grain-notice message). Map controls navigation/scale/legend/
   basemapSwitcher/layerList positioned top-right so the native cluster clears the books rail;
   status-bar EPSG:4326. Wire /export report (board PDF: legend + scalebar + north-arrow + the
   PRINTED DERIVATION + the three books as a table), a per-hydrologic-region atlas (12 pages),
   /export image, /export layer csv, and a share deep-link (?huc=). Verify responsive.small
   collapses the splitters and that THE DESCENT SURVIVES INTACT (horizontal scroll, never a list).
```

---

## 6. Verify (benchmark to Esri/Arc Hydro · AQUARIUS · KISTERS · Aquaveo/WEAP · StreamStats)

| Check | Pass |
|---|---|
| A tenth CA ArcGIS server discovered, enumerated (28 folders · 86 root · 569 hosted) and CORS-verified | ✅ verified 2026-08-10 |
| W1 confirmed, not assumed: the `/arcgis/` path returns an Imperva page **under HTTP 200** | ✅ body inspected on both FeatureServer and MapServer |
| California's crawled estate proven near-empty of water measurement (2,532 endpoints; **2** measurement, **2** claim-register services) | ✅ asserted in `build_watershed_catalog.py`; build reports it every run |
| 87.5 % of HUC12 watersheds have no active flow gage — **corroborated independently** by the state's own `huc12_prop_needgage` (0.868) | ✅ two routes, two columns |
| 37.4 % of licensed diversions sit in watersheds with no active flow gage; 12.6 % against any-gage-ever; **the retreat = 24.8 points** | ✅ live, HUC10 join, zero-padded |
| 382 of 486 SB 19 **Tier-1** watersheds still ungaged (78.6 %) | ✅ live |
| 91.3 % of stream length never gaged; coverage collapses upstream (order 1: 99 % → order 7: 19 %) | ✅ live groupBy |
| Order-9 excluded and the exclusion documented + toggleable (W6) | ⛏ design; assert at build |
| 1,603,153 reaches never assessed, shown as the denominator beside every "Not Listed" | ✅ count verified |
| Every blocked source CONFIRMED: eWRIMS WAF-200, data.cnra 403, data.ca.gov 403, waterrightsmaps 503 | ✅ all four |
| Every `layerId` + field verified against the live service (§4); no field written from memory | ✅ |
| Light-vs-dark decided by MEASUREMENT; the amber AA failure it exposed is fixed by splitting the token | ✅ contrast + ΔE computed |
| Six non-nesting partitions counted and shown as a panel rather than joined away | ✅ 7,008 / 4,469 / 1,128 / 515 / 203 / 12 / 9 |
| ≥3 `connections` fire; the signature loop works end-to-end | ✅ driven — **31 wired behaviours**; cell → map + rail + table + band + URL |
| The grain notice fires at every descent level and names the apportioned figure | ✅ driven — 0 estimated at HUC8/10, exactly 1 at HUC12 |
| Apportionment CONSERVES the published total and every estimate is labelled | ✅ 200+300 → 500 across all three levels; italic + tooltip |
| The honesty control moves the headline and prints the cost | ✅ driven — Russian River 25→14 unmeasured subwatersheds, a ~23.6-pt local retreat |
| `responsive.small` collapses every side-by-side row; the descent survives intact | ✅ CSS asserted (scrolls, never re-flows) — **not** visually confirmed at phone width |
| Basemap keyless; everything EPSG:4326; three hues never cross semantics | ✅ asserted — every vertex inside California, no keyed provider anywhere |
| Runs on Strata **and** ArcGIS (no write path at all) | ✅ by construction |
| Application built and driven | ✅ `app/` — **335 assertions** (128 live + 118 offline + 89 in real headless Chrome) |
| The map paints, the descent has clickable geometry, the console is clean | ✅ driven — 21 blocks, 8 hatched, 0 console errors, 0 failed requests |
| Every KPI is a toggle: select a population, click again to clear, one at a time | ✅ driven — 7 → 2 rows and back, `aria-pressed` both ways |
| Selecting is a toggle in the DESCENT too — a second click releases to the parent level | ✅ driven — 1801011004 → 18010110 |
| A table row selects on click and clears on a second click, WITHOUT changing scope | ✅ driven — the row survives its own click; double-click is the descend gesture |
| The layer drawer offers the whole crawled estate — 264 role-tagged queryable FeatureServers | ✅ driven — 10 role groups, search narrows 65 → 32, adding one really puts layers on the map |
| Only QUERYABLE FeatureServers are offered — no checkbox that ticks and paints nothing | ✅ generated by `build_watershed_catalog.py`, so the drawer and `data_sources_ca.md` cannot drift |
| Map chrome matches the mining recipe's vocabulary (one cluster · SVG icons · drawer beside it) | ✅ driven — 6 buttons, 6 SVGs, MapLibre's own cluster suppressed, drawer geometrically clear of it |
| The legend is a control surface with live counts, and isolating it does not move the reading | ✅ driven — byte-for-byte identical rail before and after |
| Dark mode: the descent stays readable and every control has explicit text colour | ✅ driven — label stays `rgb(16,32,43)` on its data-coloured fill; selects `rgb(229,231,235)` |
| The shipped inline module parses before any markup assertion runs | ✅ `node --check` guard, negative-tested |
| No silent truncation anywhere — paging, and the descent's own band cap | ✅ paged flowlines match the server count; overflow prints “+N more” |
| The `<StrataApp>` / `AppLayout` path | ⛏ authored in §2.1 but **unrendered** — `strata/node_modules` is absent from this checkout |
| Composed PDF / atlas export | ⛏ **not built** — belongs to the `<StrataApp>` path; CSV + board paragraph ship |
| AR/RTL binding | ⛏ **not built** — and declared decorative for this audience (§2.2) |

**On-par-or-better:** matches the watershed vocabulary of Arc Hydro and the assessment vocabulary of
the Integrated Report estate, and **exceeds every product named above on the one axis none of them can
compete on — stating, per watershed and per level of the hierarchy, whether the availability and
quality figures are measured or modelled, and how many licensed diversions sit on streams nobody
gauges** — plus the AI-authored build, the sovereign/on-prem posture, cross-widget interactivity, a
one-click board report, MIT, on Strata or ArcGIS. **Honestly less than:** AQUARIUS / WISKI / Hydstra on
time-series management, rating curves and QA of continuous records (this app manages no time series at
all); Aquaveo / DHI / WEAP on modelling and allocation simulation (it models nothing); StreamStats on
regression-estimated flow statistics for an ungaged site (it deliberately estimates nothing); Arc Hydro
on delineation (it consumes published boundaries); Esri on enterprise integration. **We ship three
published records, on one nested navigation, with the grain of each one printed — and we say so on
screen.**

---

## 7. Harvest (gaps → strata-core)

Log as strata-core issues:

- **`basin-descent` as a core widget** (`DESIGN-PROPOSAL.md` §9) — hierarchical descent as navigation:
  nested blocks, size ∝ magnitude, fill ∝ ratio, click to descend a level. The generic form —
  *"navigate a containment hierarchy, and state at each level which records can still answer"* — would
  serve any recipe whose subject nests: administrative geography, org structure, chart of accounts,
  network hierarchy, product taxonomy. Promote after a second use.
- **A `grain notice` primitive.** This is the app's signature and it is hand-rolled: a bound line that
  states, for the current scope, which of N sources are published at this grain and which are being
  apportioned. Every recipe that mixes sources at different resolutions needs it and none has it.
  Closely related to the **fourth-time-requested** per-widget "assumption" affordance below.
- **Per-widget "assumption" affordance** — a first-class way to attach a printed derivation + source to
  any KPI. This is now the **fifth** recipe to hand-roll it (`education_campus-operations`,
  `real-estate_site-selection`, `marketing_catchment-and-market-share-analyzer`,
  `telecom_underserved-gap-and-take-rate`, this one). It is a pattern, not a one-off.
- **Split the `warning` token in `@strata/theme`.** `#f59e0b` is correct as a fill and fails WCAG AA as
  text on a light panel (2.15:1). **Two recipes have now independently been forced to hand-split it**
  into `--strata-warning-fill` + `--strata-warning`. The theme compiler should derive a
  contrast-corrected text variant per role automatically, exactly as it already derives `-contrast`.
- **An `es` (Spanish) dictionary for `@strata/i18n`.** Second request. The package ships `en`/`ar` only.
- **A "body, not status code" probe in `/add-data`.** A WAF that answers **HTTP 200** with an HTML
  challenge page defeats every status-code check in the toolchain (W1). `/add-data` should sniff the
  content type and first bytes and say *"this host is behind a WAF"* rather than failing as a parse
  error three steps later.
- **A `maxRecordCount` sanity guard.** `NHD_Not_Assessed` is 1.6 M rows against a 2,000 cap. Any attempt
  to page a layer whose count exceeds N× `maxRecordCount` should require an explicit opt-in — the
  telecom recipe asked for a *layer-count* guard; this is the *row-count* twin.
- **String-safety on join keys in `/analyze`.** HUC, FIPS, GEOID and PWSID are fixed-width identifier
  strings that happen to look numeric. This recipe *expected* an unpadded-key trap, tested for it and
  found the data clean (W4) — but the inverse hazard is real and untooled: any step that infers a
  numeric type for such a column breaks prefix scoping without producing a visibly wrong value.
  `/analyze` and `/convert` should treat a fixed-width all-digit column as an identifier by default.
- **Outer-join as a first-class option in `/analyze`.** Restated from the telecom and catchment
  recipes: here, inner-joining gages to watersheds would delete exactly the 87.5 % the app is about.
- **A `layer-state` vocabulary for `LayerPanel`.** Reconfirmed with two new states this recipe found:
  *query capability intermittently failing* (`i08_CriticallyOverdraftedBasins` 500s, then answers) and
  *no OID field at all* (`FASS_Streams/0`). Neither renders as anything but an ordinary ticked box today.

Editing is not applicable — this app has **no write path by design**.

---

## 8. Sources

- **The gaging gap (the recipe's spine):**
  [SWRCB — SB 19 Stream Gaging Plan](https://www.waterboards.ca.gov/waterrights/water_issues/programs/stream_gaging_plan/) ·
  [SB 19 technical report (PDF)](https://waterboards.ca.gov/waterrights/water_issues/programs/stream_gaging_plan/docs/sb19-report.pdf) ·
  [SB 19 appendices (PDF)](https://www.waterboards.ca.gov/waterrights/water_issues/programs/stream_gaging_plan/docs/sb19-appendices.pdf) ·
  [DWR public meeting — SB 19 Stream Gaging Plan](https://water.ca.gov/News/Events/2022/May-2022/Public-Meeting--SB-19-Stream-Gaging-Plan) ·
  [USGS Federal Priority Streamgages](https://www.usgs.gov/mission-areas/water-resources/science/federal-priority-streamgages-fps) ·
  [USGS National Streamgaging Network](https://www.usgs.gov/mission-areas/water-resources/science/usgs-national-streamgaging-network) ·
  [CRS R45695 — USGS Streamgaging Network: Overview and Issues for Congress](https://www.congress.gov/crs-product/R45695)
- **Availability, water rights & over-allocation:**
  [SWRCB — Water Availability Information](https://www.waterboards.ca.gov/waterrights/water_issues/programs/water_availability/) ·
  [SWRCB — Fully Appropriated Stream Systems](https://www.waterboards.ca.gov/waterrights/water_issues/programs/fully_appropriated_streams/) ·
  [SWRCB — Water Rights Applications](https://www.waterboards.ca.gov/waterrights/water_issues/programs/applications/) ·
  [Grantham & Viers — 100 years of California's water rights system (Env. Res. Lett. 2014)](https://cawaterlibrary.net/document/100-years-of-californias-water-rights-system-patterns-trends-and-uncertainty/) ·
  [UC Merced — California overspends water rights by 300 million acre-feet](https://es.ucmerced.edu/news/2014/california-overspends-water-rights-300-million-acre-feet) ·
  [California WaterBlog — You can't manage what you don't measure](https://californiawaterblog.com/2014/08/20/california-water-rights-you-cant-manage-what-you-dont-measure/) ·
  [Trout Unlimited — Water Availability Analysis overview (PDF)](https://www.casalmonandsteelhead.org/wp-content/uploads/2021/08/Water-Availability-Analysis-Presentation_2021-08-17.pdf)
- **Esri (nominative use):**
  [GIS for Water Resources / Watershed Management](https://www.esri.com/en-us/industries/water-resources/overview) ·
  [Arc Hydro](https://www.esri.com/en-us/industries/water-resources/arc-hydro) ·
  [GIS for Water](https://www.esri.com/en-us/industries/water) ·
  [Water utility solutions](https://www.esri.com/en-us/industries/water-utilities/overview) ·
  [Introduction to Water Distribution Data Management](https://doc.arcgis.com/en/arcgis-solutions/11.3/reference/introduction-to-water-distribution-data-management.htm)
- **Non-Esri platforms:**
  [KISTERS WISKI](https://www.kisters.eu/product/wiski/) · [KISTERS Hydstra](https://www.kisters.com.au/product/hydstra/) ·
  [Aquatic Informatics / AQUARIUS](https://aquaticinformatics.com/) ·
  [Aquaveo software](https://aquaveo.com/software) · [WEAP (SEI)](https://www.weap21.org/) ·
  [USGS StreamStats](https://streamstats.usgs.gov/) ·
  [USGS Water Resources software catalog](https://water.usgs.gov/water-resources/water-resources-software-catalog/) ·
  [RTI — water management software & tools](https://www.rti.org/water-resources-tools/water-management-software-tools)
- **Data endpoints (all curl-verified 2026-08-10 — see §4):** SWRCB ArcGIS
  `gispublic.waterboards.ca.gov/portalserver/rest/services` (28 folders · 86 root · 569 hosted) ·
  DWR `gis.water.ca.gov/arcgis/rest/services` ·
  [USGS NWIS instantaneous values](https://waterservices.usgs.gov/nwis/iv/) ·
  `data.cnra.ca.gov` + `data.ca.gov` CKAN (both **403**) ·
  `waterrightsmaps.waterboards.ca.gov` (**503**)
- **Internal:** `DESIGN-PROPOSAL.md` (this recipe's full design record) · `watershed-catalog-ca.json` +
  `build_watershed_catalog.py` · `../APP-TEMPLATE-LIBRARY.md` · `../DESIGN-CONTEXT.md` ·
  `../DESIGN-REQUEST-PROMPT.md` · `../../data_sources/data_sources_ca.md` (§ *Water & Watersheds — [WAT]*) ·
  `../telecom_underserved-gap-and-take-rate/RECIPE.md` (format exemplar; its amber-split, CKAN-403 and
  fire-service-naming findings are all reconfirmed here) ·
  `../marketing_catchment-and-market-share-analyzer/RECIPE.md` ·
  `strata/recipes/COMPONENT-MANIFEST.md` (§10 freestyle charter) · `strata/docs/guide/app-design.md` ·
  `strata/docs/reference/human-language.md`

---

## Modernization (parity release)

> Applies the Experience-Builder-parity features (Phases 1–7). See `strata/recipes/MODERNIZATION.md` +
> `COMPONENT-MANIFEST.md` §8. Cross-cutting: a structured **`theme`**, app-shell
> (`header`/`footer`/`splash`), and `dataSource.{ sourceId }` linking (widgets sharing a source link
> with **no `connections`**).

- **DataSource linking is the backbone, not a nicety.** Nine widgets — four KPIs, the gauge, the
  three-books bar, the grain notice, the derivation block and the board paragraph — bind to `sourceId`
  `reading` and relink on every watershed adoption with **zero** wiring. Without it this app would need
  roughly ten more connections.
- **`recordsChange` + `fromWidget` as the app's spine.** The descent publishes the adopted watershed set
  as the `reading` output source; six widgets consume it. The signature loop is therefore *one* trigger
  feeding both a `filter` fan-out and an output source.
- **Structured `ThemeSpec` doing semantic work.** Three hues encode the three books (`primary` =
  measured, `secondary` = promised, `warning` = nobody measured) and `warning` is redefined as *absence*
  rather than danger, with `danger` held back for a real §303(d) listing — a theme carrying an argument,
  not a palette. `overrides.kpi`/`overrides.table` put counts in mono with `tabular-nums`.
- **App-shell + `splash`.** The splash states the thesis in two sentences before the first render; this
  app's argument has to land before its numbers do.
- **`accordion`** collapses five secondary panels (beneficial uses · groundwater mismatch · what this
  cannot show · derivation · board paragraph) into one rail slot, so the three books stay above the fold.
- **`splitter`** twice (descent ↔ body, map ↔ table) with `responsive.small` collapse — and the explicit
  rule that **the descent survives the collapse intact**.
- **`arcade`** one `valueExpression` deriving the descent's paint class from `gaged_area_share` on the
  live renderer, so the *measured means* toggle repaints without a rebuild.
- **`FileDataSource`** is the client path: an agency's own gage inventory or eWRIMS extract turns the
  greyed volume KPIs live, with no schema change.
- **`RestDataSource`** optionally mounts USGS NWIS for a live discharge column beside the SB 19 inventory.
- **Freestyle charter §10.2** invoked exactly once, for `basin-descent`, with a named `table` fallback
  that preserves the signature loop.
