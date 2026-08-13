# Recipe — Site Selection: **The Swing Slate** (Commerce & Real Estate)

A reproducible path to a **candidate-site ranking** app on strata-core: load your shortlist, pick a
category, set the five weights — and read, in one column, **which site leads, what its score is made
of, and how far your own assumptions would have to move before a different site won**. Household
demand, spending power, traffic exposure, the competitive set and transit access, all from data that
is already public and keyless — plus the California **AB 2097 parking gate** that changes the deal
economics. Two supporting pages carry the rest of the sector's brief: a **site dossier** and an
**open · expand · close** page that runs the same score over an existing estate.

> **Scope (honest).** This is a **transparent screening and ranking**, not a forecast. It computes a
> weighted, normalised score from five criteria you choose, and — the point of the app — the
> **smallest single-weight change that reverses each adjacent pair**. It does **not** forecast sales,
> it is **not** a calibrated Huff model, and it does **not** perform cannibalisation analysis. It
> **does not compute leakage/surplus**: the supply side of that equation needs actual retail
> receipts, and the only public California source (CDTFA) is reported by **permit holder's city, not
> store location** — modelling it would fabricate the one number that moves capital. **It never
> observes foot traffic**: the only visit data in open California holdings is the **19 April –
> 9 May 2020 COVID lockdown window**, which is quarantined behind an explicit badge and used for
> nothing. Every number in the packet carries its source and its vintage. Feature writes (the
> committee decision) need a writable ESRI backend.

Run the §5 prompt-script on a fresh strata-core project (or via `/new-app`). ESRI Web Map JSON is the
map contract; the app is read-only on Strata Serve.

---

## 0. Provenance

| | |
|---|---|
| **Source** | `https://tabaqat.net/solutions` — **Commerce & Real Estate** section |
| **Name on site** | Site Selection |
| **Tagline on site** | "Where to open, expand or close based on catchment and demographics" |
| **Scaffolded** | 2026-07-22 (`select-export` placeholder) |
| **Researched & designed** | 2026-07-28 — see `DESIGN-PROPOSAL.md` |
| **Template** | **`open-design` ("swing-slate")** — the `select-export` assignment is released back to the Commerce & Real Estate sector |
| **Catalog** | `sitesel-catalog-ca.json` — 2,339 CA services, **321 role-tagged** across 11 roles, **18 external feeds (13 curl-verified live)** |
| **Built & driven** | 2026-07-28 — `app/`, **95 live-data assertions green** (81 regression + 14 loop) |

---

## 0.1 The working app in this folder

`app/` holds a **runnable demo of this design** — no build step, no API keys, everything fetched live
from the §3 services.

```bash
cd app && python3 server.py          # -> http://localhost:8052   (or double-click START-APP.command)
node test-slate.mjs                  # 81 assertions against LIVE Cal OES / Caltrans / CAL FIRE data
node drive-slate.mjs                 # 14 assertions: drives the signature loop end-to-end
```

| File | What it is |
|---|---|
| `app/index.html` | **Site Selection** ("The Swing Slate"), self-contained: the assumption rail (trade-area band, five weights that renormalise to 100, missing-input policy), the decomposed slate with flip chips, five live KPIs, the map with on-map basemap + layer controls, the site dossier (9-bucket income distribution, competitor table, CA land-use context), the open·expand·close estate page, the printable committee packet, deep-linkable weights, the jurisdiction read (incorporated city vs. unincorporated county), **six draggable gutters** (rail | slate | map, the two page splits, evidence | provenance, and the strip height), **dark by default** with a light print/counter mode, EN |
| `app/test-slate.mjs` | Regression test that **extracts the shipped functions out of `index.html`** and checks them against live data — planar round-trip, shoelace, Sutherland-Hodgman clipping, areal apportionment, point-in-polygon-with-holes, all three AADT traps reproduced against the live service, the SafeGraph quarantine, normalisation, the missing-input policy, a hand-verified flip solve, and every service's reachability + CORS |
| `app/drive-slate.mjs` | Headless drive of the **signature loop**: measures the five demo sites live, ranks them, then verifies that applying each predicted flip actually reverses its pair — and that *half* the predicted change does not |
| `app/server.py`, `app/START-APP.command` | Zero-dependency static server (MapLibre cannot run from `file://`) |
| `app/sitesel-catalog-ca.json` | Copy of the built catalog |
| `presentation/index.html` | 10-slide deck (house template, self-contained, prints to PDF one slide per page): the gap → the Huff failure mode → the slate → the proof → the CA data spine → the four traps → honest scope → where it sits next to the incumbents |
| `presentation/linkedin-article.md` | ~800-word article + teaser post + a **claims note citing every figure to its source**, with an explicit do-not-claim list and the untested AB 2097 branch volunteered |

**Verified 2026-07-28 — 95 assertions pass:**

| Check | Result |
|---|---|
| Areal apportionment is real, not decorative | ✅ 34 tracts contribute at Elk Grove Blvd, **17 of them partial**; counting them whole would overstate demand by **+35.1%** (57,661 → 42,689 households) |
| AADT string trap reproduced live | ✅ naive `orderBy` returns **"99000"** as the statewide max; `aadtNum()` recovers **335,000**; `where AHEAD_AADT > 100000` errors, `CAST(… AS INTEGER)` returns 3,342 |
| AADT duplication reproduced live | ✅ exactly 2× at every site probed — `30→15 · 32→16 · 20→10 · 26→13 · 18→9` |
| SafeGraph quarantine is **structural** | ✅ service resolves to `CA_051220_WeeklyPatterns`, 21 visit columns `Visits04_19_2020..Visits5_9_2020`; the shipped `POI_FIELDS` allow-list binds none, and none came back on the wire |
| **The headline claim** | ✅ **4/4** predicted flips actually reversed their pair when applied (e.g. #2/#3 gap 7.9 pts → Traffic exposure ↑8.0 → reversed) |
| …and the margin is a real threshold | ✅ **half** the predicted change (Household 30→60.7 of a predicted 91.4) does **not** reverse the top pair |
| The loop is alive, not decorative | ✅ a demand-dominant weighting re-orders the slate (#1 Elk Grove Blvd → Elk Grove Florin, the highest-household site) |
| A ROBUST verdict is trustworthy | ✅ more than doubling the competition weight (20→45) leaves the order intact — exactly as the ROBUST chips predicted |
| Missing inputs are flagged, never zeroed | ✅ flag policy scores 100 where zero policy scores 80; **a bug was found and fixed here** — `aadtNum("")` returned `0`, and 30 rows carry a blank `AHEAD_AADT` (49 a blank `BACK_AADT`), so those sites would have scored *worst possible exposure* instead of *missing* |
| Jurisdiction never renders as a blank | ✅ the demo slate naturally spans both cases — **Elk Grove (incorporated)** and **unincorporated Sacramento County jurisdiction** — because the 483-city layer holds incorporated cities only |
| Live services the app boots against | ✅ tracts 9,107 · POI 82,905 · AADT 13,919 · truck 6,865 · HQTA 20,558 · stops 48,662 · cities 483 · anchors 1,014 — all CORS-reachable |
| Basemaps keyless | ✅ all five return `200 image/png` (Positron · Voyager · Dark Matter · OSM · OpenTopoMap) |
| The app never promises what it cannot compute | ✅ no leakage/surplus anywhere; `Major_Traffic_Generators` and `UCD_Parcels` not bound; scope disclaimer + lockdown explanation ship in the UI |

**What the demo deliberately does not do:** it is a MapLibre app that *renders this design*, not a
`<StrataApp>` — §5 is what produces the real `layers.json` + `AppLayout` on strata-core. Trade-area
bands are **rings** (drive-time needs a routing key — the UI says so when you pick it), the estate
page's open/expand/close thresholds are illustrative rather than a client's hurdle rates, and the
five candidate sites are **demo inputs**, not real deals: in a deployment they arrive by CSV upload,
map pin or address search (wizard Q2).

## 1. Study — how the market frames this

**The question the buyer asks:** *"We can afford one store this year. Which of these five sites — and
how sure are we?"* The second clause is the product. Every incumbent answers the first with a number;
none answers the second, and the second is what the real-estate committee actually argues about.

**The three verbs.** The tagline says *open, **expand** or **close***. **Close** is the verb no
vendor makes first-class, because no vendor wants to sell software that recommends shutting a store.
It is the same score with the candidate set replaced by your own estate — which is why the `estate`
page exists and costs almost nothing to build.

**Reference solutions (benchmark + coexist, never copy).**

- **Esri — ArcGIS Business Analyst** (the incumbent). Its **suitability analysis** workflow
  identifies and scores sites against weighted criteria drawn from the data browser — population
  density, household income, **average annual daily traffic** — combined with fields from the input
  layer, each variable weighted to define its influence. The **June 2026** web-app release doubled
  the **top retailers list to the top 50 US retailers** (for anchor-tenant and brand-presence
  filtering) and added retail as a featured point list in the nearby-analysis workflow; a new **Esri
  Categories** system shipped February 2026. The industry's canonical output is Esri's **Retail
  MarketPlace Profile** and its **Leakage/Surplus Factor** — an index from **+100 (total leakage)**
  to **−100 (total surplus)** measuring demand (household spending potential) against supply (retail
  sales), across the **27 NAICS retail industry groups plus 4 food-service groups**. This is a
  licensed data product inside a licensed platform, run by an analyst.
- **Non-Esri, foot-traffic and AI-scoring:** **Placer.ai** (mobile-location foot traffic and
  predictive analytics; pricing unpublished, third-party-reported enterprise contracts
  **$12,000–$50,000+/yr**), **Buxton** (proprietary "Customer DNA" over **200M+ consumer profiles**,
  sales-potential forecasting, strong cannibalisation and portfolio optimisation, ~**$20k+/yr**),
  **Kalibrate** (machine-learning site selection; natural-language querying via Microsoft Azure AI
  Foundry added April 2025), **SiteZeus** (AI site selection for retail and restaurants; relaunched
  as the conversational "**Atlas**" platform in May 2026), plus **GrowthFactor**, **MapZot.AI**,
  **Geod** and **PassBy**.
- **What the market's own comparison literature says they differ on:** *"scoring transparency, data
  depth, and who can actually run the analysis — a two-person real estate team or a dedicated GIS
  department."* Transparency is named as a top-three axis. That is the axis this recipe takes.
- **The methodological spine — and its documented failure mode.** The academic gold standard is the
  **Huff model** (David Huff, 1963): rather than drawing a boundary, it assigns every demand origin a
  *probability* of patronising each competing store, from attractiveness (mass) and distance —
  which is what makes honest cannibalisation and white-space work possible. Its weakness is the one
  that runs through this entire industry: **the answer is dominated by an assumption nobody
  re-examines.** The feasibility literature is explicit that **the trade-area delineation method, not
  the data, drives the conclusion** — ring, drive-time, gravity and customer-derived boundaries each
  distort demand in a different direction, and the choice between them is rarely revisited once made.
  Practitioner critique of retail leakage analysis (N. David Milder) makes the parallel point about
  the supply side. Both failures share one shape: **a single unexamined assumption, quietly deciding
  everything downstream.**

**Our edge.** AI-authored, MIT/open, runs on **Strata *or* ArcGIS**, sovereign/on-prem — and the
decisive one: **we can show the arithmetic because we have nothing proprietary to protect inside
it.** The incumbents sell a score and must keep the model closed; we sell the score *and its
sensitivity*, which is only sellable if the model is open. Coexistence, never "replace ArcGIS": a
retailer who owns Business Analyst should push its Retail MarketPlace numbers **into** this app as
additional criteria — the criterion list is data-driven, not hard-coded.

**Standards, specifications & organizations to speak fluently.**

- **ICSC** (International Council of Shopping Centers) — the *US Shopping Center Definition Standard*
  and the *Dictionary of Shopping Center Terms, 4th ed.*: **eight principal centre types** and the
  GLA bands that classify a site — **neighbourhood centre 30,000–125,000 sq ft** (supermarket and/or
  large drugstore anchored, convenience for day-to-day needs), **community centre 125,000–400,000
  sq ft** (general merchandise, wider apparel/soft-goods offer), plus the strip/mall configuration
  distinction. **ULI** for the development-economics vocabulary.
- **NAICS** — the classification every retail dataset keys on (and the key the SafeGraph extract
  actually carries).
- **US Census** — ACS 5-year, TIGER/TIGERweb, **County Business Patterns**, the Economic Census;
  **BLS QCEW** for employment.
- **CDTFA** — *Taxable Sales in California*, quarterly, by city and by type of business: the only
  public retail-supply ground truth in the state.
- **Commercial spine** — **CoStar**, **LightBox**, **Regrid** (parcels and lease comps);
  **SafeGraph/Advan**, **Placer.ai**, **Foursquare** (POI and mobility).
- **California statute (§1.1 below)** — **AB 2097**, **AB 2011**, **SB 6**; **HCD** guidance;
  **CEQA**.

### 1.1 What makes California different (and why the demo is CA)

California has legislated the ground under retail siting in a way no other state has:

- **AB 2097** (signed 22 Sep 2022, in force **1 Jan 2023**) — a public agency **may not impose or
  enforce any minimum automobile parking requirement** on a residential, commercial or other
  development within **half a mile of a major transit stop** (an existing rail transit station, a
  ferry terminal served by bus or rail, or the intersection of two or more major bus routes with
  **≤15-minute** peak-period headways). Hotel/motel/B&B/transient-lodging portions are excluded.
  This changes a retail pro forma's land take, site plan and deal economics more than most
  demographic variables do — and **Caltrans publishes the modelled HQTA geometry openly** (§3).
- **AB 2011** — *Affordable Housing and High Road Jobs Act of 2022*, effective **1 Jul 2023**: a
  **ministerial, CEQA-exempt**, time-limited approval path for multifamily housing on
  **commercially zoned** land, including "commercial corridors", conditioned on prevailing wages and
  BMR affordability targets (100% BMR, or mixed-income ~15% BMR on corridors).
- **SB 6** — *Middle Class Housing Act of 2022*, effective **1 Jul 2023**: housing allowed on
  parcels zoned office, retail or parking **without rezoning**, all cities including charter cities.
  Unlike AB 2011 it permits market-rate but is **not ministerial** — discretionary review and CEQA
  still apply, and a skilled-and-trained workforce is required.

**Net effect: the retail pad you are evaluating is simultaneously a by-right housing site.** Land
basis on commercial corridors is now set by a competing use with a statutory fast lane. A California
site-selection app that ignores this is modelling 2019. This recipe **surfaces it as labelled
context on the site dossier and does not model it as a criterion** — there is no open statewide
"commercial corridor" geometry to compute it from, and inventing one would be exactly the sin §1
accuses the incumbents of.

**Honest scope — what this is not.** Not a sales forecast. Not a calibrated Huff model or a
cannibalisation study. Not a leakage/surplus calculator (see Scope). Not a foot-traffic product.
Not a system of record for parcels or leases (CoStar/LightBox/the county assessor are). Not a legal
determination on parking, zoning or entitlement. Not a substitute for a site visit.

## 2. UI design spec (front-loaded)

### 2.1 Layout (Template: **`open-design` — "swing-slate"**)

- **Template `open-design`** under the freestyle charter (`strata/recipes/COMPONENT-MANIFEST.md`
  §10): registry widgets and manifest config keys only, plus **two** §10.2 new widgets **each with a
  named day-1 fallback** (§2.6). Do **not** fall back to `split-dashboard`.
- **Why this silhouette:** the buyer's real question is not *which site* but *how sure are we*, so
  the navigation is a **rail of assumptions driving a ranked, decomposed slate** — and the
  protagonist is the **flip margin** between adjacent ranks. Anti-collision (Commerce & Real
  Estate): not `nearby-finder` (catchment-market owns rings), not `ops-command` (retail-network
  coverage), not `chart-board` (real-estate portfolio), not `tabbed-workbench`
  (construction-engineering), not `triage-console` (facilities-management). **No recipe in any
  sector uses decision *stability* as its navigation** — aviation navigates a vertical feet-MSL
  ladder, agriculture a from×to transition matrix, utilities a voltage rail.
- **Signature loop:** *move a weight → the score re-ranks → the slate re-sorts and the flip chips
  recompute → the map repaints rank symbology → the evidence strip and the committee packet follow.*
- **Wiring floor:** ≥3 live `connections` on first render — this design ships **17** (§2.7), and
  **eight** widgets link with **no connection at all** via `dataSource.fromWidget:"score"`.

```
┌ HEADER  ◧ SITE SELECTION · demo slate · Grocery 4451 ▾ · [Slate][Site dossier][Open·expand·close] · + Add site · ◐ · [Committee packet] ┐
├─────────────────────║──────────────────────────────║─────────────────────────────────────┤
│ ASSUMPTION RAIL 318px║ THE SLATE                    ║  MAP — half the screen, on every page      │
│                      ║ ┌KPI┬KPI┬KPI┬KPI┬KPI┐        ║                                          │
│ Trade area           ║ │win│flip│ hh│comp│stab│       ║   ◉ candidate sites, sized by rank       │
│  ● ring  ○ drive-min  ║ └───┴────┴───┴────┴────┘       ║   ◌ trade area of the selected site      │
│  ├──●─────┤ 3.0 mi   ║ #1 ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓ 80.5      ║   ▲ competitors in category (NAICS)      │
│                      ║    Elk Grove Blvd @ Bruceville║   ┃ nearest AADT station (de-duplicated) │
│ Weights        Σ 100  ║    demand│spend│expo│comp│acc  ║   ▨ HQTA ½-mile (AB 2097, advisory)      │
│  demand      ●──  30 ║  ╬╬ ROBUST  gap 21.0 pts —    ║                                          │
│  spend      ●───  20 ║     #2 needs demand ↑ 61.4    ║                        ╤ ▤ basemap  ┐      │
│  exposure    ─●─  20 ║ #2 ▓▓▓▓▓▓▓▓▓▓▓ 59.5 Laguna  ║                        │ ◈ layers   │ one  │
│  competition ─●─  20 ║  ╬╬ FRAGILE gap 7.9 — traffic║                        │ + / −     ┘ stack│
│  access     ●───  10 ║     ↑ 8.0 flips it            ║                        ┴                 │
│  ↺ reset  ⤓ preset ▾  ║ #3 ▓▓▓▓▓▓▓▓▓ 51.6 · #4 · #5 ║                                          │
├─────────────────────╩──────────────────────────────╩─────────────────────────────────────┤
│════════════════════ drag to open the strip ═══════════════════════════════ (y gutter) ══│
│ EVIDENCE                 (half)            ║ PROVENANCE                     (half)      │
│ 42,689 households · 34 tracts, apportioned ║ Demand      enriched tracts — _CY vintage  │
│ $61,204 MEDDI · 19 competitors NAICS 4451   ║ Exposure    AADT — String-typed, deduped   │
│ 135,000 AADT Rte 99 · 15 stations (of 30)   ║ Competitors SafeGraph — visits EXCLUDED    │
│ Elk Grove (incorporated) · outside any HQTA ║ Access      HQTA — advisory only           │
└──────────────────────────────────────────╩─────────────────────────────────────┘
  ║ = draggable gutter (6 in all)   ·   strip defaults to 152 px   ·   dbl-click a gutter to reset
  Figures are the LIVE drive output (drive-slate.mjs, 2026-07-28) — not illustrative placeholders.
```

*Phone (`responsive.small`):* the **slate is the page** — rows stack full width, each score bar still
decomposed, flip chips inline. The assumption rail becomes a `panel` `dock:"bottom"` sheet behind an
"Assumptions" button; map and evidence collapse into an `accordion`. The packet button stays pinned.
Every side-by-side `row`/`splitter` carries a `responsive.small` collapse to `column`.

**Pages** (behind a `page-nav` in the header — but **page 1 alone answers the purpose sentence**):

| Page | Type | What it is |
|---|---|---|
| `slate` | fixed | **The swing slate** — the product |
| `site` | fixed | **Site dossier** — one candidate: income distribution, competitor table, evidence with vintages, the AB 2011/SB 6 context note |
| `estate` | fixed | **Open · expand · close** — the same score over *your own* stores; a `views` node (`nav:"tabs"`, `mapState` per tab) for the three verbs |

`splash` (`once:true`) carries the one-sentence teach and the scope disclaimer. Page `header` holds
title, slate picker, category picker, packet button, `theme-switch`, `page-nav`; `footer` carries the
evidence strip, the provenance line and "screening and ranking — not a sales forecast".

**Map-anchored controls (bottom-right stack).** `basemap` (globe glyph → keyless gallery), `layers`
(show / hide / **remove** per operational layer), then `+/−`. ⚠ MapLibre **prepends** controls in a
bottom corner, so add navigation *first* for the two buttons to sit *above* it.

### 2.2 Theme

```jsonc
{ "mode": "dark",                       // inverted after building — see the note below
  "colors": { "primary":   "#0f766e",   // deep teal — the app's one saturated ink
              "secondary": "#7c3aed",   // the selected criterion segment
              "success":   "#15803d",   // ROBUST  — the rank survives a big assumption move
              "warning":   "#b45309",   // FRAGILE — a small move reverses it
              "danger":    "#b91c1c",   // a gate fails, or an input is missing
              "info":      "#0369a1", "light": "#f7f8f7", "dark": "#111c1a" },
  "fonts": { "scale": "default" },
  "variables": { "--strata-radius-md": "6px",
                 "--strata-elevation-1": "0 1px 3px rgb(0 0 0 / .10)" },
  "overrides": { "kpi": { "--strata-mono": "ui-monospace, 'SF Mono', monospace" } } }
```

**The mode inverted when the design was built.** The spec called for light — *"a deal sheet, not a
control room"*, reasoning that the output is printed into a committee packet and read across a
boardroom table. On screen that reasoning turned out to be about the *artifact*, not the *tool*: the
slate is a dense stack of saturated score segments, and on paper-white the five criterion colours
fight the background instead of reading as one bar. **Dark won** — the segments become the only
bright objects on screen and the decomposition reads at a glance — so the app now **launches dark**,
and **light became the print/counter mode** reached through `theme-switch`, which is exactly where
the original reasoning does apply (the packet, and a public-facing counter). The semantic roles carry
the thesis in both modes: **green = the ranking is robust, amber = fragile, red = a gate failed or a
number is missing.** KPI numerals are tabular. *(The same inversion happened in
`environment_agriculture-land-use`, for the same reason — a dense protagonist wants a dark ground. Two
recipes is a pattern worth recalling before specifying light for a data-dense console again.)*

**Basemap gallery — keyless, OSM-derived only, user-switchable:** **CARTO Dark Matter** (default,
paired to the dark mode) · **CARTO Positron** (the quietest ground for the light/print mode) · CARTO
Voyager · OpenStreetMap · OpenTopoMap. `theme-switch` pairs the basemap to the mode; the gallery can
then override that pairing. ⚠ **MapLibre paint cannot read CSS variables**, so ink-coloured layers
(candidate halos, trade-area outline) must be **re-painted on every light↔dark basemap flip**.
Polygon fills ~40/255 so trade area + HQTA + tracts stack readably.

**Chrome (built 2026-07-28).** The header keeps the design guide's app-shell proportions — it is
real chrome, not a condensed toolbar. The three map controls (**zoom · layers · basemap**) are **one
uniform family**: all three are custom `.mapbtn` groups sharing identical chrome and a single
inline-SVG glyph treatment (24-box, `fill:none`, `stroke:currentColor` at 1.8), stacked flush into
one column with only the outer corners rounded. ⚠ **MapLibre's native `NavigationControl` is
deliberately not used** — its +/− buttons are CSS `background-image` sprites that cannot be made to
match an inline-SVG glyph at any size or in any theme.

**The map is half the screen on every page.** One shared `.mapcol` rule (`flex:0 1 50%`) governs the
slate, the site dossier and the estate page alike, so the map never changes size as you move between
them. It is exactly 50% down to ~1,280 px and then *yields* rather than overflowing, because the
slate carries a hard **300 px floor** — the protagonist can never be squeezed out by the map.

**The evidence strip is a resizable panel with equal halves.** It defaults to **152 px** — room for
several lines of facts and the full provenance list — with each column scrolling inside its own box
and its label pinned sticky, so a long note can never push the slate off-screen. **Evidence and
Provenance each take half** the strip. Provenance is rendered as a **label/value grid** rather than a
run-on paragraph — source names align down a narrow first column, caveats flow in the second — and
the same markup is reused verbatim in the committee packet at full size. *(An earlier pass gave
Provenance 58% of a 76 px strip; doubling the height is what actually bought its readability, so the
width went back to an even split.)*

**Six drag gutters** ship: four on the x axis (rail | slate | map, the dossier split, the estate
split), one on the x axis inside the strip (evidence | provenance), and one on the y axis above the
strip. All use **pointer capture** so a drag survives crossing the MapLibre canvas, a
`requestAnimationFrame`-throttled `map.resize()`, and **double-click to restore** the default
proportion. The clamp is generic rather than hard-coded: a drag may never push a sibling below the
`min-width` it declares in CSS, so every pane owns its own floor. On phone every gutter is hidden and
the strip reverts to auto height (capped at 40 vh) with the border the gutter was providing.

**Language: EN + AR/RTL.** Unlike the aviation recipe, nothing here is a regulatory record quoting
statute numbers — the score, the criterion labels and the evidence strip are ordinary business
language, and retail expansion is exactly the GCC use case. Ship `@strata/i18n` + `lang-switch` with
EN + AR, and use **logical CSS properties** throughout so the assumption rail and the slate mirror as
a unit. The AB 2097 gate is CA-only and labelled as such; an AR deployment simply carries no local
gate chip.

### 2.3 KPI cards

Five, live via `stat` + a shared source — **no `connections` needed**:

| Card | Value | Threshold colouring |
|---|---|---|
| **Leading site** (`kpi`) | the name at rank #1 | neutral |
| **Flip margin** (`kpi`) | pts between #1 and #2, `deltaLabel` = the criterion that would flip it | `warning` under 8 pts, else `success` |
| **Households in trade area** (`kpi`, `stat:{field:"TOTHH_CY",op:"sum"}`) | apportioned sum | neutral |
| **Competitors in category** (`kpi`, `stat:{field:"OBJECTID",op:"count"}`) | live count, recomputed on `extentChange` | neutral |
| **Ranking stability** (`gauge`, thresholds 25/60) | 0–100: how large an assumption move the whole order survives | `invertColors` off |

Plus a **local-gate chip** (`kpi`, `status`): *"parking relief likely — AB 2097, confirm with the
jurisdiction"* / *"outside a mapped HQTA"* / *"no HQTA data for this area"* — **never a green tick**.

### 2.4 Charts & table

- **`slatebars` (`stacked-bar`, `horizontal:true`)** — the score decomposition, one series per
  criterion. This is also the day-1 fallback view for `decision-slate` (§2.6).
- **`incomechart` (`chart`, bar)** — the trade area's **nine-bucket household income distribution**
  on real columns `HINC0_CY · HINC15_CY · HINC25_CY · HINC35_CY · HINC50_CY · HINC75_CY ·
  HINC100_CY · HINC150_CY · HINC200_CY`. This is what tells a premium banner from a value banner, and
  it is the reason to prefer the enriched layer over a bare median. Brush → `rangeSelect` cross-filters.
- **`comptable` (`table`, server-paged `AttributeTablePanel`)** — the competitive set on real
  columns: `location_name · brands · top_category · sub_category · naics_code · street_address ·
  city`. Row → `zoomTo` + `flash`; CSV/GeoJSON export. **The `total_visits` / `daily_average` /
  `Visits*` columns are deliberately excluded** (§3 gotcha).
- **`estatetable`** — the same machinery over an uploaded estate, sorted ascending so closure
  candidates surface first.

### 2.5 Capabilities to use (Phases 0–7)

- **`/analyze`** — `buffer` (ring bands), `pointsWithin` (competitors and AADT stations in band),
  **`overlay` + `aggregate`** (the load-bearing one: **areal apportionment** of tracts into the trade
  area — never count a straddling tract whole), `weighted-overlay` (the score). **No
  `hexbin`/`hotspot`** — clustering 82,905 POI just reproduces the population map.
- **WIF `connections`** — §2.7; plus `dataSource.fromWidget:"score"` linking **eight** widgets with
  none.
- **Panels/widgets** — `layer-panel` and `basemap` as **map-anchored controls** (§2.1), plus
  `legend`, `feature-info`, `data-actions`, `filter`, `query`, `analysis`, `near-me`, `add-data`
  (catalog drawer over the 2,339-service library), `draw`/`measure` (both revert to `identify`),
  `bookmarks`, `share`, `status-bar`, `weighted-overlay`.
- **Plugins** — `@strata/plugin-search` (address → candidate site, Nominatim),
  `@strata/plugin-routing` (**drive-time** trade-area bands, OSRM/Valhalla — the alternative to
  rings, and the assumption the Huff literature says matters most), `@strata/plugin-statusbar`.
  **Not `plugin-timeslider`:** each layer has one vintage; a slider would animate nothing real.
- **Composed export** — the **committee packet** (`/export report`: the slate + decomposition + flip
  margins + evidence with source and vintage per number + map + a dissent note), an **atlas** (one
  page per candidate), `/export image`, and a `share` deep-link with `setUrlParam` on **weights** and
  **category** so a packet is reproducible from its URL.
- **Modernization (§8 patterns)** — structured `theme`, app-shell (`header`/`footer`/`splash`),
  `splitter`/`panel`/`window`/`views`+`mapState`, **`FileDataSource`** (the candidate-site and estate
  CSV uploads — this is how the app gets its subject), `RestDataSource` (the catalog JSON; the Census
  ACS fallback), `kpi`/`gauge` `stat` linking, `animate` (`fly` + `stagger` on the slate rows, so a
  re-rank *reads* as a re-order rather than a repaint; never on the map container).
- **Arcade** — rank-band symbology on the candidate layer; competitor classing on the `naics_code`
  prefix; the **`CAST(AHEAD_AADT AS INTEGER)`** normalisation; a "vintage unknown" label expression.
- **`@strata/i18n` / `lang-switch` — used** (EN + AR/RTL). See §2.2.
- **Writes 🔶** — the committee decision (`updateRecord`: chosen / deferred / rejected + note) only
  behind `assertEsriBackend`. **The whole screening, scoring and packet path is read-only and works
  on Strata Serve.**

### 2.6 The two new widgets (§10.2/§10.3) and their day-1 fallbacks

Deliberately only two — the assumption rail is the **shipped `weighted-overlay` widget**, not a new one.

| Registry key | Purpose | Emits | Day-1 fallback (ships without any core change) |
|---|---|---|---|
| **`decision-slate`** | ranked candidates, each row's score **decomposed into weighted criterion contributions**, with the flip margin between adjacent ranks | `rowSelect`, `categorySelect`, `recordsChange` | a **`stacked-bar`** (`horizontal:true`, one series per criterion) inside a `list` ordered by score, **plus** a `table` on the same source sorted by `total` (row → `zoomTo`+`flash`), **plus** a `kpi` with `deltaLabel` carrying the #1↔#2 margin. Loses the inline flip chips and click-a-segment; **keeps the ranking, the decomposition and the margin** |
| **`flip-meter`** | owns scoring **and sensitivity**: normalise → weight → solve for the smallest single-weight change that reverses each adjacent pair; publishes the slate as an output | `recordsChange`, `countChange` | the shipped **`weighted-overlay`** computes the score via `onApply` and `analysis` publishes the per-site values; sensitivity degrades to a **preset sweep** — three named weightings (Convenience / Destination / Value) as three table columns, so the committee still sees *"site B wins under the Destination weighting"*. Loses the continuous solve, not the insight |

`flip-meter` props worth fixing now: `normalize: "minmax"|"zscore"` and — critically —
**`missingPolicy: "flag"`** (default). A site with no AADT station within range must be **flagged,
never silently scored zero**; scoring a missing input as zero is how a screening tool quietly kills a
good site.

Both obey the §10.2 contract: app-local `registry` override, `--strata-*` tokens only, logical CSS
properties, the map driven **through the store**, data via `dataSource`, cross-widget behaviour
declared in `connections` — never hard-wired to a sibling.

### 2.7 `connections` (17 — every wire uses a shipped emitter)

`buttonClick`/`timer`/`mapClick`/`sketchComplete` **emitters are Phase-2 pending**, so nothing
load-bearing depends on them; the two new widgets emit only shipped trigger types.

| from | trigger | to | action | options |
|---|---|---|---|---|
| `weights` | `filterChange` | `map` | `filter` | `{ "layerId": "candidates" }` |
| `weights` | `filterChange` | — | `setUrlParam` | `{ "param": "w" }` |
| `catpick` | `filterChange` | `map` | `filter` | `{ "layerId": "competitors" }` |
| `catpick` | `filterChange` | `comptable` | `filter` | — |
| `catpick` | `filterChange` | — | `setUrlParam` | `{ "param": "naics" }` |
| `slate` | `rowSelect` | `map` | `zoomTo` | `{ "layerId": "candidates" }` |
| `slate` | `rowSelect` | `map` | `flash` | `{ "layerId": "candidates" }` |
| `slate` | `rowSelect` | `evidence` | `viewInTable` | — |
| `slate` | `categorySelect` | `map` | `filter` | `{ "layerId": "tracts" }` |
| `slate` | `categorySelect` | `slatebars` | `filter` | — |
| `score` | `recordsChange` | `map` | `filter` | `{ "layerId": "candidates" }` |
| `score` | `recordsChange` | `stability` | `showStatistics` | — |
| `map` | `featureSelect` | `slate` | `viewInTable` | — |
| `map` | `featureSelect` | `evidence` | `viewInTable` | — |
| `map` | `extentChange` | `kpi-comp` | `showStatistics` | — |
| `comptable` | `rowSelect` | `sitemap` | `zoomTo` | `{ "layerId": "competitors" }` |
| `incomechart` | `rangeSelect` | `comptable` | `filter` | — |

**Zero-connection links** (`dataSource.fromWidget` / `sourceId`): `kpi-winner`, `kpi-flip`,
`stability`, `slate`, `slatebars`, `evidence` and `estatetable` all bind to `score`; `kpi-hh` binds
to the `tracts` source and `kpi-comp` to `competitors`.

## 3. Data sources

All EPSG:4326 (reproject on ingest); `OBJECTID` is the OID throughout. California is the researched
column; Maryland is included where a **verified** equivalent exists (site-selection layers are far
thinner there and one key candidate is unreachable — noted honestly rather than filled).

| Role | California | Maryland | National / federal | CORS · licence · gotcha |
|---|---|---|---|---|
| **Competitive set (POI)** | **Cal OES `Retail/FeatureServer/0`** — **82,905** SafeGraph POI; `naics_code`, `top_category` (16 groups), `sub_category`, `brands` (25,076), `street_address` | Baltimore `PABC/Retail_Business_District` — **HTTP 000, unreachable 2026-07-28** | SafeGraph/Advan, Foursquare, Overture (commercial/licensed) | CORS `*`. ⚠ **Real layer name is `CA_051220_WeeklyPatterns`; the 21 visit columns run `Visits04_19_2020`..`Visits5_9_2020` — the COVID lockdown trough.** Use geometry + NAICS + brand; **quarantine every visits column.** Column naming is inconsistently zero-padded — parse defensively. Grocery (`naics_code LIKE '4451%'`) = **13,363** |
| **Anchor cross-check** | Cal OES `Big_Box_Grocery_Stores` (**1,014**) | — | — | Four attributes only — no NAICS, no GLA, no banner, no open date; **the city column is `City_1`**. `maxRecordCount` **1000** — page it or lose 14 rows |
| **Household demand** | **Cal OES `CA_CensusTracts_2023`** — **9,107** tracts, **164 Esri-enriched fields**: `TOTHH_CY`, `MEDHINC_CY`, `MEDDI_CY`, `HINC0_CY`..`HINC200_CY`, `OWNER_CY`, `VACANT_CY`, `MEDVAL_CY`, **`TOTHH_FY`/`MEDHINC_FY`/`POPGRWCYFY`** | `mdgeodata` `Demographics/MD_AmericanCommunitySurvey/0` — ACS tracts (**1,394**): `MEDHHINC`, `Tothhs`, `PCT_BA`, `PCTUMPLD`, `POV_POP`, `geoid` | Census **TIGERweb** `Tracts_Blocks/MapServer` + **ACS 5-year API** | CORS `*` on CA. ⚠ **`_CY`/`_FY` are Esri vintages, not census years — print the vintage.** ⚠ **Licensed derivative** — check redistribution; keep the ACS fallback wired. ⚠ **Areally apportion** tracts into the trade area, never count whole. MD exposes **`CV_*` coefficient-of-variation columns** — an honesty feature: show the margin of error |
| **Demand frame (cities)** | Cal OES `Enriched_California_Incorporated_Cities___Cities` (**797** rows, `TOTPOP_CY`, `apportionmentConfidence`) | — | — | **797 rows for 483 cities — not one row per city**; de-duplicate on NAME+COUNTY. `HasData=0` means placeholder, not zero |
| **Traffic exposure** | **Caltrans `CHhighway/Traffic_AADT/FeatureServer/0`** — **13,919 rows ≈ 6,960 stations**; `BACK_AADT`, `AHEAD_AADT`, `RTE`, `PM`, `DESCRIPTION` | — (verify MD SHA AADT before binding) | FHWA HPMS | Caltrans **reflects the request Origin** (browser-safe). ⚠⚠⚠ **Three traps — see §4.** (1) every AADT field is a **String**: `orderBy` reports **99000** as the max when the truth is **335000**; (2) `where AHEAD_AADT > 100000` **errors 400** — use `CAST(... AS INTEGER)` (3,342 rows); (3) **every station is duplicated 2×** at the same RTE+PM. Geometry is **wkid 2875** — request `outSR=4326`. **No vintage field at all** |
| **Exposure vintage + numeric fallback** | **Caltrans `CHhighway/Truck_Volumes_AADT/FeatureServer/0`** (**6,865**): `VEHICLE_AADT_TOTAL` **(Integer)**, `TOT_TRK_AADT`, `TRK_PERCENT_TOT`, **`EST_YEAR`** | — | — | **The workaround.** Same server, proper numeric types — `where VEHICLE_AADT_TOTAL > 100000` works (**1,535**) — **and it carries the count year `Traffic_AADT` omits.** Join on RTE+POSTMILE. `TRK_PERCENT_TOT` doubles as corridor character: a 25% truck share says logistics corridor, not convenience retail |
| **Transit access + the AB 2097 gate** | **Caltrans `CHrailroad/CA_HQ_Transit_Areas`** (**20,558** polys; `hqta_type`, `hqta_details`) · **`CA_HQ_Transit_Stops`** (**48,662** pts; **`avg_trips_per_peak_hr`**, `stop_id`, `mpo`) | — | — | **Legally load-bearing in CA (§1.1).** ⚠ **Not a legal determination** — a modelled planning product on a GTFS vintage; AB 2097's own definition governs and the ½ mile is measured by the agency. Render *"relief likely — confirm"*, never a green tick. `stop_id` is **only unique within `agency_primary`** |
| **Jurisdiction frame** | CAL FIRE `California_Incorporated_City_Boundaries` (**483**; `CITY`, `COUNTY`) · `California_Places_2016` (**1,522**, incl. CDPs) | `mdgeodata` `Boundaries/MD_CensusStatisticalBoundaries` | Census TIGERweb places | CORS `*`. ⚠ **Unincorporated county territory is in neither city layer** — "no city" must render as *"unincorporated — county jurisdiction"*, never as a gap. Places layer is **2016 vintage** — labelling only, not jurisdiction |
| **Site inventory (parcels)** | State geoportal `Boundaries/UCD_Parcels/MapServer/0` (`OCIO_CAParcels`) · DWR `i15_Parcels_Assessor_Lightbox` (licensed) | Baltimore `Foresty/ParcelInquiry/MapServer` | Regrid / LightBox / CoStar (commercial) | ⚠ **The statewide CA parcel layer is nearly useless for siting** — one attribute (`PARNO`), **`query` not enabled** ("The requested capability is not supported"), `maxRecordCount` 100. Draw-only. **Real CA site inventory is assembled county by county.** Make the parcel source **pluggable**; never promise statewide parcel analytics |
| **Trade-area character** | Caltrans `CHboundary/Adjusted_Urban_Area` (**185**; `Population`, `UACE10`, `UACE20`) | — | Census urban areas | Carries **both** 2010 and 2020 urban-area codes — not interchangeable; the 2020 redefinition dropped the urbanised-cluster tier. Pick one vintage and label it |
| **Demand generators (anchors)** | State geoportal `Society/California_Schools` · `Society/Colleges_Universities` · Cal OES `Hospitals`, `Universities_and_Colleges_in_California` | `mdgeodata` `Demographics/*` | HIFLD | ⚠ **Do NOT use Caltrans `Major_Traffic_Generators`** — it lives in the **CHrailroad** folder and its **69** records are **freight rail and port intermodal facilities** (UP Oakland, CSX Long Beach), not retail generators. A name trap that will waste a day |
| **Incentive / corridor context** | Caltrans `CHhqcore/Economic_Opportunity` (**8,035** tracts; income+housing+transport burden) | **`mdgeodata` `BusinessEconomy/MD_IncentiveZones`** — layer **1** *Main Street Areas* (**44**), layer **4** *Enterprise Zones* (**32**), plus A&E districts, BRAC, Sustainable Communities | Federal Opportunity Zones | ⚠ MD `MD_IncentiveZones` **layer IDs start at 1 — there is no layer 0** (`Invalid Layer or Table ID: 0`). ⚠ CA `Economic_Opportunity` is a **transport-equity index, not a spending variable** — a low score may mean high transport cost, not a poor market; 8,035 of 9,107 tracts scored, so **render the ~1,000-tract gap, do not impute**. ⚠ MD `MD_LocalBusinesses` is **not** a business inventory — its four layers are Wineries (69), Distilleries, Breweries, Farmers Markets |
| **Retail supply ground truth** | **CDTFA Taxable Sales by City** (quarterly, by type of business) | — | Census CBP / Economic Census; BLS QCEW | ⚠ **data.ca.gov CKAN returned HTTP 403 to a server-side fetch on 2026-07-28** — resolve the resource URL in a browser at build time. Reported by **permit-holder city, not store location** (HQ and online sellers distort totals), small cities suppressed, CDTFA business types ≠ NAICS. **This is why §Scope does not compute leakage/surplus** |
| **The catalog itself** | `sitesel-catalog-ca.json` — 2,339 services, **321 role-tagged** (11 roles), 18 external feeds | — | — | Built by `build_sitesel_catalog.py`; bound as a `RestDataSource` behind the `add-data` drawer |

Attribution: Cal OES, CAL FIRE, Caltrans, California Dept. of Technology/GIO, SafeGraph (via Cal
OES), Esri (demographic enrichment), US Census Bureau, Maryland iMap / MDOT — each with "no warranty;
screening use only."

## 4. Verify each URL first (terminal)

Every command below was run on **2026-07-28** and its output is quoted in §3's gotcha column.

```bash
OES=https://services.arcgis.com/BLN4oKB0N1YSgvY8/arcgis/rest/services
CT=https://caltrans-gis.dot.ca.gov/arcgis/rest/services
CF=https://services1.arcgis.com/jUJYIo9tSA7EHvfZ/arcgis/rest/services

# 1. The competitive set. NOTE the service is named "Retail" but the LAYER is a SafeGraph extract.
curl -s "$OES/Retail/FeatureServer/0?f=json" | python -c "import json,sys;d=json.load(sys.stdin);print(d['name'])"
#  -> CA_051220_WeeklyPatterns          <-- not a generic retail layer
curl -s "$OES/Retail/FeatureServer/0/query?where=1=1&returnCountOnly=true&f=json"          # -> 82905
curl -s "$OES/Retail/FeatureServer/0/query?where=brands<>''&returnCountOnly=true&f=json"   # -> 25076
curl -s "$OES/Retail/FeatureServer/0/query?where=naics_code+LIKE+'4451%25'&returnCountOnly=true&f=json"  # -> 13363
curl -s "$OES/Retail/FeatureServer/0/query?where=1=1&outFields=top_category&returnDistinctValues=true&returnGeometry=false&f=json"
#  -> 16 NAICS retail groups: Grocery Stores, Clothing Stores, Gasoline Stations, Automobile Dealers, ...

# 2. THE VISIT-DATA TRAP: the visit columns are the COVID lockdown trough. Read the column NAMES.
curl -s "$OES/Retail/FeatureServer/0?f=json" | python -c "
import json,sys; d=json.load(sys.stdin)
v=[f['name'] for f in d['fields'] if f['name'].lower().startswith('visits')]
print(len(v),'columns:',v[0],'..',v[-1])"
#  -> 21 columns: Visits04_19_2020 .. Visits5_9_2020
#     => 19 Apr - 9 May 2020. total_visits/daily_average measure the pandemic, NOT the market.
#     QUARANTINE these columns. (Note the inconsistent zero-padding: Visits04_19 vs Visits5_3.)

# 3. The demand side — 164 fields of Esri enrichment, free and keyless.
curl -s "$OES/CA_CensusTracts_2023/FeatureServer/0/query?where=1=1&returnCountOnly=true&f=json"   # -> 9107
curl -s "$OES/CA_CensusTracts_2023/FeatureServer/0?f=json" | python -c "
import json,sys; d=json.load(sys.stdin); f=[x['name'] for x in d['fields']]
print(len(f),'fields'); print([n for n in f if n.startswith('HINC')])"
#  -> 164 fields; ['HINC0_CY','HINC15_CY','HINC25_CY','HINC35_CY','HINC50_CY','HINC75_CY',
#                  'HINC100_CY','HINC150_CY','HINC200_CY']   <-- the 9-bucket income distribution
#     Also present: TOTHH_CY, MEDHINC_CY, MEDDI_CY, MEDVAL_CY, and the _FY 5-year PROJECTIONS.

# 4. TRAP 1 — every AADT field is a STRING. A naive "rank by traffic" is lexicographic nonsense.
curl -s "$CT/CHhighway/Traffic_AADT/FeatureServer/0?f=json" | python -c "
import json,sys; d=json.load(sys.stdin)
print([(f['name'],f['type']) for f in d['fields'] if 'AADT' in f['name']])"
#  -> [('BACK_AADT','esriFieldTypeString'), ('AHEAD_AADT','esriFieldTypeString')]
curl -s "$CT/CHhighway/Traffic_AADT/FeatureServer/0/query?where=AHEAD_AADT+IS+NOT+NULL&outFields=CNTY,RTE,DESCRIPTION,AHEAD_AADT&orderByFields=AHEAD_AADT+DESC&resultRecordCount=3&returnGeometry=false&f=json"
#  -> "99000" reported as the statewide MAXIMUM (SJ Rte 5, Benjamin Holt Drive).
#     The TRUE maximum is 335000 on I-805 at SAN DIEGO, HOME AVENUE. '99000' > '335000' as strings.

# 5. TRAP 1b — a numeric predicate does not merely mis-sort, it ERRORS.
curl -s "$CT/CHhighway/Traffic_AADT/FeatureServer/0/query?where=AHEAD_AADT+%3E+100000&returnCountOnly=true&f=json"
#  -> {"error":{"code":400,...,"details":["Query with count request failed."]}}
curl -s "$CT/CHhighway/Traffic_AADT/FeatureServer/0/query?where=CAST%28AHEAD_AADT+AS+INTEGER%29+%3E+100000&returnCountOnly=true&f=json"
#  -> {"count":3342}     <-- CAST is mandatory, server-side AND client-side

# 6. TRAP 2 — every count station is duplicated exactly 2x. Any "stations near me" KPI doubles.
curl -s "$CT/CHhighway/Traffic_AADT/FeatureServer/0/query?where=1=1&returnCountOnly=true&f=json"   # -> 13919
curl -s "$CT/CHhighway/Traffic_AADT/FeatureServer/0/query?where=DESCRIPTION='SAN+DIEGO,+HOME+AVENUE'&outFields=OBJECTID,RTE,PM,AHEAD_AADT&returnGeometry=false&f=json"
#  -> OBJECTID 2099 and 2100 are BYTE-IDENTICAL (Rte 805, PM 13.95, AHEAD_AADT 335000)
#     Group-by DESCRIPTION histogram: 1748 of 2000 have exactly 2 rows.
#     => 13,919 rows are ~6,960 real stations. De-duplicate on (RTE, PM).
#     Also: geometry SR is wkid 2875 (NAD83 CA State Plane V, ft) -> always request outSR=4326.

# 7. THE WORKAROUND — the sibling truck layer is properly typed AND carries the missing vintage.
curl -s "$CT/CHhighway/Truck_Volumes_AADT/FeatureServer/0?f=json" | python -c "
import json,sys; d=json.load(sys.stdin)
print([(f['name'],f['type']) for f in d['fields'] if f['name'] in ('VEHICLE_AADT_TOTAL','EST_YEAR')])"
#  -> [('VEHICLE_AADT_TOTAL','esriFieldTypeInteger'), ('EST_YEAR','esriFieldTypeInteger')]
curl -s "$CT/CHhighway/Truck_Volumes_AADT/FeatureServer/0/query?where=VEHICLE_AADT_TOTAL+%3E+100000&returnCountOnly=true&f=json"
#  -> {"count":1535}    numeric WHERE works normally. 6,865 stations total.

# 8. The AB 2097 gate — modelled HQTA geometry, with the frequency reason on the stops layer.
curl -s "$CT/CHrailroad/CA_HQ_Transit_Areas/FeatureServer/0/query?where=1=1&returnCountOnly=true&f=json"  # -> 20558
curl -s "$CT/CHrailroad/CA_HQ_Transit_Stops/FeatureServer/0/query?where=1=1&returnCountOnly=true&f=json"  # -> 48662
#  -> stops carry avg_trips_per_peak_hr + hqta_type (major_stop_rail|_ferry|_bus|hq_corridor_bus)

# 9. The jurisdiction frame.
curl -s "$CF/California_Incorporated_City_Boundaries/FeatureServer/0/query?where=1=1&returnCountOnly=true&f=json"  # -> 483

# 10. Anchor cross-check. NOTE the city column is City_1, and maxRecordCount is 1000 (not 2000).
curl -s "$OES/Big_Box_Grocery_Stores/FeatureServer/0/query?where=1=1&returnCountOnly=true&f=json"  # -> 1014

# 11. THE PARCEL DISAPPOINTMENT — statewide parcels exist and cannot be queried.
curl -s "https://services.gis.ca.gov/arcgis/rest/services/Boundaries/UCD_Parcels/MapServer/0?f=json" | python -c "
import json,sys; d=json.load(sys.stdin); print([f['name'] for f in d['fields']], 'max', d['maxRecordCount'])"
#  -> ['OBJECTID','Shape','PARNO','Shape_Length','Shape_Area'] max 100      <-- PARNO is the ONLY attribute
curl -s "https://services.gis.ca.gov/arcgis/rest/services/Boundaries/UCD_Parcels/MapServer/0/query?where=1=1&returnCountOnly=true&f=json"
#  -> {"error":{"code":400,"message":"Requested operation is not supported by this service."}}

# 12. THE NAME TRAP — "Major Traffic Generators" is a FREIGHT RAIL layer (folder: CHrailroad).
curl -s "$CT/CHrailroad/Major_Traffic_Generators/FeatureServer/0/query?where=1=1&outFields=NAME,ASSOC,CITY&resultRecordCount=3&returnGeometry=false&f=json"
#  -> PACIFIC HARBOR LINE / UP-OAKLAND-CA-200 BURMA / CSX INTERMODAL-LONG BEACH   (69 rows total)

# 13. CORS — everything the browser app touches.
for U in "$CT/CHhighway/Traffic_AADT/FeatureServer/0" "$OES/CA_CensusTracts_2023/FeatureServer/0" \
         "$OES/Retail/FeatureServer/0" "$CF/California_Incorporated_City_Boundaries/FeatureServer/0"; do
  curl -sD- -H "Origin: http://localhost:8044" "$U/query?where=1=1&returnCountOnly=true&f=json" | grep -i access-control-allow-origin
done
#  -> Caltrans REFLECTS the Origin (http://localhost:8044); the hosted AGOL orgs return `*`. Both fine.

# 14. Maryland (verified subset only — MD site-selection holdings are much thinner).
MD=https://mdgeodata.md.gov/imap/rest/services
curl -s "$MD/Demographics/MD_AmericanCommunitySurvey/FeatureServer/0/query?where=1=1&returnCountOnly=true&f=json"  # -> 1394
curl -s "$MD/BusinessEconomy/MD_IncentiveZones/FeatureServer/1/query?where=1=1&returnCountOnly=true&f=json"        # -> 44  (Main Street Areas)
curl -s "$MD/BusinessEconomy/MD_IncentiveZones/FeatureServer/0/query?where=1=1&returnCountOnly=true&f=json"
#  -> {"error":...,"details":["Invalid Layer or Table ID: 0."]}   <-- MD_IncentiveZones layer IDs START AT 1
curl -s "$MD/BusinessEconomy/MD_LocalBusinesses/FeatureServer?f=json" | python -c "
import json,sys; print([(l['id'],l['name']) for l in json.load(sys.stdin)['layers']])"
#  -> [(0,'Wineries'),(1,'Distilleries'),(2,'Breweries'),(3,'Farmers Markets')]   NOT a business inventory
```

**The scoring arithmetic, from these fields only** (print it in the packet beside every score):

```
# per candidate site s, per criterion c
raw[s][c]   = demand   : SUM over tracts t intersecting band(s) of TOTHH_CY[t] * areaFraction(t, band(s))
              spend    : area-weighted mean of MEDDI_CY over the same tracts
              exposure : MAX( CAST(AHEAD_AADT AS INT), CAST(BACK_AADT AS INT) ) at the nearest
                         DE-DUPLICATED station (key: RTE+PM) within maxStationDistance
              compete  : -1 * COUNT(competitors in band(s) WHERE naics_code LIKE '<prefix>%')
              access   : 1 if band(s) centroid within an HQTA polygon else 0   # advisory, see §1.1

norm[s][c]  = (raw[s][c] - min_s raw[.][c]) / (max_s raw[.][c] - min_s raw[.][c])      # minmax
score[s]    = 100 * SUM_c ( w[c]/100 * norm[s][c] )
parts[s][c] = 100 * ( w[c]/100 * norm[s][c] )        # the decomposition the slate draws

# the flip margin between adjacent ranks a (higher) and b (lower):
#   solve for the SMALLEST single-criterion weight delta d, renormalising the rest to sum 100,
#   such that score_b >= score_a. Report (d, criterion). If no single change flips it -> ROBUST.
# missingPolicy = "flag": a criterion with no input is EXCLUDED from that site's weight base and
#   the site is flagged. NEVER score a missing input as zero.
```

## Guided wizard — **the prompts that assign the app's defaults**

Claude runs this like an ArcGIS Instant-App wizard: ask each group, **apply the default so "accept
all" builds a complete app**, confirm a one-line summary, then run §5. Launch with
`/recipe real-estate_site-selection`. Every answer *sets an application default* baked into
`layers.json` / the `AppLayout`. Phrasing per `strata/docs/reference/human-language.md`.

| # | Wizard question | Options → **default** | The default it assigns |
|---|---|---|---|
| 1 · App | Title & as-of date? | free text → **"Site Selection"**, today | header title + `strata:notes.asOf` + footer vintage line |
| 2 · Candidates | Where do the candidate sites come from? | **upload a CSV (name, lat, lon)** · drop pins on the map · search addresses · click parcels | the `FileDataSource` binding and the `candidates` layer. **The app has no subject until this is answered** |
| 3 · Category | Which retail category? | any of the 16 SafeGraph `top_category` groups → **Grocery Stores (NAICS 4451, 13,363 CA POI)** | the `catpick` default, the competitor filter, and the packet's category line |
| 4 · Trade area | How is the trade area defined? | **12-minute drive time** (`plugin-routing`) · ring in miles · sketch a polygon | the `band` widget config — **and this is the app's single most consequential assumption; the wizard says so** |
| 5 · Criteria | Which criteria, and at what weights? `[multi]` | demand · spending power · exposure · competition · access → **all five at 30/20/20/20/10** | the `weights` criteria array and the score's weight base |
| 6 · Demand source | Licensed enrichment or open ACS? | **Cal OES Esri-enriched tracts (164 fields)** · Census ACS + TIGERweb (redistributable) | the `tracts` binding. Choose ACS if the numbers will be shipped to a client |
| 7 · Exposure | How is traffic handled where no station is near? | **flag the site, exclude the criterion** · score it 0 · require a manual entry | `flip-meter.missingPolicy`. The default refuses to quietly kill a good site |
| 8 · Local gate | Include the California AB 2097 parking chip? | **yes — HQTA if mapped, "confirm with jurisdiction" always** · no | the gate chip + the `hqta` layers; never a green tick |
| 9 · Estate | Include the open · expand · close page? | **yes — upload your existing stores** · skip | the `estate` page and its `views` tabs |
| 10 · Output | Packet, theme & language | **committee-packet PDF + batch atlas**; **light (default)** · dark; **EN** · EN+AR | wires `/export report` + atlas; `ThemeSpec.mode`; `lang-switch` locales |

**Then:** Claude echoes *"Site Selection · 5 uploaded candidates · Grocery 4451 · 12-min drive time ·
5 criteria at 30/20/20/20/10 · enriched tracts · missing-exposure flagged · AB 2097 chip on · estate
page on · packet + atlas · light · EN"* and, on confirmation, runs §5 — so the app opens **fully
configured**.

## 5. Prompt-script (run in order)

```
A. /new-app — a "Site Selection" open-design app ("swing-slate"): three pages (slate / site /
   estate), structured LIGHT theme (primary #0f766e, secondary #7c3aed, success/warning/danger per
   §2.2, kpi override for tabular numerals), app-shell header (title + slate picker + category
   picker + [Committee packet] + theme switch + page-nav), footer carrying the evidence strip and
   the "screening and ranking - not a sales forecast" line, and a splash (once:true) that teaches
   the flip margin in one sentence. EN + AR/RTL via @strata/i18n with logical CSS properties
   throughout. Install deps + run command.

B. /add-data the verified layers from §3: Cal OES CA_CensusTracts_2023 (tracts), Cal OES Retail
   (competitors - bind geometry/naics_code/brands/top_category/sub_category/street_address ONLY,
   and EXCLUDE every Visits*/total_visits/daily_average column, see §4 cmd 2), Caltrans Traffic_AADT
   (outSR=4326, de-duplicated on RTE+PM, see §4 cmd 6), Caltrans Truck_Volumes_AADT (for EST_YEAR and
   the numeric cross-check), Caltrans CA_HQ_Transit_Areas + CA_HQ_Transit_Stops, CAL FIRE
   California_Incorporated_City_Boundaries, Cal OES Big_Box_Grocery_Stores. Add the candidates layer
   from the wizard's CSV via FileDataSource. Set the initial extent to the candidate bounding box
   plus the trade-area band.

C. /symbology + /popup - genuine ESRI drawingInfo/popupInfo on verified fields only.
   Candidates: uniqueValue on the computed rank band, graduated size, halo on the selection.
   Competitors: uniqueValue on the naics_code prefix via Arcade, hollow where brands is blank.
   Tracts: classBreaks on TOTHH_CY, low-alpha fill (~40/255). HQTA: light hatch labelled "AB 2097 -
   advisory". AADT: graduated circles on CAST(AHEAD_AADT AS INTEGER) - never on the raw string.
   Popups: candidate popup shows score + decomposition + flip margin; competitor popup shows
   location_name/brands/sub_category/naics_code/street_address; tract popup shows TOTHH_CY/MEDHINC_CY/
   MEDDI_CY WITH the _CY vintage printed; HQTA popup shows hqta_type + avg_trips_per_peak_hr as the
   REASON, plus "confirm with the jurisdiction".

D. /analyze - the numbers this app exists for. buffer (or plugin-routing isochrone) for the trade-area
   band; overlay + aggregate for the AREAL APPORTIONMENT of tracts into the band (never count a
   straddling tract whole - this is the load-bearing op); pointsWithin for competitors and AADT
   stations in band; weighted-overlay for the score. These four ARE the day-1 fallback for
   `flip-meter` - build them whether or not the widget ships.

E. /panel statistics as the KPI row: Leading site, Flip margin, Households in trade area
   (stat TOTHH_CY sum), Competitors in category (stat count), Ranking stability (gauge, thresholds
   25/60), plus the AB 2097 gate chip. Bind with dataSource stat + fromWidget:"score" / sourceId so
   they update with NO connections. The gate chip must render "confirm with the jurisdiction" - never
   a green tick - and a site with no AADT station must render "flagged", never a zero score.

F. /panel chart + /panel table - slatebars (stacked-bar, horizontal, one series per criterion),
   incomechart (bar on HINC0_CY..HINC200_CY, the 9 real buckets), comptable (AttributeTablePanel on
   location_name/brands/top_category/sub_category/naics_code/street_address/city, server-paged,
   row -> zoom + flash, CSV/GeoJSON export), and estatetable sorted ASCENDING so closure candidates
   surface first.

G. WIF: author AppLayout.connections - the 17 wires in §2.7. Verify each emitter is shipped
   (table->rowSelect, chart->categorySelect/rangeSelect, filter->filterChange, map->featureSelect/
   extentChange, output->recordsChange); do NOT wire buttonClick/timer/mapClick/sketchComplete, whose
   emitters are Phase-2 pending. Register the two §2.6 widgets app-locally via
   <StrataApp registry={{...}}>; ship each one's named fallback in the same layout so the app is
   complete either way.

H. Controls + export: navigation/scale/geolocate; put the LayerPanel and BasemapPanel on the map as
   bottom-right controls ABOVE the zoom cluster (MapLibre prepends in bottom corners - add navigation
   first), with per-layer show/hide/REMOVE; Legend, Measure + Draw (both revert to identify), search
   (Nominatim) -> add a candidate site, and a status bar. Re-paint the ink-coloured layers on every
   basemap light<->dark flip (MapLibre paint cannot read CSS variables). Wire /export report as the
   committee packet (slate + decomposition + flip margins + evidence with SOURCE AND VINTAGE per
   number + map + dissent note), a batch atlas (one page per candidate), /export image, and a share
   deep-link with setUrlParam on weights (w) and category (naics) so a packet is reproducible from
   its URL.
```

## 6. Verify (benchmark: ArcGIS Business Analyst suitability analysis · Placer.ai · Buxton · SiteZeus)

| Check | Pass | Evidence |
|---|---|---|
| Silhouette is the **assumption rail + decomposed slate**; distinct at a glance from every Commerce sibling (nearby-finder · ops-command · chart-board · tabbed-workbench · triage-console) | ✅ | built in `app/`; no sibling in any sector navigates by decision stability |
| The signature loop works end-to-end: move a weight → slate re-sorts → flip chips recompute → map repaints → packet follows | ✅ | `drive-slate.mjs` §4 — a demand-dominant weighting moves #1 from Elk Grove Blvd to Elk Grove Florin |
| **Every predicted flip actually reverses its pair when applied** | ✅ | `drive-slate.mjs` §3 — **4/4**; and *half* the predicted change does **not** reverse it |
| Every `layerId` + field verified against the service (§4) — no invented field names | ✅ | `test-slate.mjs` §15 — all 8 services return their expected row counts |
| **`AHEAD_AADT` is CAST to integer everywhere**; no ranking, filter or symbology reads the raw string | ✅ | `test-slate.mjs` §6–7 — naive `orderBy` gives 99,000; `aadtNum()` recovers 335,000 |
| **AADT stations de-duplicated on (RTE, PM)** — the "stations near this site" count is not doubled | ✅ | `drive-slate.mjs` §2 — exactly 2× at all five sites (`30→15 · 32→16 · 20→10 · 26→13 · 18→9`) |
| A site with **no AADT station in range is flagged, not scored zero** | ✅ | `test-slate.mjs` §11; **a real bug was found and fixed** — `aadtNum("")` returned `0`, and 30 rows carry a blank `AHEAD_AADT` |
| **No `Visits*` / `total_visits` / `daily_average` column is bound, charted, exported or scored** | ✅ | `test-slate.mjs` §9 + §13 — the allow-list is structural, and nothing leaked on the wire |
| Tracts are **areally apportioned**, and the packet says "apportioned" | ✅ | `drive-slate.mjs` §2 — 17 of 34 tracts partial; whole-counting overstates demand **+35.1%** |
| A site in unincorporated territory renders "unincorporated — county jurisdiction", not a gap | ✅ | `drive-slate.mjs` §5 — the demo slate spans both cases |
| The app **never prints a leakage/surplus figure or a sales forecast**; the scope line is visible in the splash and the packet | ✅ | `test-slate.mjs` §16 |
| Basemap **keyless throughout**; everything EPSG:4326 | ✅ | `test-slate.mjs` §15 — all five basemaps `200 image/png` |
| Every number in the packet carries its **source and vintage**; the weighting is reproducible from the share URL | ✅ | packet renders per-site provenance; `syncUrl()` puts weights + category + band in the URL |
| **The map is half the screen on every page** (slate, dossier, estate) from one shared rule | ✅ | layout simulation — 50.0% at 1920/1600/1440; yields to 48.6% at 1280 so the slate's 300 px floor holds |
| Panes are resizable and a drag can never squeeze the protagonist out | ✅ | six gutters; the clamp reads each sibling's declared `min-width` from computed style rather than a hard-coded constant |
| **Zoom · layers · basemap read as one control family** | ✅ | all three are custom `.mapbtn` groups on one inline-SVG glyph treatment; the native `NavigationControl` (CSS sprite buttons) is not instantiated |
| Launches dark, with light as the print/counter mode; no light flash on load | ✅ | the `night` class is server-rendered on `<body>`, not applied by script; Dark Matter is both the default and the `bm()` fallback |
| The evidence strip is short, resizable, and sized to its content | ✅ | 152 px default, equal halves, each column scrolls in its own box with a sticky label; provenance renders as a label/value grid reused in the packet |
| The AB 2097 chip reads "relief likely — confirm with the jurisdiction", shows `avg_trips_per_peak_hr` as its reason, and is **never a green tick** | ⚠ partial | logic + copy verified (`test-slate.mjs` §14 returns a definite yes/no); **no demo site currently falls inside an HQTA**, so the positive branch is untested against live data |
| AADT rendered with a vintage from `EST_YEAR` where joinable, and "vintage unknown" where not | ⚠ partial | implemented and rendered; the join is opportunistic and not yet asserted |
| ≥3 `connections` fire on first render (design ships 17); eight widgets link via `fromWidget` with none | ⛏ | applies to the `<StrataApp>` build (§5), not the MapLibre demo |
| Enriched-tract licensing acknowledged, and the ACS/TIGERweb fallback actually runs when selected | ⛏ | licensing is stated in the UI provenance; **the fallback is not implemented in the demo** |
| `responsive.small` collapses every side-by-side row; the slate is legible as the phone page | ⛏ | CSS breakpoints written (860 px / 1100 px); not browser-verified — no browser automation in this environment |
| Ink-coloured layers re-paint on a light↔dark basemap flip | ⛏ | implemented (`setStyle` → `styledata` → `buildLayers` + `paintMap`); not browser-verified |
| AR/RTL mirrors the rail + slate as a unit; no physical CSS properties leak | ⚠ partial | **layout is RTL-ready — 0 inline-axis properties remain** (25 logical; the surviving `top`/`bottom` rules are block-axis and are not mirrored). **The demo ships EN only**: the string table and `lang-switch` belong to the `<StrataApp>` build |
| Committee packet exports with slate + decomposition + flip margins + evidence; batch atlas paginates | ⚠ partial | the packet renders and prints; **no batch atlas in the demo** |
| Runs on Strata **and** ArcGIS; the whole screening path is read-only (writes 🔶 behind `assertEsriBackend`) | ⛏ | the demo is read-only by construction; backend parity belongs to the §5 build |


**On-par-or-better:** matches Business Analyst's suitability workflow (weighted criteria including
household income and AADT, exactly as Esri documents it) on open, keyless data — while adding the
thing none of Esri, Placer.ai, Buxton, Kalibrate or SiteZeus ships: **the sensitivity of the ranking
to the buyer's own assumptions**, decomposed per criterion and solved per adjacent pair. It also
reaches the persona the incumbents price out — the two-person real-estate team that cannot fund a
$12k–$50k/yr subscription or a GIS department. **Honest gap:** no sales forecast, no calibrated Huff
model, no cannibalisation, no leakage/surplus, no observed foot traffic, no statewide parcel
attributes, and no consumer-segmentation product (Tapestry-class).

## 7. Harvest (gaps → strata-core)

Log as strata-core issues: a **`decision-slate`** widget (a ranked list whose rows decompose into
weighted contributions with a margin between adjacent ranks — generalises well beyond retail: vendor
scoring, grant ranking, project prioritisation, mitigation-measure selection) — **this design's
harvest candidate, and a numbered template if it earns reuse twice**; a **`flip-meter`** /
sensitivity-solver widget (the reusable "how stable is this ranking" primitive); a **`normalize`
analyze op** (minmax/zscore over a feature set, the missing prerequisite for every weighted overlay);
an **areal-apportionment analyze op** (`apportion(polygons, band, field)` — currently assembled from
`overlay`+`aggregate` and re-invented per recipe); a **typed-field coercion helper** for the
depressingly common String-typed-numeric ESRI column (the aviation recipe needed `TRIM()` for the
same class of problem — these two want one shared "field hygiene" layer); and a **vintage/provenance
popup element** (value + source + as-of, rendered consistently). Bigger asks: a **gravity/Huff
analyze op** (blocked on an attractiveness input, not on code), and **spend-potential variables** —
the licensed layer we most conspicuously lack.

## 8. Sources

**Market & competitive (fetched 2026-07-28)**
- Esri — [Retail site suitability & selection](https://www.esri.com/en-us/industries/retail/strategies/site-suitability) ·
  [ArcGIS Business Analyst overview](https://www.esri.com/en-us/arcgis/products/arcgis-business-analyst/overview) ·
  [What's New in Business Analyst Web App, June 2026](https://www.esri.com/arcgis-blog/products/bus-analyst/announcements/whats-new-in-arcgis-business-analyst-web-app-june-2026) ·
  [Perform a suitability analysis (BA web help)](https://doc.arcgis.com/en/business-analyst/web/suitability-analysis.htm) ·
  [How suitability analysis works (Pro)](https://doc.esri.com/en/arcgis-pro/latest/tool-reference/business-analyst/understanding-suitability-analysis.html) ·
  [Discover Retail Opportunities with Retail MarketPlace Data (ArcWatch)](https://www.esri.com/news/arcwatch/0809/retail-marketplace-data.html) ·
  [Retail MarketPlace Profile — sample report](https://www.scacog.org/files/files/Esri%20Business%20Analyst%20Sample%20Reports/Retail%20MarketPlace%20Profile.pdf)
- Non-Esri — [Retail Analytics Platform Comparison 2026 (GrowthFactor)](https://www.growthfactor.ai/resources/blog/retail-analytics-platform-complete-guide) ·
  [Retail site selection software: compare top AI platforms (GrowthFactor)](https://www.growthfactor.ai/resources/blog/retail-site-selection-software) ·
  [7 Placer.ai alternatives (GrowthFactor)](https://www.growthfactor.ai/resources/blog/placer-ai-alternatives) ·
  [Retail site selection software — what to look for (PassBy)](https://passby.com/blog/retail-site-selection-software/) ·
  [Best site selection software for retail expansion (Geod)](https://www.geod.app/resources/best-site-selection-software-retail-expansion) ·
  [Best retail site selection software (MapZot.AI)](https://www.mapzot.ai/resources/blog/retail-site-selection-software/)
- Methodology — [Huff model: probabilistic retail trade areas (Mapular)](https://mapular.com/glossary/huff-model) ·
  [Huff gravity model (GIS Geography)](https://gisgeography.com/huff-gravity-model/) ·
  [Huff gravity model & estimating potential sales (Lumen)](https://courses.lumenlearning.com/wm-retailmanagement/chapter/huff-gravity-model-and-estimating-potential-sales/) ·
  [Calibrating the dynamic Huff model for business analysis (Univ. of Wisconsin, T-GIS)](https://geography.wisc.edu/wp-content/uploads/sites/28/2022/05/2020_TGIS_DynamicHuffModel.pdf) ·
  [Validating gravity-based market share models using large-scale transactional data (arXiv)](https://arxiv.org/pdf/1902.03488) ·
  [How trade-area definition drives feasibility conclusions — five delineation methods](https://www.mmcginvest.com/post/how-trade-area-definition-drives-feasibility-conclusions-five-delineation-methods-and-when-each-dis) ·
  [Retail cannibalization analysis (Geod)](https://www.geod.app/blog/cannibalization-analysis-site-selection) ·
  [Retail leakage analyses should be treated with great caution (N. David Milder, DANTH)](https://www.ndavidmilder.com/2016/09/retail-leakage-analyses-should-be-treated-with-great-caution-by-analysts-and-end-users-2-some-serious-data-issues)
- Standards — [ICSC US Shopping Center Definition Standard](https://www.icsc.com/uploads/t07-subpage/US-Shopping-Center-Definition-Standard.pdf) ·
  [ICSC Dictionary of Shopping Center Terms, 4th ed.](https://www.icsc.com/news-and-views/newsstand/icscs-dictionary-of-shopping-center-terms-fourth-edition) ·
  [ICSC shopping center definitions — basic configurations and types](https://incorporacaoimobiliaria.com/wp-content/uploads/2009/09/scdefinitions99.pdf) ·
  [Shopping center classifications: challenges and opportunities (deLisle)](https://jrdelisle.com/research/NewSCDef_V23_WP1.pdf)

**California statute & policy**
- AB 2097 — [HCD technical assistance memo](https://www.hcd.ca.gov/sites/default/files/docs/policy-and-research/ab-2097-ta.pdf) ·
  [HCD minimum parking requirements (April 2026)](https://www.hcd.ca.gov/sites/default/files/docs/planning-and-community/min-parking-requirements-ab-2097.pdf) ·
  [California Lawyers Association analysis](https://calawyers.org/real-property-law/california-assembly-bill-2097-eliminating-parking-minimums-for-new-developments-near-major-transit-stops/) ·
  [Cox Castle — overriding local parking requirements near transit](https://www.coxcastle.com/publication-california-legislature-overrides-local-parking-requirements-for-development-projects-within-one-half-mile-of-public-transit) ·
  [LA City Planning — AB 2097](https://planning.lacity.gov/project-review/assembly-bill-2097)
- AB 2011 / SB 6 — [SB 6 bill text (leginfo)](https://leginfo.legislature.ca.gov/faces/billTextClient.xhtml?bill_id=202120220SB6) ·
  [HCD — Middle Class Housing Act (SB 6), April 2026](https://www.hcd.ca.gov/sites/default/files/docs/planning-and-community/middle-class-housing-act-sb-6.pdf) ·
  [Holland & Knight — pathways for residential development on commercially zoned land](https://www.hklaw.com/en/insights/publications/2022/09/california-legislature-creates-pathways-for-residential-development) ·
  [Venable — AB 2011 and SB 6 take effect](https://www.venable.com/insights/publications/2023/07/ab-2011-and-sb-6-take-effect-in-effort-to-boost) ·
  [Terner Center — AB 2011 and commercial zones](https://ternercenter.berkeley.edu/research-and-policy/ab-2011-commercial-zones/) ·
  [UrbanFootprint — can commercial corridors solve California's housing crisis?](https://urbanfootprint.com/blog/policy/ab2011-analysis/)
- Retail supply data — [CDTFA Open Data Portal](https://cdtfa.ca.gov/dataportal/) ·
  [CA Taxable Sales by City (data.ca.gov)](https://data.ca.gov/dataset/ca-taxable-sales-by-city) ·
  [Taxable Sales by Type of Business](https://cdtfa.ca.gov/dataportal/dataset.htm?url=TaxSalesStatewide)

**Internal** — `DESIGN-PROPOSAL.md` · `build_sitesel_catalog.py` → `sitesel-catalog-ca.json` ·
`../APP-TEMPLATE-LIBRARY.md` (assignment: `open-design` "swing-slate"; `select-export` released) ·
`../../data_sources/data_sources_ca.md` · `../../data_sources/data_sources_md.md` ·
`strata/recipes/COMPONENT-MANIFEST.md` (§8 modernization, §10 freestyle charter) ·
`strata/docs/guide/app-design.md` · `strata/docs/reference/human-language.md`

---

## Modernization (parity release)

> Applies the Experience-Builder-parity features (Phases 1–7). See `strata/recipes/MODERNIZATION.md`
> + `COMPONENT-MANIFEST.md` §8. Cross-cutting: a structured **`theme`**, app-shell
> (`header`/`footer`/`splash`), and `dataSource.{ sourceId }` linking.

- **Source linking is the architecture, not a garnish:** one `score` output feeds `slate`,
  `slatebars`, `kpi-winner`, `kpi-flip`, `stability`, `evidence` and `estatetable` through
  `dataSource.fromWidget` — **seven widgets, zero `connections`** — and `kpi-hh`/`kpi-comp` bind to
  the shared `tracts`/`competitors` sources so the `filter` → `store.setDefinition` path updates them
  for free. That is eight of the app's widgets alive with no wiring at all.
- **Layout nodes:** `splitter` (rail | slate | map, resizable — the user decides how much of the
  argument vs. how much of the map), `panel` (the assumption rail, dock-left → **dock-bottom sheet on
  phones**), **`window`** (the committee packet, `open:false`, opened by `showHide`), `views` +
  `mapState` (the `estate` page's open / expand / close tabs each fly the map and re-filter),
  `accordion` (the phone collapse), `section` `mode:"fixed"` (the map box).
- **Data-source kinds:** **`FileDataSource`** is load-bearing here in a way it is not in any sibling
  — the candidate slate and the existing estate are *both* user CSV uploads, so the app's subject
  arrives through it. `RestDataSource` for `sitesel-catalog-ca.json` and the Census ACS fallback.
  **No `StreamDataSource`** — nothing here is real-time; a trade area's demographics do not tick.
- **Motion:** `fly` + `stagger` on the slate rows so a re-rank **reads as a re-order** rather than a
  repaint — the one place motion is carrying meaning rather than polish; `fade` on the flip chips.
  Nothing on the map container.
- **Theme:** one structured `ThemeSpec` derives hover/active/focus states and the type scale, with
  the semantic roles carrying robust/fragile/failed; `theme-switch` swaps the default **light** deal
  sheet ⇄ a dark analyst mode and pairs the basemap to it; the on-map keyless gallery can then
  override that pairing. A `kpi` `overrides` block gives the numerals a tabular mono face.
- **i18n:** EN + AR/RTL via `@strata/i18n` + `lang-switch`, logical CSS properties throughout — the
  rail and slate mirror as a unit.
- **Writes 🔶:** the committee decision (chosen / deferred / rejected + note) via `updateRecord`
  behind `assertEsriBackend`; everything else is read-only and works unchanged on Strata Serve.
