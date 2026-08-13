# Recipe — Access to Care: **"Who's Outside"** (Health & Community)

A reproducible path to a **coverage-and-deficit** app on strata-core: pick a service line, pick the
standard that actually applies in this county, and read — from one curve — **how many people are
beyond it, who they are, and whether the compliance rests on an exception**. Facilities, service
lines, travel on the real road network, and the populations on the wrong side of a legal threshold,
all from data that is already public and keyless. Two supporting pages carry the rest of the brief: a
**place dossier** and a **closure page** that answers the question a California county board is
actually asking in 2026 — *what does closing this unit do to the drive?*

> **Scope (honest).** This is a **geographic screen**, not a **network-adequacy certification**. It
> measures the distance and drive time from population-weighted origins to the nearest *licensed*
> site in a chosen service line, and compares it to a stated standard. It is **not** a measure of a
> particular plan's contracted network (those rosters are not public — the app accepts one by
> upload), **not** a provider directory, **not** an appointment-availability or wait-time measure,
> and **not** a legal determination. **It does not know which hospital still delivers babies**: no
> open California layer carries service-line status, and the only mapped birthing indicator is a
> 2021 licence snapshot (§1.1, §4 cmd 2). Travel times are **car-only, free-flow, from a community
> OSM router** — a floor, not a lived journey — and every number carries "car assumed" until an ACS
> vehicle-availability variable is wired. Every number in the packet carries its source and its
> vintage. Feature writes (a county's own verification notes) need a writable ESRI backend.

Run the §5 prompt-script on a fresh strata-core project (or via `/new-app`). ESRI Web Map JSON is the
map contract; the app is read-only on Strata Serve.

---

## 0. Provenance

| | |
|---|---|
| **Source** | `https://tabaqat.net/solutions` — **Health & Community** section |
| **Name on site** | Access to Care |
| **Tagline on site** | "Facilities, services and travel-distance to care with coverage gaps" |
| **Scaffolded** | 2026-07-22 (`nearby-finder` placeholder) |
| **Researched & designed** | 2026-07-29 — see `DESIGN-PROPOSAL.md` |
| **Template** | **`open-design` ("coverage-curve")** — the `nearby-finder` assignment is released back to the Health & Community sector |
| **Catalog** | `access-catalog-ca.json` — 2,339 CA services, **456 role-tagged** across 11 roles, **26 external feeds (22 curl-verified)**, 9 keyword false-friends excluded |
| **Built & driven** | 2026-07-29 — `app/`, **195 live-data assertions green** (96 regression + 19 loop + 80 render) |

---

## 0.1 The working app in this folder

`app/` holds a **runnable demo of this design** — no build step, no API keys, everything fetched live
from the §3 services.

```bash
cd app && python3 server.py          # -> http://localhost:8057   (or double-click START-APP.command)
                                     #    on Windows use `python server.py` — `python3` there is
                                     #    the Microsoft Store alias stub and exits with a prompt
node test-access.mjs                 # 96 assertions against LIVE HCAI / HRSA / CDC / OSRM data
node drive-access.mjs                # 19 assertions: drives the signature loop end-to-end
node drive-access.mjs "Modoc County" # ...and again in a second county
node test-render.mjs                 # 80 assertions: boots the shipped script in a stubbed DOM
COUNTY="Sacramento County" node test-render.mjs   # ...and in an urban county
```

| File | What it is |
|---|---|
| `RECIPE.md` | this file — the buildable spec |
| `DESIGN-PROPOSAL.md` | the `../DESIGN-REQUEST-PROMPT.md` deliverable: business analysis, three silhouettes, ASCII skeleton, `AppLayout` sketch, 18 connections, `ThemeSpec`, bindings, capability sweep, two §10.3 new-widget blocks |
| `build_access_catalog.py` → `access-catalog-ca.json` | the catalog builder — parses `../../data_sources/data_sources_ca.md` (2,339 services), role-tags them for access to care, and appends the 26 internet-verified external feeds with their load-bearing gotchas |
| `app/index.html` | **Access to Care — "Who's Outside"**, self-contained: the draggable coverage curve with its statutory and as-granted rules, five live KPIs, the map with on-map basemap/layer controls, the who-is-outside cohort list, the evidence + provenance strip, the cohort dossier, the SB 1300 closure page, the printable hearing packet, deep-linkable state, three drag gutters, **dark by default** with a light print/counter mode, EN |
| `app/test-access.mjs` | Regression test that **extracts the shipped functions out of `index.html`** and checks them against live data — typed-field coercion, point-in-polygon with holes, the distance-**OR**-time rule, curve monotonicity, band conservation, population apportionment in both the rural and the urban regime, the origin cap — plus every trap in §4 reproduced on the wire |
| `app/drive-access.mjs` | Headless drive of the **signature loop**: loads a county the way the app does, measures every origin on the road network under both origin policies, sweeps the threshold, flips the exception, and closes the county hospital |
| `app/test-render.mjs` | Boots the **shipped inline script** in a stubbed DOM + stubbed MapLibre against live data, and asserts every panel is actually populated — the render-time class of bug a browser only reports as "something went wrong" |
| `app/net-shim.mjs` | Harness-only: falls back to `node:https` where Node's `fetch` cannot connect. The app itself uses the platform `fetch` and nothing else |
| `app/server.py`, `app/START-APP.command` | Zero-dependency static server (MapLibre cannot run from `file://`) |
| `app/access-catalog-ca.json` | Copy of the built catalog |
| `presentation/index.html` | 10-slide deck (house template, self-contained, prints to PDF one slide per page): the gap → the assumption nobody re-examines → the coverage curve → the proof → the CA data spine → six traps → the exception → honest scope → where it sits next to the incumbents |
| `presentation/linkedin-article.md` | ~1,000-word article + teaser post + a **claims note citing every figure to its source**, with an explicit do-not-claim list and the honest gaps volunteered |

**Verified 2026-07-29 — 195 assertions pass. The three findings the demo exists to make:**

| Check | Result |
|---|---|
| **The origin policy decides the answer** | ✅ Trinity County, same standard, same router, same hour: **35.1% outside** from population-anchored origins vs **100.0%** from tract internal points — **64.9 points apart**. Modoc County: **18.4% vs 63.0%** — 44.6 points apart. The default is the honest one |
| **An exception erases the deficit without moving anyone** | ✅ applying the documented California mean AAS (+31.61 mi) takes Trinity's outside count from **4,332 to 0**, with all 4,332 reclassified **covered on paper** at exactly the same drive time |
| **A closure is measurable in the units the hearing argues in** | ✅ removing Trinity Hospital moves **+6,977 people** outside and names them — Weaverville **3.3 → 50.1 min**, Lewiston 25.3 → 41.0, Douglas City 9.3 → 39.8, Junction City 13.5 → 60.4. In Modoc, closing Surprise Valley Community Hospital moves Cedarville **1.2 → 39.2 min** |
| Apportionment is conservative, in both regimes | ✅ Σ origin population = Σ tract population exactly (12,327 = 12,327); Sacramento's **342 of 364** origins resolve as "tract lies inside a settlement", so the rural rule is not misapplied to a city |
| The AREA_SQMI string trap reproduced live | ✅ naive `orderBy` returns **99.21** as the largest CA tract; `where > 100` errors HTTP 400; `CAST(… AS FLOAT)` returns **267** |
| The MSSA key-join trap reproduced live | ✅ 542 vs 366 distinct ids, **exact overlap 6** — the app joins designations spatially |
| The HPSA schema trap reproduced live | ✅ `HPSA_State_Abbreviation` is an Invalid field; `PriStNM='California'` returns **180** against mental health's **2,363** components |
| The birthing flag is a 2021 fossil | ✅ **257** flagged facilities, newest licence expiry **2021-11-15**; the obstetric service line ships behind a verify badge |
| Nothing is silently capped | ✅ a county over 200 origins measures the largest-population ones first and prints *"measured 200 of 364 origins — 70% of the county's people"* in the UI **and in the packet**, with a "measure all" button |
| Every panel actually renders | ✅ the shipped script boots in a stubbed DOM in **7 s** (Trinity) / **13 s** (Sacramento) and populates the curve, KPIs, cohort list, evidence, provenance, legend, dossier, closure page and packet. **One real bug was found and fixed here** — the legend was gated behind the map style being ready, so it would have been blank on first paint |
| Keyless throughout | ✅ five basemaps return `200 image/png`; OSRM answers `Access-Control-Allow-Origin: *`; Valhalla isochrone 200; no key or token anywhere in the file |

**What the demo deliberately does not do:** it is a MapLibre app that *renders this design*, not a
`<StrataApp>` — §5 is what produces the real `layers.json` + `AppLayout` on strata-core. It ships EN
only (the AR/RTL string table belongs to the `<StrataApp>` build), it does not draw the Valhalla
isochrone contour (the compliance number comes from the matrix; the contour is a picture the §5 build
adds), and its origin cap is a courtesy to a community router, not a limit of the method.

---

## 1. Study — how the market frames this

**The question the buyer asks:** *"For this service and the standard that actually applies here, how
many of my people are outside it — and where are they?"* Every incumbent answers a different
question: *does this plan's network pass?* The buyer we are serving is often the person on the other
side of that filing.

**The gap in one sentence.** **Access is asserted in a compliance table and experienced as a drive**,
and there is no public instrument that reconciles the two. The compliance table is a private
regulatory artifact; the drive is measurable from open data; nobody publishes the difference.

**Reference solutions (benchmark + coexist, never copy).**

- **Quest Analytics — the incumbent, and it is not Esri.** *Quest Enterprise Services (QES)* is the
  SaaS platform health plans use to measure, monitor and model network adequacy; the company reports
  **400+ health plan clients including all eight of the nation's largest**, **150B+ provider records
  processed monthly**, and being **trusted by CMS, CCIIO and 30+ state agencies**. **CMS extended its
  contract with Quest for a further five-year term (announced 27 January 2026)** to measure the
  adequacy of all Medicare Part C & D and MMP provider, facility and pharmacy networks. Its adequacy
  module computes time, distance and provider-ratio requirements across the **five CMS county
  designations** and ships ready-made federal and state templates. **This is the certification
  layer, it is excellent, and we do not compete with it.**
- **The compliance long tail** — **Verisys**, **Atlas Systems** and similar vendors publish the same
  framing for the 2026 cycle: gap-density maps, coverage maps, and the warning that
  **from 1 January 2026 state-based marketplaces must adopt quantitative time-and-distance standards
  mirroring the federally-facilitated exchanges**, extending a standardisation already reaching
  Medicaid managed care.
- **Esri** — the *Access to Care* focus area is a **capability stack, not a solution app**:
  ArcGIS Pro, **Business Analyst**, **Network Analyst** ("plan routes, calculate drive times, and
  locate facilities"), StreetMap Premium, Insights, StoryMaps, under three headings — *achieve
  network adequacy · connect people to services · universal health coverage* — with named customers
  (Rancho Cucamonga, Washington State childcare, Inland Health Plan's 1.2M Southern California
  members) and one packaged solution, *Opioid Epidemic Outreach*. Esri's own ArcNews (Winter 2025)
  documents the network-adequacy workflow as a **GeoAnalytics Engine hexagon-tessellation** job — a
  data-engineering pattern, run by an analyst, on a licensed stack.
- **The public / non-profit tier** — **UDS Mapper** (HRSA-funded, built by HealthLandscape with
  John Snow Inc. and the Robert Graham Center; its users are FQHC professionals building service-area
  maps and finding expansion areas), **HealthLandscape** (a division of the American Academy of
  Family Physicians; free but login-gated), **PolicyMap** (Reinvestment Fund), **County Health
  Rankings**, **CDC PLACES**, **March of Dimes PeriStats**. These are excellent *reference* tools:
  they show you an indicator on a geography. **None of them lets you set a threshold and read the
  population beyond it on the road network**, and none of them models an exception.
- **The methodological spine, and its documented failure mode.** The academic standard is the
  **two-step floating catchment area (2SFCA)** family — a service-to-population ratio over
  overlapping floating catchments — and its **enhanced form (E2SFCA, Luo & Qi 2009)**, which adds
  distance-decay weights inside the catchment. The literature is blunt about where it breaks:
  **uniform access inside the catchment**, **fixed thresholds producing abrupt discontinuities**
  (a resident just past the boundary is excluded from a facility they can see), **edge effects**, and
  **catchment rules that transfer badly between dense and sparse geographies**. Every one of those
  failures has the same shape as the regulatory failure: **a single unexamined threshold, quietly
  deciding everything downstream.** This app's answer is not a better catchment — it is to **make
  the threshold visible and movable** and to show the mass of population sitting next to it.

**Our edge.** AI-authored, MIT/open, runs on **Strata *or* ArcGIS**, sovereign/on-prem — and the
decisive one: **we can show the arithmetic, and we can show both sides of the filing.** The incumbent
sells a certification to the party being certified; its model must stay closed and its customer is
the plan. We sell the geographic screen to whoever needs it — the county, the health centre, the
advocate, and the plan's own analyst doing a pre-check — which is only sellable if the model is open.
Coexistence, never "replace": a plan that owns QES should push **its own contracted-provider roster**
into this app as the supply set (a `FileDataSource` upload) and use it to *see* what its certified
network looks like as a distribution.

**Standards, specifications & organizations to speak fluently.**

- **HRSA / Bureau of Health Workforce** — **HPSA** (Health Professional Shortage Area) and **MUA/P**
  (Medically Underserved Area/Population). HPSA scoring uses the **population-to-provider ratio**,
  the share at or below 100% FPL, and travel time to the nearest source of care; scores run **0–25**
  for primary care, maternal care and mental health and **0–26** for dental. MUA/P turns on the
  **Index of Medical Underservice (IMU)**: **62.0 or below qualifies**, built from provider ratio
  (max 28.7 points), % at 100% FPL (25.1) and % aged 65+ (20.2). These designations carry money —
  NHSC placements, FQHC eligibility, Medicare bonuses.
- **CMS / CCIIO** — Medicare Advantage and Part D network adequacy (the Quest contract), the
  marketplace time-and-distance standards extending to **state-based marketplaces from 1 Jan 2026**,
  and the Medicaid managed-care access rule.
- **California — DHCS** (Medi-Cal managed care time-and-distance standards, the **Annual Network
  Certification**, and the **alternative access standard (AAS)** process under **APL 23-001**) and
  **DMHC** (the **Knox-Keene Act**, **28 CCR §1300.67.2.2** timely access, the annual Timely Access
  Report).
- **HCAI** (California Department of Health Care Access and Information, formerly OSHPD) — the
  licensed facility inventory, the **Medical Service Study Area (MSSA)** geography, and California's
  own **Primary Care Shortage Area (PCSA)** designation.
- **Geography** — Census tracts / block groups / TIGER, **ZCTA**, and the fact that **the standard
  is set per county** so the county polygon is a lookup key, not decoration.
- **Method** — **2SFCA / E2SFCA**, gravity models, isochrones vs. rings, population-weighted
  centroids.

### 1.1 What makes California different (and why the demo is CA)

Three things, and each one changes the app:

- **California has its own sub-county health geography.** HCAI maintains **Medical Service Study
  Areas** — tract aggregations that are the unit every *state* shortage designation is computed on,
  with their own **Urban / Rural / Frontier** classification. **9,106 tracts → 542 MSSAs**, of which
  **268 are designated Primary Care Shortage Areas**, each carrying its arithmetic in the open:
  estimated physicians, estimated FNP/PA, the population-to-provider ratio, a provider score, the
  share under 100% FPL and a poverty score. No other state publishes its shortage arithmetic this
  transparently — and, as §4 cmd 12 shows, **it no longer joins to its own current geography.**
- **The standard is not the standard.** DHCS publishes time-and-distance standards **and grants
  exceptions to them at scale.** The National Health Law Program's analysis (31 July 2019) found DHCS
  approved **nearly 10,000 new alternative access standard requests in January 2019 alone**, raising
  the required provider distance by an average of **31.61 miles**; **paediatric providers accounted
  for nearly three-quarters of approvals**; specialty approvals concentrated in physical medicine,
  infectious diseases, nephrology, ophthalmology, haematology and **OB/GYN primary care**; and
  **low-income areas received more approvals with larger distance increases than high-income ones**.
  An app that renders a county "compliant" without showing whether that compliance rests on a granted
  exception is reporting the paperwork, not the access. Hence the design's second protagonist:
  **"as written" vs "as granted"**.
- **The supply side is actively shrinking, and the legislature has noticed.** **At least 56
  California maternity wards have closed since 2012** (the findings behind **SB 1300**, signed
  **28 September 2024**, which requires **120 days' public notice** — up from 90 — **and a public
  hearing** before a health facility eliminates inpatient psychiatric or perinatal services). Madera
  County reopened its hospital in March 2025 **without** labour and delivery, leaving it the only
  San Joaquin Valley county with no hospital birthing service; Corona Regional Medical Center
  discontinued maternity services on **30 January 2026**. Nationally, people in maternity care
  deserts travel **2.6× longer — 38 minutes vs 14.4** — to reach a birthing hospital, and **19.0% of
  California counties have no hospital or birth centre offering maternity care**.
  **This is why the `closure` page exists**, and it is the page a county board will open first.

**Honest scope — what this is not.** Not a network-adequacy certification (that is a filing against a
plan's own roster). Not a provider directory. Not an appointment-availability, wait-time or
timely-access measure. Not a 2SFCA accessibility index (it deliberately reports a measured quantity
against a stated threshold instead of a composite score). Not an ambulance response-time model —
neither LEMSA response zones nor unit counts are in open statewide data. Not a legal determination
on adequacy, designation or entitlement. Not a substitute for calling the hospital to ask whether the
unit is still open.

## 2. UI design spec (front-loaded)

### 2.1 Layout (Template: **`open-design` — "coverage-curve"**)

- **Template `open-design`** under the freestyle charter (`strata/recipes/COMPONENT-MANIFEST.md`
  §10): registry widgets and manifest config keys only, plus **two** §10.2 new widgets **each with a
  named day-1 fallback** (§2.6). Do **not** fall back to `split-dashboard`.
- **Why this silhouette:** the buyer's question is a *quantity relative to a threshold*, so the
  navigation is a **cumulative coverage curve with the statutory rule drawn on it**, and the
  protagonist is **the mass of population on the far side**. The map is demoted to an evidence pane.
  Anti-collision (Health & Community): not `chart-board` (health_health-equity-overlay), not `triage-console`
  (housing-homelessness), not `scoreboard` (humanitarian-crisis), not `ops-command` (preparedness),
  not `scroll-story` (community-nonprofit). **No recipe in any sector navigates by a threshold on a
  distribution** — commerce navigates by decision stability, agriculture by a from×to transition
  matrix, aviation by a feet-MSL ladder, utilities by a voltage rail.
- **Signature loop:** *pick a service line → the county's applicable standard drops its rule onto the
  curve → the curve is computed live on the road network from population-weighted origins → drag the
  rule (or flip to "as granted") → the deficit KPI, the map's outside shading and the cohort list all
  move together → "test a candidate site" re-runs it and reports the people brought inside → the
  hearing packet follows.*
- **Second loop (the closure):** remove a facility or a service line from supply on the `closure`
  page and read the same curve before / after / difference — the SB 1300 question, answered in the
  units the statute is argued in.
- **Wiring floor:** ≥3 live `connections` on first render — this design ships **18** (§2.7), and
  **nine** widgets link with **no connection at all** via `dataSource.fromWidget:"access"`.

The full ASCII skeleton (desktop + phone) is in `DESIGN-PROPOSAL.md` §3. In short: a
`splitter` `orientation:"v"` with three bands — **the curve + KPI row** (42%), **map | cohort list**
(44%), **evidence | provenance strip** (14%, a collapsible bottom `panel`) — three drag gutters, and
`responsive.small` collapsing every side-by-side row so that **on a phone the curve is the page**.

**Pages** (behind a `page-nav` — but **page 1 alone answers the purpose sentence**):

| Page | Type | What it is |
|---|---|---|
| `coverage` | fixed | **The curve** — the product |
| `cohort` | fixed | **One place in detail** — poverty and age profile, designation status, the measured route to the nearest open site, the alternatives, and the 95% CI on every modelled number |
| `closure` | fixed | **What a closure does** — a `views` node (`nav:"tabs"`, `mapState` per tab): *before · after · who moves* |

`splash` (`once:true`) teaches the idea in one sentence and carries the scope disclaimer. The page
`header` holds the county / service-line / standard pickers, the packet button, `theme-switch`,
`lang-switch` and `page-nav`; the `footer` carries the scope line.

**Map-anchored controls (bottom-right stack).** `basemap` (globe glyph → keyless gallery), `layers`
(show / hide / remove per operational layer), then `+/−`. ⚠ MapLibre **prepends** controls in a
bottom corner, so add navigation *first* for the two buttons to sit *above* it.

### 2.2 Theme

```jsonc
{ "mode": "dark",                       // inverted after building — see the note
  "colors": { "primary":   "#1d4ed8",   // civic blue — the standard's ink, and the curve
              "secondary": "#7c3aed",   // the granted (AAS) ghost rule
              "success":   "#15803d",   // inside the standard
              "warning":   "#b45309",   // covered only under a granted exception
              "danger":    "#b91c1c",   // outside every standard
              "info":      "#0369a1", "light": "#f8fafc", "dark": "#0f172a" },
  "fonts": { "scale": "default" },
  "variables": { "--strata-radius-md": "6px",
                 "--strata-elevation-1": "0 1px 3px rgb(0 0 0 / .08)" },
  "overrides": { "kpi": { "--strata-mono": "ui-monospace, 'SF Mono', monospace" } } }
```

**The mode inverted when the design was built — for the third recipe running.** The spec called for
light, reasoning that this protagonist is *one thin line on a large empty field* whose audience is a
public hearing and a printed board packet, unlike the dense saturated stacks that pushed
`real-estate_site-selection` and `environment_agriculture-land-use` to dark. On screen that reasoning turned
out to be about the **artefact**, not the **tool**: the working surface is a console the user drags a
rule across for minutes at a time, and a thin bright curve reads best on a dark ground — as do the
red deficit mass and the amber as-granted band. **Dark won**, so the app now **launches dark**, and
light became the **print / public-counter mode** reached from the header toggle — which is exactly
where the original reasoning does apply. **Three recipes is no longer a coincidence: specify dark
first for any data-dense console and treat light as the export mode.** The semantic triad carries the
whole argument in both modes without the legend being read: **green = inside · amber = covered on
paper · red = outside**. KPI numerals are tabular.

**The rule that follows from a dark default: a light surface always carries dark ink.** Not every
surface is ours to theme — MapLibre ships a white popup, a translucent white attribution plate and a
white scalebar regardless of the palette, and paper is white whatever is on screen. Left alone, the
dark theme's light `--fg` would put **white text on a white popup**. So the app defines a single
`--ink-on-light` token, re-themes the popup to the app's own panel, pins the attribution and scalebar
to that token, gives native `option` lists explicit colours, and ships an `@media print` block that
forces the light tokens **whichever theme is on screen** — so the hearing packet always prints as a
dark-on-white document rather than a screenshot of a dark console. The map's own ink is a second
palette (`mapInk()`), because MapLibre paint cannot read CSS variables; it is re-applied on every
theme flip through `applyBasemap` → `styledata` → `buildLayers`.

**Chrome (rebuilt 2026-07-29).** The header is **app-shell chrome, not a condensed toolbar**
(`COMPONENT-MANIFEST` §2.4, §8.2 *"give the app real chrome"*). Three regions, in the order the guide
reads them: **identity** (mark, title and the one-sentence purpose — so the purpose is answerable on the
first screen), **what is being measured** (a labelled `role="group"` holding county · service line ·
standard · staffing gate · origins), and **actions** (page `nav`, the hearing packet, the theme toggle).
Every control carries a **visible label**, as this recipe's own ASCII skeleton draws them
(*"Service line: Primary care ▾"*) — the first build shipped bare selects with only an `aria-label`,
which reads as a toolbar and leaves the user guessing what a dropdown is *for*. Visible labels also
satisfy WCAG 3.3.2 more strongly than an invisible name, and avoid the 2.5.3 label-in-name mismatch.

The header spends the manifest's **design tokens rather than magic numbers** (§8.2): `--strata-space-*`
for rhythm, `--strata-radius-*`, `--strata-elevation-1` for the surface, `--strata-motion-fast` +
`--strata-ease` for state transitions, and the type steps (`--strata-h3`, `--strata-body2`, plus a
`--strata-label` step for the caps labels). The colour roles are **aliased onto the manifest's names**
(`--strata-accent`, `--strata-panel-bg`, `--strata-fg`, `--strata-border`), so porting this chrome into
the `<StrataApp>` build in §5 is a rename rather than a redesign. Page nav is a real `<nav>` and marks
the active page with **`aria-current="page"`**, not just a CSS class. Below 1,240 px the control group
wraps to its own line; below 860 px it collapses behind a **"Standard & origins" disclosure** — the
recipe's bottom-dock sheet, minus the dock.

**Basemap gallery — keyless, OSM-derived only, user-switchable:** **CARTO Dark Matter** (default,
paired to the dark mode) · **CARTO Positron** (the quietest ground for the light / print mode) ·
CARTO Voyager · OpenStreetMap · **OpenTopoMap** — the last is not decorative: terrain is *why* a rural drive is 90
minutes, and a hearing audience recognises the mountain between them and the hospital.
⚠ **MapLibre paint cannot read CSS variables**, so ink-coloured layers (the inside/outside surfaces,
the route line) must be **re-painted on every light↔dark basemap flip**. Polygon fills ~40/255 so the
coverage surface, the HPSA overlay and the tracts stack readably.

**Language: EN + AR/RTL.** The vocabulary is generic health-system planning (service line, minutes,
population, standard) and ministry-level access planning is a real GCC use case. Ship `@strata/i18n`
+ `lang-switch` with **logical CSS properties** throughout so the curve, its rules and the cohort
list mirror as a unit. The California statutory chips (Medi-Cal, Knox-Keene, SB 1300, AAS) are
labelled CA-only; an AR deployment carries a user-set standard instead.

### 2.3 KPI cards

Five, live via `stat` + a shared source — **no `connections` needed**:

| Card | Value | Threshold colouring |
|---|---|---|
| **People outside the standard** (`kpi`, `stat:{field:"TRACK_POP",op:"sum"}`) | the headline count; `deltaLabel` names the standard and its authority | `danger` when > 0, `success` at 0 |
| **Worst place** (`kpi`, `stat:{field:"travel_minutes",op:"max"}`) | the longest measured drive, with the place name | neutral |
| **Uninsured 18–64** (`kpi`, `stat:{field:"ACCESS2_CrudePrev",op:"avg"}`) | over the outside set only, recomputed on `extentChange` | `warning` above the state mean |
| **Aged 65+** (`kpi`, `stat:{field:"PERC_SENIOR_CY",op:"avg"}`) | over the outside set only | `warning` above the state mean |
| **Inside but thin** (`kpi`) | population inside on travel but in a service area over the ratio gate; names the count with no providers at all | `warning` when > 0 |
| **Deficit share** (`gauge`, thresholds 10/35, `invertColors`) | % of the county's population outside | green→amber→red |

Plus a **provenance chip** (`kpi`, `status`): *"as written — no exception on file"* /
*"as granted — AAS scenario, +31.6 mi, user-entered"* / *"origins: internal points — numbers not
reliable"*. **Never a green tick on a modelled result.**

### 2.4 Charts & table

- **`curve` (`coverage-curve`, §2.6)** — the cumulative distribution of population by travel cost with
  the statutory and granted rules on it. **This is the app.** It draws **two lines**: population without
  access on travel alone (solid, `primary`) and on **travel + staffing** (dashed, `warning`) — the gap
  between them is the population that is close enough to a site but in a service area below the
  provider-to-population ratio. The rule is draggable **and arrow-key operable**, and every move
  repaints the curve, the KPIs, the cohort list **and the map** (§6).
- **`cohortbars` (`stacked-bar`, `horizontal:true`)** — population by travel band
  (0–10 / 10–20 / 20–30 / 30–45 / 45–60 / 60–90 / 90+ min), the beyond-threshold bands in `danger`.
  This is also the day-1 fallback view for `coverage-curve`.
- **`cohorts` (`table`, virtualized `AttributeTablePanel`)** — the who-is-outside list on real
  columns: `MSSA_NAME · DEFINITION · TRACK_POP · travel_minutes · travel_miles · verdict`. Row →
  `zoomTo` + `flash`; CSV/GeoJSON export.
- **The cohort dossier** (`cohort` page) — the place's poverty and age profile on
  `PERCENT_100_FEDERAL_PROVERTY_LE` **[sic]**, `PERCENT_200_FEDERAL_PROVERTY_LE`,
  `AVERAGE_ECONOMIC_HARDSHIP_INDEX`, `PERC_SENIOR_CY`, and its `ACCESS2_CrudePrev` **with the
  `ACCESS2_Crude95CI` interval rendered**, never as a flat number.
- ⚠ **The declarative `chart` widget is not used anywhere.** `ChartPanel` destructures `charts` with
  no default and needs an `onQueryData` the layout engine does not inject — it throws at render.
  Everything chart-shaped here is `stacked-bar`, `sparkline`, or the new `coverage-curve`.

### 2.5 Capabilities to use (Phases 0–7)

- **`/analyze`** — `nearest` (nearest open site per origin), `buffer` (the **miles** leg of a standard,
  and the `access-engine` fallback), `pointsWithin` (sites in band), **`overlay` + `aggregate`** (roll
  tracts up to MSSA and county; intersect designation polygons with origins). **No `hexbin`/`hotspot`**
  — clustering 10,961 facilities reproduces the population map, and a Getis-Ord surface would imply a
  statistical significance a travel measurement does not have. **No `weighted-overlay`** — this app
  deliberately does not score places on weighted criteria; the thesis is one measured quantity against
  a legal threshold, and weighting would re-introduce the opacity the design exists to remove.
- **The two gates** — travel (road network) **and** staffing (population-to-provider ratio). Both are
  thresholds against a published standard, not weights: the app still scores nothing.
- **WIF `connections`** — §2.7; plus `dataSource.fromWidget:"access"` linking **seven** widgets with
  none, and `sourceId` linking two more.
- **Panels/widgets** — `layer-panel` and `basemap` as **map-anchored controls**, plus `legend`,
  `feature-info`, `data-actions`, `filter` ×3, `query`, `analysis`, `near-me` (the public-counter
  "how far am I" mode), `add-data` (catalog drawer over the 2,339-service library), `draw`/`measure`
  (both revert to `identify`), `bookmarks`, `share`, `status-bar`.
- **Plugins** — **`@strata/plugin-routing` is load-bearing**: `fetchIsochrone` draws the
  inside-the-standard contour (Valhalla) and the OSRM provider backs the duration/distance matrix.
  `@strata/plugin-search` (Nominatim) for address → "am I outside?". `@strata/plugin-statusbar`.
  **Not `plugin-timeslider`:** every layer has one vintage; a slider would animate nothing real.
- **Composed export** — the **hearing packet** (`/export report`: the curve, the threshold and its
  authority, the outside count, the cohort table with demographics and CIs, the map, the measured
  route to the nearest open site, and every number's source + vintage), an **atlas** (one page per
  MSSA for a county-wide submission), `/export image`, and a `share` deep-link with `setUrlParam` on
  **county**, **service line**, **standard** and **dragged threshold** so a packet is reproducible
  from its URL.
- **Modernization (§8 patterns)** — structured `theme`, app-shell (`header`/`footer`/`splash`),
  `splitter`/`panel`/`window`/`views`+`mapState`, **`FileDataSource`** (the candidate-site CSV, the
  closure list, and a plan's own contracted-provider roster — this is how the plan persona gets a
  real supply set), `RestDataSource` (the catalog JSON, the OSRM matrix, the Valhalla contour),
  `kpi`/`gauge` `stat` linking, `animate` (`fly` + `stagger` on the KPI row so a re-run *reads* as a
  recomputation; never on the map container).
- **Arcade** — `verdict` symbology on the origins layer (inside / covered-as-granted / outside); the
  **`CAST(AREA_SQMI AS FLOAT)`** normalisation; a "no estimate" label expression where a PLACES tract
  is missing; HPSA score banding (remember dental runs to 26, the others to 25).
- **`@strata/i18n` / `lang-switch` — used** (EN + AR/RTL). See §2.2.
- **Writes 🔶** — a county's own verification note on a place ("confirmed with the hospital
  2026-08-01", "AAS contested") via `updateRecord`, only behind `assertEsriBackend`. **The entire
  screening, curve, cohort and packet path is read-only and works on Strata Serve.**

### 2.6 The two new widgets (§10.2/§10.3) and their day-1 fallbacks

| Registry key | Purpose | Emits | Day-1 fallback (ships without any core change) |
|---|---|---|---|
| **`coverage-curve`** | the cumulative distribution of population by travel cost, with labelled, draggable statutory / granted thresholds, and a rendered confidence band where the demand variable is model-based | `rangeSelect`, `categorySelect`, `recordsChange` | a **`stacked-bar`** (`horizontal:true`) of population by travel band with the beyond-threshold bands in `danger`, **plus** a `kpi` whose `deltaLabel` carries the threshold and its authority, **plus** a `sparkline` of the cumulative series, **plus** the `table` sorted by travel cost descending. Loses the drag and the ghost rule; **keeps the distribution, the threshold and the count** |
| **`access-engine`** | owns the arithmetic — population-weighted origins → road-network travel matrix → nearest in-service-line site → verdict against the applicable standard *and* against the granted alternative; publishes the whole result as an output | `recordsChange`, `countChange` | the shipped **`analysis`** widget over `@strata/processing`: `buffer` (the statute's miles leg as rings) + `pointsWithin` + `nearest` + `aggregate`, with `plugin-routing`'s `fetchIsochrone` for the minutes leg. Degrades road-network distance to straight-line rings, **badged *"straight-line — not the standard's measure"***. Keeps the entire loop, the verdict and the packet |

Two `access-engine` props that are **not cosmetic**:

- **`originPolicy: "population-weighted" | "block-group" | "internal-point"` (default
  population-weighted).** The alternative *manufactures deserts* — §4 cmd 13 proves the same 3,879
  people are **1.7 minutes** or **152.5 minutes** from the same hospital depending on this one
  setting. When `internal-point` is chosen, the widget must badge **every** number it emits.
- **`missingPolicy: "flag" | "exclude"` (default `flag`).** An origin with no reachable site in the
  service line must render as **unmeasured**, never as a very large travel time. *"We could not
  measure this"* and *"this is 400 minutes away"* are different findings and only one of them is true.

Both obey the §10.2 contract: app-local `registry` override, `--strata-*` tokens only, logical CSS
properties, the map driven **through the store**, data via `dataSource`, cross-widget behaviour
declared in `connections` — never hard-wired to a sibling.

> **Both are implemented in `app/` already**, as plain functions rather than registry widgets:
> `coverage-curve` is `drawCurve()` (an SVG with a pointer-captured draggable rule) and
> `access-engine` is `apportionOrigins()` + `capOrigins()` + `measure()` + `summarise()`. Porting
> them into the widget contract is mechanical — the arithmetic and its 195 assertions come across
> unchanged. `apportionOrigins` gained a third branch during the build (see §2.6.1).

#### 2.6.1 What the build changed about `originPolicy`

The design specified two origin policies. Building it against 58 real counties forced a third
branch, because **California contains both regimes and one rule cannot serve them**:

| Regime | Evidence | Rule |
|---|---|---|
| **Urban** — a small tract lying wholly inside a settlement | Sacramento County: **342 of 364** tracts have their internal point inside a mapped place | keep the tract's own internal point. It *is* where the people are |
| **Rural** — a huge tract containing a few settlements | Modoc County: only **1 of 4**; Trinity: four tracts of 449–1,431 sq mi, the county seat inside one of them | apportion `TRACK_POP` across the CDC PLACES place points inside the tract, weighted by each place's own `TotalPopulation` |
| **Unmapped** — no settlement at all | the residual | internal point, **flagged**, and the flag travels into the cohort list and the packet |

The tract remains the denominator in all three, so population is conserved exactly and percentages
stay on the authoritative frame. Every origin carries a `via` string saying which branch placed it,
and that string is printed in the UI — the modelling choice is never invisible.

### 2.7 `connections` (18 — every wire uses a shipped emitter)

`buttonClick`/`timer`/`mapClick`/`sketchComplete` **emitters are Phase-2 pending**, so nothing
load-bearing depends on them; the two new widgets emit only shipped trigger types.

| from | trigger | to | action | options |
|---|---|---|---|---|
| `svcline` | `filterChange` | `map` | `filter` | `{ "layerId": "facilities" }` |
| `svcline` | `filterChange` | `cohorts` | `filter` | — |
| `svcline` | `filterChange` | — | `setUrlParam` | `{ "param": "svc" }` |
| `countypick` | `filterChange` | `map` | `filter` | `{ "layerId": "origins" }` |
| `countypick` | `filterChange` | — | `setUrlParam` | `{ "param": "county" }` |
| `stdpick` | `filterChange` | `curve` | `filter` | — |
| `stdpick` | `filterChange` | — | `setUrlParam` | `{ "param": "std" }` |
| `curve` | `rangeSelect` | `map` | `filter` | `{ "layerId": "origins" }` |
| `curve` | `rangeSelect` | `cohorts` | `filter` | — |
| `curve` | `rangeSelect` | — | `setUrlParam` | `{ "param": "t" }` |
| `curve` | `categorySelect` | `cohortbars` | `filter` | — |
| `access` | `recordsChange` | `map` | `filter` | `{ "layerId": "origins" }` |
| `access` | `recordsChange` | `gauge-deficit` | `showStatistics` | — |
| `cohorts` | `rowSelect` | `map` | `zoomTo` | `{ "layerId": "origins" }` |
| `cohorts` | `rowSelect` | `map` | `flash` | `{ "layerId": "origins" }` |
| `cohorts` | `rowSelect` | `evidence` | `viewInTable` | — |
| `map` | `featureSelect` | `cohorts` | `viewInTable` | — |
| `map` | `extentChange` | `kpi-unins` | `showStatistics` | — |

**Zero-connection links** (`dataSource.fromWidget` / `sourceId`): `curve`, `kpi-outside`,
`kpi-worst`, `gauge-deficit`, `cohorts`, `cohortbars` and `evidence` all bind to `access`;
`kpi-unins` binds to `places-tracts` and `kpi-senior` to `enriched-tracts`. **Nine widgets alive with
no wiring at all.**

## 3. Data sources

All EPSG:4326 (reproject on ingest); `OBJECTID` is the OID throughout. California is the researched
column; Maryland is included only where a **verified** equivalent exists — and for this vertical it
mostly does not, which is stated rather than filled.

> **The finding the catalog encodes.** The crawled California ArcGIS library (6 servers, 2,339
> services) carries an **emergency-management** view of health and essentially no **care-delivery**
> view. Role-tagging it for access to care yields **8 care-site services** — all Cal OES/Caltrans
> continuity-of-operations layers (Hospitals as critical infrastructure, medical-surge facilities,
> disaster medical response partners, hospital heliports) — and **zero health shortage-designation
> services** (the three that match "shortage" are DWR **water**-shortage vulnerability layers).
> Nine further services are keyword false friends (CAL FIRE fuel-"treatment" areas, a waste-water
> "treatment" plant, Cal OES "Hospitality") and are excluded by name. **Every layer this app is
> actually built on is external — HCAI, HRSA and CDC.** That is the finding, not a gap in the crawl.

| Role | California | Maryland | National / federal | CORS · licence · gotcha |
|---|---|---|---|---|
| **Supply — the facility spine** | **HCAI `Current_Healthcare_Facility_Listing`** — **10,961** facilities (**10,946 Open**, **458** General Acute Care Hospitals); `FACILITY_LEVEL_DESC` (**18** service levels), `ER_SERVICE_LEVEL_DESC` (**5**), `TOTAL_NUMBER_BEDS`, `LATITUDE`/`LONGITUDE`, `COUNTY_NAME`, `OSHPD_ID` | — (MD licensed-facility service not verified in this pass) | HRSA Health Sites (128,697 national) | CORS `*`. ⚠ geometry is **Web Mercator 3857** — request `outSR=4326`. ⚠ `maxRecordCount` **1000**, not 2000. ⚠ `COUNTY_NAME` is bare ("Trinity") while HCAI's MSSA layer says "Trinity County" — **normalise or the join silently returns nothing**. ⚠⚠ **facility-open ≠ service-line-open** |
| **Supply — safety net** | **HRSA `Health_Center_Service_Delivery_and_LookAlike_Sites`** — 18,963 national, **3,020 CA**; `Site_Name`, **`Operating_Hours_per_Week`**, `Health_Center_Type`, site-setting description | — | same service | CORS `*`. **`Operating_Hours_per_Week` is the field nobody uses and everybody needs** — a site open 8 h/week is not the supply a 60 h/week site is. Mobile-van and school-based settings are flagged in the setting description and **must not be scored as fixed origins**. Field names are 40+ character HRSA warehouse labels — bind verbatim |
| **Supply — enrichment (stale)** | **HCAI `CDPH_healthcare_facility_locations`** — 11,309; `NPI`, `CAPACITY`, `TRAUMA_CTR` (**77**, LEVEL I–IV), `CRITICAL_ACCESS_HOSPITAL` (**34**), `TYPE_OF_CARE`, `FAC_FDR` (**21** types incl. **PRIMARY CARE CLINIC**), **`BIRTHING_FACILITY_FLAG='YES'` (257)** | — | — | CORS `*`. ⚠⚠⚠ **Newest `LICENSE_EXPIRATION_DATE` is 2021-11-15 — this is a ~2021 snapshot.** Use it for NPI / trauma / critical-access enrichment; **never render the birthing flag as current availability** (§4 cmd 2) |
| **Supply — provider density** | HCAI `Providers` — **11,269** points, `F_provider_MSSA_ID`, `COUNTY` only | — | NPPES NPI registry | HCAI's own working artifact for the PCSA ratio: **no name, no specialty, no NPI, no FTE**. A density surface, not a directory |
| **Demand — the origin geography** | **HCAI `MSSA2024_Tract`** — **9,106** tracts, `TRACK_POP` (Σ **38,589,882**), `MSSA_ID`, `MSSA_NAME`, `COUNTY`, `DEFINITION` (Urban\|Rural), `AREA_SQMI`, `INTPTLAT`/`INTPTLON` | `mdgeodata` `Demographics/MD_AmericanCommunitySurvey/0` (1,394 tracts) | Census TIGERweb + ACS API | CORS `*`. ⚠ **`AREA_SQMI` is a String** — `orderBy` reports **99.21** sq mi as the largest CA tract when the truth is **982.13** (Glenn County); `where AREA_SQMI > 100` **errors 400**; `CAST(AREA_SQMI AS FLOAT) > 100` = **267**. ⚠⚠⚠ **`INTPTLAT`/`INTPTLON` are TIGER geometric internal points, not population-weighted** — see §4 cmd 13 |
| **Demand — equity profile** | **HCAI `MSSA_Tracts_Demographics`** — 9,106; **`PERCENT_100_FEDERAL_PROVERTY_LE`**, **`PERCENT_200_FEDERAL_PROVERTY_LE`**, `AVERAGE_ECONOMIC_HARDSHIP_INDEX`, `PERCENT_HISPANIC` and five other race shares | MD ACS layer exposes `CV_*` coefficient-of-variation columns — an honesty feature worth copying | Census ACS | ⚠ **the poverty columns are misspelled in the schema — PROVERTY, and truncated at 30 chars.** Bind the typo verbatim; the corrected name returns "Invalid field" |
| **Demand — insurance & utilisation** | **CDC PLACES Tracts layer 3** — **9,070 CA**; **`ACCESS2_CrudePrev`** (uninsured 18–64) + `ACCESS2_Crude95CI`, `CHECKUP_`, `DENTAL_`, `MHLTH_`, `DEPRESSION_` | same national service | same | CORS `*`. ⚠ **model-based small-area estimates, not counts** — every measure ships a 95% CI and hiding it overstates precision. ⚠ **9,070 vs 9,106 vs 9,107** — three tract universes across the three demand layers; render missing tracts as "no estimate", never zero. Filter `StateAbbr='CA'` before counting |
| **Demand — age structure** | **Cal OES `CA_CensusTracts_2023`** — 9,107 tracts, 164 Esri-enriched fields; `POP65_CY`, `SENIOR_CY`, **`PERC_SENIOR_CY`**, `AGEDEP_CY`, `MEDAGE_CY`, `TOTHH_CY`, `MEDHINC_CY`, `_FY` projections | `mdgeodata` ACS tracts | Census ACS | CORS `*`. ⚠ `_CY`/`_FY` are **Esri vintages, not census years**. ⚠ **licensed derivative** — check redistribution, keep the ACS fallback wired. ⚠ carries **no insurance and no vehicle variable** |
| **Demand — the missing variable** | — | — | **Census ACS `B08201` (no-vehicle households)**, `S2701` (uninsured) | ⚠ a bare keyless subject-table call **redirected HTTP 302 with no body on 2026-07-29**; `&in=` clauses must be separate parameters and production wants a free key. **Until it is wired, every travel number must say "car assumed"** |
| **Designation — federal** | **HRSA HPSA Primary Care designation boundaries** — 3,087 national, **180 CA** (`PriStNM='California'`), all Designated; `HpsScore` 1–25, `HpsTypDes`, `HpsPpPdRtG`, `RurStatDes`. Mental-health components **2,363 CA**; dental components **1,131 CA** | same national service | same (Federal AGOL org) | CORS `*`. ⚠ **the three layers do not share a schema**: primary care has **no `StAbbr`/`CntNM`** (filter on `PriStNM` or get "Invalid field") and is 43 fields of *designations*; mental/dental are 64 fields of *components*. **Counting rows across disciplines compares different objects** — compare designated population instead |
| **Designation — California's own** | **HCAI `PCSA_-_Primary_Care_Shortage_Area`** — **542** MSSAs, **268** with `PCSA='Yes'`; `Provider_R` (population-to-provider ratio), `EST_Physic`, `EST_FNPPA`, `PCT_100FPL`, `Score_Prov`, `Score_Pove`, `Score_Tota`, `Effective` = **2020-01-30** | — | — | CORS `*`. ⚠⚠⚠ **It does not join to `MSSA2024_Tract`.** 542 vs 366 distinct `MSSA_ID`; **exact overlap 6**; after normalising the zero-padding **31 of 542**; `MSSA_NAME` overlap **0**. The 2024 MSSA redefinition renumbered the geography and the shortage layer was not re-cut. **Join spatially, print both vintages** (§4 cmd 12) |
| **Designation — facility-based** | HCAI `HPSA_PNTPC_SHP` — **4,850** point HPSAs (health centres, correctional, tribal) | — | HRSA | Additive to the polygons, not an alternative: a county can look covered on the polygon layer and still contain facility HPSAs |
| **Travel — the engines** | **OSRM demo `/table`** (`https://router.project-osrm.org`) — road-network **duration + distance** matrix; **Valhalla** (`https://valhalla1.openstreetmap.de/isochrone`) — keyless contours | same | same | **CORS `*` on OSRM** (verified). ⚠ **100-coordinate cap per `/table` request** — architect as one origin × 99 destinations and batch. ⚠ **community demo instances under fair use** — a statewide sweep belongs on a self-hosted engine. ⚠ **car-only, free-flow, no time of day, unstated OSM vintage** — record engine + date beside every published number |
| **Travel — car-free leg** | Caltrans `CHrailroad/CA_HQ_Transit_Stops` (**48,662**; **`avg_trips_per_peak_hr`**, `hqta_type`, `agency_primary`) + `CA_HQ_Transit_Areas` (20,558) | — | GTFS / Transitland | Caltrans **reflects the request Origin** (browser-safe). ⚠ `stop_id` is **only unique within `agency_primary`**. ⚠ frequency ≠ connectivity: it tells you service exists, not that a route joins this origin to that clinic |
| **Emergency leg** | Cal OES `AmbulanceServices` — **209** | — | HIFLD EMS stations | ⚠ **no layer 0 — it is `FeatureServer/1`**. USGS-structures lineage: base-of-operations addresses only, **no units, no staffing, no response zones**. Render as context; **refuse to compute a response time from it** |
| **Frame** | State geoportal `Boundaries/CA_Counties` · CAL FIRE `California_Incorporated_City_Boundaries` (**483**) | `mdgeodata` `Boundaries/MD_CensusStatisticalBoundaries` | Census TIGERweb | **The standard is set per county** — the county polygon is the lookup key that decides which threshold applies. ⚠ unincorporated territory is in no city layer, and unincorporated rural California is exactly where the deficit lives: render "no city" as *"unincorporated — county jurisdiction"* |
| **Barrier context** | CA **Healthy Places Index 3.0** (tract) · Caltrans `CHhqcore/Environmental_Health`, `Economic_Opportunity` | — | CDC/ATSDR SVI | ⚠ the HPI URL commonly found is an **LA County republication**, not the publisher's own service — confirm vintage with the Public Health Alliance. ⚠ the Caltrans indices are **transport-planning equity indices, not health measures**; a low score may mean high transport cost |
| **The staffing test** | **HCAI `PCSA_-_Primary_Care_Shortage_Area`** — `Provider_R` (population per provider), `EST_Provid`, `EST_Physic`, `EST_FNPPA`, joined **spatially** | — | HRSA HPSA `HpsPpPdRtG`, `HpsFte` | The second of the two tests every framework applies. Gate at **3,500:1** (HRSA primary care; 3,000:1 high need). ⚠⚠ **`Provider_R = 0` on 30 MSSAs means NO PROVIDERS AT ALL** (`EST_Provid`=0, provider score 5/5, PCSA=Yes) — the worst case, not the best. A naive `ratio > threshold` scores every one of them as **passing**. ⚠ Vintage 2020-01-30; **111 of 542 MSSAs fail the gate, holding 3,600,562 Californians (9.4%)** |
| **Equity profile** | **HCAI `2025_MSSAs_from_HCAI`** — `PERCENT_10` (under 100% FPL), `PERCENT_20` (200% FPL), `PERCENT_HI`, `AVERAGE_EC`, **`TOTAL_POP_`** | — | Census ACS | ⚠⚠⚠ **Do NOT use `MSSA_Tracts_Demographics`, the obviously-named layer: it is destroyed by its own schema.** Its `PERCENT_*` columns are **Integer** while the values are 0–1 proportions, so every one truncates — statewide the poverty column runs min 0, max 1, **mean 0.0076**, i.e. essentially every California tract reports 0% poverty. Its `MSSA_POP` also repeats the MSSA total on every tract. This 2025 sibling carries the same measures as **Doubles** with real values and the true tract population, under shapefile-truncated 10-character names |
| **The standard itself** | **DHCS** Medi-Cal time-and-distance standards (Attachment B / APL 23-001) · **DMHC** Knox-Keene + 28 CCR §1300.67.2.2 timely access | — | CMS marketplace + Medicare Advantage standards; **state-based marketplaces adopt quantitative time-and-distance standards from 1 Jan 2026** | ⚠ **It is a PDF, and `dhcs.ca.gov` returns HTTP 403 to a server-side fetch (2026-07-29)** — open it in a browser at build time. The defaults shipped here (Medi-Cal **10 mi / 30 min** primary care, **15 mi / 30 min** hospital; Knox-Keene **15 mi / 30 min**) are page-verified from secondary summaries; **the per-county alternatives were not verified in this pass** (§4 cmd 17). ⚠ The standard is written as **distance OR time**, not AND — the `verdict` arithmetic must use `OR` or it will over-report the deficit |
| **The exception register** | **none published as data** — DHCS AAS approvals exist only inside filings | — | — | ⚠ ~**10,000 AAS approved in Jan 2019 alone, averaging +31.61 miles** (NHeLP). With no machine-readable docket, the "as granted" rule is a **labelled scenario** or a user-supplied filing — **never rendered as a fact**. A `placeholder` widget holds the docket panel until a jurisdiction supplies one |
| **The catalog itself** | `access-catalog-ca.json` — 2,339 services, **456 role-tagged** (11 roles), **26 external feeds**, 9 false friends excluded | — | — | Built by `build_access_catalog.py`; bound as a `RestDataSource` behind the `add-data` drawer |

Attribution: California Department of Health Care Access and Information (HCAI), California Department
of Public Health, Cal OES, CAL FIRE, Caltrans, California Department of Technology/GIO, US Health
Resources and Services Administration (HRSA), US Centers for Disease Control and Prevention (CDC
PLACES), US Census Bureau, Esri (demographic enrichment), OpenStreetMap contributors and Project OSRM
/ FOSSGIS Valhalla — each with "no warranty; screening use only."

## 4. Verify each URL first (terminal)

Every command below was run on **2026-07-29** and its output is quoted in §3's gotcha column.

```bash
HCAI=https://services5.arcgis.com/fMBfBrOnc6OOzh7V/arcgis/rest/services
FED=https://services2.arcgis.com/FiaPA4ga0iQKduv3/arcgis/rest/services
PLACES=https://services3.arcgis.com/ZvidGQkLaDJxRSJ2/arcgis/rest/services/PLACES_LocalData_for_BetterHealth/FeatureServer/3
OES=https://services.arcgis.com/BLN4oKB0N1YSgvY8/arcgis/rest/services

# 1. THE SUPPLY SPINE. Note maxRecordCount 1000 and the Web-Mercator geometry.
curl -s "$HCAI/Current_Healthcare_Facility_Listing/FeatureServer/0/query?where=1=1&returnCountOnly=true&f=json"
#  -> {"count":10961}
curl -s "$HCAI/Current_Healthcare_Facility_Listing/FeatureServer/0/query?where=FACILITY_STATUS_DESC='Open'&returnCountOnly=true&f=json"
#  -> {"count":10946}
curl -s "$HCAI/Current_Healthcare_Facility_Listing/FeatureServer/0/query?where=LICENSE_CATEGORY_DESC='General Acute Care Hospital'&returnCountOnly=true&f=json"
#  -> {"count":458}
curl -s "$HCAI/Current_Healthcare_Facility_Listing/FeatureServer/0/query?where=1=1&outFields=ER_SERVICE_LEVEL_DESC&returnDistinctValues=true&returnGeometry=false&f=json"
#  -> Emergency - Basic | Emergency - Comprehensive | Emergency - Standby | None | Not Applicable
#     FACILITY_LEVEL_DESC has 18 values incl. Community Clinic, Free Clinic, Chronic Dialysis Clinic,
#     Psychiatric Health Facility, Alternative Birthing Center, Surgical Clinic, Skilled Nursing Facility.
#     ⚠ maxRecordCount is 1000 (not 2000). ⚠ served geometry is wkid 3857 -> request outSR=4326.
#     ⚠ LATITUDE IS NULL returns {"count":0} — all 10,961 rows carry usable coordinates.

# 2. THE MATERNITY TRAP — the only mapped birthing flag in open CA data is a 2021 fossil,
#    and it is FACILITY-level, not SERVICE-LINE-level.
curl -s "$HCAI/CDPH_healthcare_facility_locations/FeatureServer/0/query?where=1=1&outFields=BIRTHING_FACILITY_FLAG&returnDistinctValues=true&returnGeometry=false&f=json"
#  -> [null, "YES"]      (257 rows carry YES)
curl -s "$HCAI/CDPH_healthcare_facility_locations/FeatureServer/0/query?where=1=1&outStatistics=[{\"statisticType\":\"max\",\"onStatisticField\":\"LICENSE_EXPIRATION_DATE\",\"outStatisticFieldName\":\"mx\"}]&returnGeometry=false&f=json"
#  -> mx = 1636934400000  =  2021-11-15      <-- the newest licence in the whole layer
# Cross-check the 257 against the CURRENT listing on OSHPD_ID:
#  -> 6 absent from the current listing (all free-standing birth centres); 0 are non-Open; 246 are GACH.
#     => facility-level the layer looks fine. SERVICE-level it is wrong: 56 California maternity wards
#        have closed since 2012 (SB 1300 findings) and Corona Regional closed L&D on 30 Jan 2026.
#     NEVER render this flag as current availability.

# 3. THE ORIGIN GEOGRAPHY — California's own MSSA tracts.
curl -s "$HCAI/MSSA2024_Tract/FeatureServer/0/query?where=1=1&returnCountOnly=true&f=json"          # -> 9106
curl -s "$HCAI/MSSA2024_Tract/FeatureServer/0/query?where=1=1&outFields=DEFINITION&returnDistinctValues=true&returnGeometry=false&f=json"
#  -> Urban | Rural          (923 tracts are Rural; the PCSA layer additionally uses "Frontier")
#     Σ TRACK_POP over all 9,106 tracts = 38,589,882

# 4. THE MISSPELLED COLUMN. Bind the typo or the query fails.
curl -s "$HCAI/MSSA_Tracts_Demographics/FeatureServer/0?f=json" | python -c "
import json,sys; d=json.load(sys.stdin)
print([f['name'] for f in d['fields'] if 'PROVERTY' in f['name']])"
#  -> ['PERCENT_200_FEDERAL_PROVERTY_LE', 'PERCENT_100_FEDERAL_PROVERTY_LE']    <-- PROVERTY, truncated

# 5. TRAP — AREA_SQMI IS A STRING. Same family as the Caltrans AADT trap.
curl -s "$HCAI/MSSA2024_Tract/FeatureServer/0?f=json" | python -c "
import json,sys; d=json.load(sys.stdin)
print([(f['name'],f['type']) for f in d['fields'] if f['name'] in ('AREA_SQMI','INTPTLAT','MSSA_ID')])"
#  -> [('MSSA_ID','esriFieldTypeString'), ('AREA_SQMI','esriFieldTypeString'), ('INTPTLAT','esriFieldTypeString')]
curl -s "$HCAI/MSSA2024_Tract/FeatureServer/0/query?where=1=1&outFields=AREA_SQMI,COUNTY&orderByFields=AREA_SQMI+DESC&resultRecordCount=3&returnGeometry=false&f=json"
#  -> '99.21242625482625' (Calaveras) reported as the LARGEST tract in California.
#     The true maximum is '982.1266911196911' (Glenn County). '99' > '982' as strings.
curl -s "$HCAI/MSSA2024_Tract/FeatureServer/0/query?where=AREA_SQMI+%3E+100&returnCountOnly=true&f=json"
#  -> {"error":{"code":400,...,"details":["Unable to perform query. Please check your parameters."]}}
curl -s "$HCAI/MSSA2024_Tract/FeatureServer/0/query?where=CAST%28AREA_SQMI+AS+FLOAT%29+%3E+100&returnCountOnly=true&f=json"
#  -> {"count":267}     <-- 267 tracts >= 100 sq mi, holding 836,798 people (2.2% of the state)

# 6. THE SAFETY NET.
curl -s "$HCAI/Health_Center_Service_Delivery_and_LookAlike_Sites/FeatureServer/0/query?where=Site_State_Abbreviation='CA'&returnCountOnly=true&f=json"
#  -> {"count":3020}     (18,963 nationally)

# 7. CALIFORNIA'S OWN SHORTAGE SCORE.
curl -s "$HCAI/PCSA_-_Primary_Care_Shortage_Area/FeatureServer/0/query?where=1=1&returnCountOnly=true&f=json"        # -> 542
curl -s "$HCAI/PCSA_-_Primary_Care_Shortage_Area/FeatureServer/0/query?where=PCSA='Yes'&returnCountOnly=true&f=json" # -> 268
curl -s "$HCAI/PCSA_-_Primary_Care_Shortage_Area/FeatureServer/0/query?where=1=1&outFields=MSSA_ID,MSSA_NAME,PCSA,Provider_R,Score_Tota&resultRecordCount=3&returnGeometry=false&f=json"
#  -> {'MSSA_ID':'10','MSSA_NAME':'Oroville/Palermo/Thermalito','PCSA':'Yes','Provider_R':1215.8,'Score_Tota':5}
#     Effective = 1580342400000 = 2020-01-30

# 8. THE INSURANCE VARIABLE (model-based — carry the CI).
curl -s "$PLACES/query?where=StateAbbr='CA'&returnCountOnly=true&f=json"      # -> 9070   (NOT 9,106 or 9,107)
#     ACCESS2_CrudePrev = lack of health insurance, adults 18-64, + ACCESS2_Crude95CI

# 9. THE AGE STRUCTURE.
curl -s "$OES/CA_CensusTracts_2023/FeatureServer/0?f=json" | python -c "
import json,sys,re; d=json.load(sys.stdin); f=[x['name'] for x in d['fields']]
print(len(f),'fields'); print([n for n in f if re.search(r'SENIOR|POP65|AGEDEP',n)])"
#  -> 164 fields; ['SENIOR_CY','AGEDEP_CY','POP65_CY','PERC_SENIOR_CY']

# 10. THE FEDERAL DESIGNATION — and the schema trap. This layer has NO StAbbr column.
curl -s "$FED/Health_Professional_Shortage_Areas_in_Primary_Care_Designation_Boundaries/FeatureServer/0/query?where=HPSA_State_Abbreviation='CA'&returnCountOnly=true&f=json"
#  -> {"error":{"code":400,...,"details":["'Invalid field: HPSA_State_Abbreviation' parameter is invalid"]}}
curl -s "$FED/Health_Professional_Shortage_Areas_in_Primary_Care_Designation_Boundaries/FeatureServer/0/query?where=PriStNM='California'&returnCountOnly=true&f=json"
#  -> {"count":180}        (3,087 nationally; all 180 are HpsStatDes='Designated')

# 11. …and its two siblings use a DIFFERENT schema and a different granularity.
curl -s "$FED/Health_Professional_Shortage_Areas_in_Mental_Health_Component_Boundaries/FeatureServer/0/query?where=StAbbr='CA'&returnCountOnly=true&f=json"   # -> 2363
curl -s "$FED/Health_Professional_Shortage_Areas_in_Dental_Health_Component_Boundaries/FeatureServer/0/query?where=StAbbr='CA'&returnCountOnly=true&f=json"   # -> 1131
#     180 vs 2,363 vs 1,131 is NOT a finding about mental health — primary care is published as
#     DESIGNATION boundaries (43 fields) and the other two as COMPONENT boundaries (64 fields).
#     Compare designated POPULATION, never row counts. (mental-health maxRecordCount is 1000.)

# 12. THE JOIN THAT DOES NOT EXIST — California's shortage score vs California's current geography.
#     (paged distinct on both layers)
#  -> MSSA2024_Tract distinct MSSA_ID: 366   e.g. '001.1','001.11','001.2'   (ZERO-PADDED)
#  -> PCSA          distinct MSSA_ID: 542   e.g. '1.1','1.2','10','100'      (UNPADDED)
#  -> EXACT string overlap: 6
#  -> after stripping the zero-padding: 31 of 542
#  -> MSSA_NAME overlap: 0   (262 vs 542 distinct; 'Alturas city, Cedarville CDP' vs
#                             'Oroville/Palermo/Thermalito' — different naming conventions entirely)
#     => the 2024 MSSA redefinition renumbered the geography; the 2020 PCSA layer was not re-cut.
#        JOIN SPATIALLY. Print both vintages beside the score.

# 13. ⚠⚠⚠ THE ONE THAT DECIDES WHETHER THE APP IS HONEST.
#     Tract INTPTLAT/INTPTLON are TIGER GEOMETRIC internal points, not population-weighted centroids.
#     Live OSRM measurement, Trinity County:
#       Weaverville town centre       (40.7307,-122.9414) -> Trinity Hospital:    1.7 min /  0.6 mi
#       tract 06105000102 int. point  (41.0094,-122.8214) -> Trinity Hospital:  152.5 min / 36.8 mi
#       tract 06105000101 int. point  (41.0172,-122.6402) -> Trinity Hospital:   66.6 min / 41.1 mi
#     Tract 06105000102 holds 3,879 people. Measured from internal points, 100% of Trinity County's
#     12,327 residents fall outside BOTH the 30-minute and the 10-mile Medi-Cal standard — in a county
#     whose seat has a general acute care hospital in town.
#     Blast radius: 267 tracts >= 100 sq mi (cmd 5) holding 836,798 people.
#     => ORIGINS MUST BE POPULATION-WEIGHTED. If internal points are used at all, badge every number.

# 14. THE ROUTING ENGINES — keyless, CORS-open, and rate-limited.
curl -s "https://router.project-osrm.org/table/v1/driving/<100 lon,lat pairs>?sources=0&annotations=duration,distance"
#  -> {"code":"Ok", ...}   100 coordinates accepted; the 100-coord cap is the batching constraint
curl -sD- -H "Origin: http://localhost:8060" "https://router.project-osrm.org/route/v1/driving/-121.4,38.58;-121.3,38.6?overview=false" | grep -i access-control
#  -> Access-Control-Allow-Origin: *
curl -s -o /dev/null -w "%{http_code}\n" "https://valhalla1.openstreetmap.de/isochrone?json={...contours:[{time:15}],polygons:true}"
#  -> 200

# 15. THE LAYER-ID TRAP.
curl -s "$OES/AmbulanceServices/FeatureServer/0?f=json"
#  -> {"error":...,"details":["The requested layer (layerId: 0) was not found."]}
curl -s "$OES/AmbulanceServices/FeatureServer/1/query?where=1=1&returnCountOnly=true&f=json"   # -> 209

# 16. CORS + the remaining counts — everything the browser app touches.
curl -s "$CT/CHrailroad/CA_HQ_Transit_Stops/FeatureServer/0/query?where=1=1&returnCountOnly=true&f=json"   # -> 48662
curl -s "$CT/CHrailroad/CA_HQ_Transit_Areas/FeatureServer/0/query?where=1=1&returnCountOnly=true&f=json"   # -> 20558
curl -s "$CF/California_Incorporated_City_Boundaries/FeatureServer/0/query?where=1=1&returnCountOnly=true&f=json"  # -> 483
#  -> HCAI (services5), Federal AGOL (services2), CDC PLACES (services3), Cal OES, CAL FIRE: `*`
#  -> Caltrans and the CA state geoportal REFLECT the request Origin.  Both fine.

# 17. ⚠ THE STANDARD ITSELF IS NOT MACHINE-READABLE — and its own PDF blocks a server-side fetch.
curl -s -o /dev/null -w "%{http_code}\n" "https://www.dhcs.ca.gov/wp-content/uploads/2025/10/Attachment-B-Time-and-Distance-Standards.pdf"
#  -> 403      (dhcs.ca.gov refuses server-side fetches; open it in a browser at build time)
#     The headline standards used as defaults here are page-verified from secondary sources
#     (Disability Rights California's plain-language summary of the Medi-Cal standards; the
#     Knox-Keene summary): Medi-Cal primary care 10 mi / 30 min, Medi-Cal hospital 15 mi / 30 min,
#     Knox-Keene 15 mi / 30 min.  **The PER-COUNTY alternative thresholds in DHCS Attachment B were
#     NOT verified in this pass** — they must be transcribed from the PDF in a browser before a
#     deployment quotes a county-specific number. Until then the app shows the statewide default
#     with its authority printed beside it, and the wizard's Q4 says so.
```

**The arithmetic, from these fields only** (print it in the packet beside every number):

```
# per origin o (a population-weighted point carrying weight w[o] = TRACK_POP), per service line L
supply(L)   = HCAI Current_Healthcare_Facility_Listing
              WHERE FACILITY_STATUS_DESC='Open' AND FACILITY_LEVEL_DESC IN (<L's levels>)
              [UNION HRSA FQHC sites WHERE Site_State_Abbreviation='CA'  — for primary care/dental]

t[o]        = MIN over s in supply(L) of  OSRM driving duration(o -> s)      # minutes
d[o]        = MIN over s in supply(L) of  OSRM driving distance(o -> s)      # miles
              (batched 1 origin x 99 destinations; candidate set pre-filtered by great-circle radius)

verdict[o]  = "inside"            if  t[o] <= T.minutes  OR  d[o] <= T.miles     # the standard is OR
              "covered-as-granted" if outside T but inside the GRANTED alternative
              "outside"           otherwise
              "unmeasured"        if no reachable site   # missingPolicy="flag": NEVER a large number

outside     = Σ w[o] over verdict[o] == "outside"
deficit_pct = 100 * outside / Σ w[o]
curve(x)    = 100 * ( Σ w[o] where t[o] > x ) / Σ w[o]        # the cumulative coverage curve

# The two rules drawn on the curve:
#   "as written"  x = T.minutes            (authority printed beside it)
#   "as granted"  x = T + AAS delta        (LABELLED A SCENARIO unless the filing is supplied)
#
# originPolicy = "population-weighted" (default). "internal-point" is offered ONLY with a badge on
#   every emitted number — see cmd 13.
# Every published number carries: engine + OSM vintage + layer vintage + "car assumed".
```

## Guided wizard — **the prompts that assign the app's defaults**

Claude runs this like an ArcGIS Instant-App wizard: ask each group, **apply the default so "accept
all" builds a complete app**, confirm a one-line summary, then run §5. Launch with
`/recipe health_access-to-care`. Every answer *sets an application default* baked into `layers.json` /
the `AppLayout`. Phrasing per `strata/docs/reference/human-language.md`.

| # | Wizard question | Options → **default** | The default it assigns |
|---|---|---|---|
| 1 · App | Title & as-of date? | free text → **"Access to Care — Who's Outside"**, today | header title + `strata:notes.asOf` + footer vintage line |
| 2 · Area | Which county or region? | any of the 58 CA counties · a multi-county region · statewide (self-hosted router required) → **one county, chosen by the user** | initial extent, the `origins` filter, and **which time-and-distance standard applies** |
| 3 · Service line | Which service? | primary care · hospital · emergency department · **obstetric (badged)** · behavioural health · dental · dialysis → **primary care (adult)** | the `facilities` `definitionExpression`, the supply union, and the designation overlay |
| 4 · Standard | Whose standard? | **Medi-Cal managed care (10 mi / 30 min primary care; 15 mi / 30 min hospital)** · Knox-Keene (15 mi / 30 min) · CMS marketplace · a custom threshold | the statutory rule on the curve and the `verdict` arithmetic — **and the app names the authority beside it**. ⚠ The wizard states that the **per-county** alternatives in DHCS Attachment B are not bundled (§4 cmd 17) and offers the custom threshold for a county-specific number |
| 5 · Staffing | Apply the second test? | **yes — 3,500:1 (HRSA primary care)** · 3,000:1 (high need) · off (travel only) | `access-engine.ratioMax`, the `thin` verdict and the second curve. **Off means the app reports one of the two tests every framework applies, and says so** |
| 6 · Exception | Model an alternative access standard? | **yes — as a labelled scenario (+31.61 mi, the documented CA average)** · enter the actual filing · no | the ghost rule, the amber "covered on paper" class, and the packet's exception section |
| 6 · Origins | Where are the people measured from? | **population-weighted centroids** · block-group centroids · tract internal points (**badged as unreliable**) | `access-engine.originPolicy`. **This is the app's single most consequential setting and the wizard says so** (§4 cmd 13) |
| 7 · Travel | How is travel measured? | **road network, car (OSRM)** · straight-line rings · bring your own matrix | `access-engine.engine`; rings degrade the loop honestly and are labelled "not the standard's measure" |
| 8 · Supply | Whose supply set? | **all licensed sites in the service line** · a plan's contracted roster (CSV upload) · add candidate sites | the `facilities` binding and the `FileDataSource` |
| 9 · Missing | What if no site is reachable? | **flag the origin as unmeasured** · score it at the maximum · exclude it | `access-engine.missingPolicy`. The default refuses to invent a number |
| 10 · Closure | Include the closure page? | **yes — model removing a facility or a service line** · skip | the `closure` page and its `views` tabs (the SB 1300 workflow) |
| 11 · Output | Packet, theme & language | **hearing-packet PDF + per-MSSA atlas**; **light (default)** · dark; **EN** · EN+AR | wires `/export report` + atlas; `ThemeSpec.mode`; `lang-switch` locales |

**Then:** Claude echoes *"Access to Care · Trinity County · primary care (adult) · Medi-Cal 10 mi /
30 min · AAS scenario +31.6 mi · population-weighted origins · OSRM road network · all licensed sites
· unmeasured flagged · closure page on · packet + atlas · light · EN"* and, on confirmation, runs §5
— so the app opens **fully configured**.

## 5. Prompt-script (run in order)

```
A. /new-app — an "Access to Care — Who's Outside" open-design app ("coverage-curve"): three pages
   (coverage / cohort / closure), structured LIGHT theme (primary #1d4ed8, secondary #7c3aed,
   success/warning/danger per §2.2, kpi override for tabular numerals), app-shell header (title +
   county picker + service-line picker + standard picker + [Hearing packet] + theme switch + lang
   switch + page-nav), footer carrying the "geographic screen - not a network-adequacy
   certification" line, and a splash (once:true) that teaches the threshold idea in one sentence.
   EN + AR/RTL via @strata/i18n with logical CSS properties throughout. Install deps + run command.

B. /add-data the verified layers from §3: HCAI Current_Healthcare_Facility_Listing (facilities -
   request outSR=4326, page at 1000, build points from LATITUDE/LONGITUDE), HRSA
   Health_Center_Service_Delivery_and_LookAlike_Sites filtered Site_State_Abbreviation='CA' (fqhc),
   HCAI MSSA2024_Tract (origins), HCAI MSSA_Tracts_Demographics (mssa-demog - bind the PROVERTY
   typo verbatim), CDC PLACES Tracts layer 3 filtered StateAbbr='CA' (places-tracts), Cal OES
   CA_CensusTracts_2023 (enriched-tracts), HRSA HPSA primary-care designation boundaries filtered
   PriStNM='California' (hpsa-pc), HCAI PCSA (pcsa), Caltrans CA_HQ_Transit_Stops (hqta-stops),
   CAL FIRE California_Incorporated_City_Boundaries (cities), CA_Counties (counties). Set the
   initial extent to the chosen county. Normalise COUNTY_NAME ('Trinity') against COUNTY
   ('Trinity County') before any join.

C. /symbology + /popup - genuine ESRI drawingInfo/popupInfo on verified fields only.
   Origins: uniqueValue on the computed `verdict` (inside green / covered-as-granted amber /
   outside red / unmeasured hatched), low-alpha fill (~40/255), graduated by TRACK_POP.
   Facilities: uniqueValue on FACILITY_LEVEL_DESC via Arcade, larger symbol where
   ER_SERVICE_LEVEL_DESC <> 'None'. HPSA: light hatch graduated on HpsScore (1-25; remember dental
   runs to 26). PCSA: outline only, labelled with its 2020-01-30 vintage.
   Popups: origin popup shows people, drive minutes AND miles, the verdict, the nearest open site by
   name, and the standard with its authority; facility popup shows FACILITY_NAME /
   FACILITY_LEVEL_DESC / ER_SERVICE_LEVEL_DESC / TOTAL_NUMBER_BEDS / FACILITY_STATUS_DESC WITH the
   licence vintage; tract popup shows ACCESS2_CrudePrev WITH its 95% CI, never bare.
   ⚠ Do NOT bind BIRTHING_FACILITY_FLAG as current availability - render it only behind a
   "2021 licence snapshot - verify with the hospital" badge.

D. /analyze - the numbers this app exists for. Population-weight the origins FIRST (block-level
   weighting, or block-group origins) - this is load-bearing, see §4 cmd 13. Then resolve the SECOND
   test: join each origin SPATIALLY to the HCAI PCSA layer and read Provider_R / EST_Provid. Treat
   Provider_R = 0 as INFINITE (no providers at all), never as a pass - 30 MSSAs are in that state.
   Gate at 3,500:1 (HRSA primary care) and emit a `thin` verdict for origins inside on travel and
   over the ratio. An origin with no published ratio stays inside and is counted as ratio-unknown. Then: `nearest`
   (nearest open site per origin, per service line), `buffer` for the miles leg of the standard,
   `pointsWithin` for sites in band, `overlay` + `aggregate` to roll tracts to MSSA and county and
   to intersect the HPSA/PCSA polygons with the origins. These ARE the day-1 fallback for
   `access-engine` - build them whether or not the widget ships. Use CAST(AREA_SQMI AS FLOAT)
   anywhere that column is read.

E. /panel statistics as the KPI row: People outside the standard (stat TRACK_POP sum over the
   outside set), Worst place (stat travel_minutes max), Uninsured 18-64 (stat ACCESS2_CrudePrev
   avg), Aged 65+ (stat PERC_SENIOR_CY avg), Deficit share (gauge, thresholds 10/35, invertColors),
   plus the provenance chip. Bind with dataSource stat + fromWidget:"access" / sourceId so they
   update with NO connections. The chip must say "as written" or "as granted - scenario" and an
   origin with no reachable site must render "unmeasured", never a large travel time.

F. /panel chart + /panel table - the equity comparison (county vs the outside cohort, population-
   weighted, on PERCENT_10 / PERCENT_20 / AVERAGE_EC / PERCENT_HI from 2025_MSSAs_from_HCAI - NOT
   from MSSA_Tracts_Demographics, whose PERCENT_* columns are Integer and truncate every value to 0),
   cohortbars (stacked-bar, horizontal, population by travel band, beyond-threshold bands in danger), cohorts (AttributeTablePanel on MSSA_NAME / DEFINITION /
   TRACK_POP / travel_minutes / travel_miles / verdict, virtualized, row -> zoom + flash,
   CSV/GeoJSON export), and the cohort-page profile on PERCENT_100_FEDERAL_PROVERTY_LE [sic],
   PERCENT_200_FEDERAL_PROVERTY_LE, AVERAGE_ECONOMIC_HARDSHIP_INDEX and PERC_SENIOR_CY.
   ⚠ Do NOT use the declarative `chart` widget anywhere - ChartPanel throws at render without an
   injected onQueryData.

G. WIF: author AppLayout.connections - the 18 wires in §2.7. Verify each emitter is shipped
   (table->rowSelect, filter->filterChange, map->featureSelect/extentChange, output->recordsChange,
   and the two new widgets' rangeSelect/categorySelect/recordsChange); do NOT wire buttonClick/
   timer/mapClick/sketchComplete, whose emitters are Phase-2 pending. Register the two §2.6 widgets
   app-locally via <StrataApp registry={{ "coverage-curve": ..., "access-engine": ... }}>; ship each
   one's named fallback in the same layout so the app is complete either way.

H. Accessibility to WCAG 2.2 AA, which is not optional for a public-hearing deliverable: the
   threshold is a role="slider" with aria-valuemin/max/now/text, operable by arrow keys (1 min),
   Page Up/Down (5) and Home/End, with the instruction visible on screen; the headline is announced
   in an aria-live="polite" region; NO verdict is carried by colour alone - each also has a glyph and
   a word in the list, and the map repeats it in ring weight; every select carries a VISIBLE label
   (not just aria-label, which would trip 2.5.3); page nav is a <nav> with aria-current="page";
   :focus-visible rings on everything; prefers-reduced-motion honoured, including the map's flyTo.

I. Controls + export: navigation/scale/geolocate; put the LayerPanel and BasemapPanel on the map as
   bottom-right controls ABOVE the zoom cluster (MapLibre prepends in bottom corners - add
   navigation first), with per-layer show/hide/remove; Legend, Measure + Draw (both revert to
   identify), search (Nominatim) -> "am I outside?", and a status bar. Re-paint the ink-coloured
   layers on every light<->dark basemap flip (MapLibre paint cannot read CSS variables). Wire
   /export report as the hearing packet (the curve + the threshold and its authority + the outside
   count + the cohort table with demographics and 95% CIs + the map + the measured route + engine,
   OSM vintage and layer vintage per number), a per-MSSA atlas, /export image, and a share
   deep-link with setUrlParam on county, service line, standard and the dragged threshold so a
   packet is reproducible from its URL.
```

## 6. Verify (benchmark: Quest Analytics QES · Esri Access to Care stack · UDS Mapper · HealthLandscape)

| Check | Pass | Evidence |
|---|---|---|
| **Origins are population-weighted, and the alternative is proven to lie** | ✅ | §4 cmd 13 — the same 3,879 people are **1.7 min** or **152.5 min** from Trinity Hospital depending on this setting; 267 tracts ≥ 100 sq mi hold 836,798 people |
| **`AREA_SQMI` is CAST everywhere**; no ranking, filter or symbology reads the raw string | ✅ | §4 cmd 5 — naive `orderBy` gives 99.21; the true max is 982.13; `> 100` errors, `CAST` returns 267 |
| **The MSSA join is spatial, never by key**, and both vintages are printed | ✅ | §4 cmd 12 — 6 exact matches of 542; 31 after normalising; **0** name matches |
| **The birthing flag is never rendered as current availability** | ✅ | §4 cmd 2 — 2021-11-15 is the newest licence in the layer; the service line ships behind a verify badge |
| **The three HPSA layers are never row-counted against each other** | ✅ | §4 cmd 10–11 — 180 designations vs 2,363 + 1,131 components; `PriStNM` is the only state filter that works on primary care |
| Every `layerId` + field verified against the service — no invented field names | ✅ | §4 cmds 1–15; the misspelled `PROVERTY` columns bound verbatim |
| Model-based estimates carry their confidence interval | ✅ | `ACCESS2_Crude95CI` rendered on the curve and in the packet (§2.4) |
| Three tract universes are reconciled, not silently joined | ✅ | §4 cmds 3, 8, 9 — 9,106 / 9,070 / 9,107; missing tracts render "no estimate" |
| `AmbulanceServices` is read from **layer 1**; no response time is computed from it | ✅ | §4 cmd 15 |
| Facility geometry requested as `outSR=4326`; points built from LAT/LON | ✅ | §4 cmd 1 — served geometry is wkid 3857; 0 of 10,961 rows have a null LATITUDE |
| Basemaps keyless; routing keyless and CORS-open; everything EPSG:4326 | ✅ | §4 cmds 14, 16 — OSRM returns `Access-Control-Allow-Origin: *`; Valhalla 200 |
| The app **never claims network-adequacy compliance** — the scope line is in the splash, the footer and the packet | ✅ | §Scope, §2.1, §5 A |
| Silhouette is the **coverage curve with a statutory rule on it**; distinct at a glance from every Health sibling (chart-board · triage-console · scoreboard · ops-command · scroll-story) | ✅ | `DESIGN-PROPOSAL.md` §2 anti-collision; no sibling in any sector navigates by a threshold on a distribution |
| **The signature loop works end-to-end**: drag the rule → KPI, map, cohort list and evidence move together | ✅ | `test-render.mjs` §6 — 35.1% at 30 min → 1.2% at 60 min, recomputed from the shipped `render()` |
| **The closure page moves the curve by the right amount, and names who moves** | ✅ | `drive-access.mjs` §7 — Trinity **+6,977 people**, four named origins; Modoc **+1,018**, Cedarville 1.2 → 39.2 min |
| **The exception is visibly different from compliance** | ✅ | `drive-access.mjs` §6 — 4,332 outside → 0 outside / 4,332 *covered on paper* at unchanged drive times |
| **The urban regime is not measured with the rural rule** | ✅ | `test-access.mjs` §11 — 342 of 364 Sacramento origins resolve as "tract lies inside a settlement" |
| **No silent caps** | ✅ | the cap is disclosed in the chip, the evidence strip and the packet, with a "measure all" button |
| Every panel renders, not just the ones you look at | ✅ | `test-render.mjs` §3–5 — a real bug found and fixed: the legend was gated behind the map style |
| **Launches dark with no light flash**; light is the print/counter mode, and both repaint the map ink | ✅ | `test-render.mjs` §7 — no theme class on `<body>` (nothing to flash), basemap paired both ways, `mapInk()` flips |
| **Both tests are applied**, not just the geographic one | ✅ | `test-access.mjs` §16 — the 3,500:1 HRSA gate; every CA MSSA over it is already state-designated; 111 of 542 fail it, holding 9.4% of Californians |
| **A service area with ZERO providers fails the ratio test** | ✅ | `test-access.mjs` §16 — `Provider_R = 0` on 30 MSSAs maps to Infinity, not to a pass; flagged separately in the KPI and the packet |
| **The equity profile comes from a layer whose numbers survive their column type** | ✅ | `test-access.mjs` §17 — the obvious layer's poverty column averages **0.0076** statewide; the app binds the 2025 sibling and asserts the true tract population |
| **Equity is population-weighted and reports its own coverage** | ✅ | `test-access.mjs` §18 — missing values skipped rather than counted as zero; empty cohort returns null, not NaN |
| **Dragging or arrowing the rule repaints the MAP, not just the table** | ✅ | `test-render.mjs` §13 — the map's origins source is asserted to match `verdictOfRow` for every row, before and after a move. **A shipped bug, found by the user:** a “light” render skipped `paintMap()` entirely, so the cohort list re-coloured while the map kept the old verdicts — two different answers to the same question on one screen |
| **The header is app-shell chrome on the manifest's tokens**, every control visibly labelled | ✅ | `test-render.mjs` §12 — three labelled regions, `--strata-space/radius/elevation/motion` + type steps, colour roles aliased for the port, `<nav>` + `aria-current`, phone disclosure |
| **WCAG 2.2 AA**: keyboard-operable threshold, live-announced deficit, no colour-only encoding, named controls, optional motion | ✅ | `test-render.mjs` §11 — 14 assertions incl. an actual arrow-key press moving the rule |
| **Every light surface carries dark ink** — including the ones MapLibre owns | ✅ | `test-render.mjs` §8 — popup re-themed, attribution + scalebar pinned to `--ink-on-light`, `option` lists explicit, print forces light tokens from either theme |
| ≥3 `connections` fire on first render (design ships 18); nine widgets link via `fromWidget`/`sourceId` with none | ⛏ | applies to the §5 `<StrataApp>` build; the demo wires the same loop imperatively |
| `responsive.small` collapses every side-by-side row; the curve is legible as the phone page | ⛏ | CSS breakpoint written (860 px); not browser-verified — no browser automation in this environment |
| Ink-coloured layers re-paint on a light↔dark basemap flip | ⚠ partial | implemented (`setStyle` → `styledata` → `buildLayers` + `paintMap`); not browser-verified |
| AR/RTL mirrors the curve, its rules and the cohort list as a unit | ⛏ | the demo ships **EN only**; the string table and `lang-switch` belong to the `<StrataApp>` build |
| Hearing packet exports with curve + threshold + authority + cohort + CIs + vintages; per-MSSA atlas paginates | ⚠ partial | the packet renders and prints (`test-render.mjs` §5); **no batch atlas in the demo** |
| Runs on Strata **and** ArcGIS; the whole screening path is read-only (writes 🔶 behind `assertEsriBackend`) | ⛏ | the demo is read-only by construction; backend parity belongs to the §5 build |
| The verdict uses **distance OR time**, as the standard is written — not AND | ✅ | §4 arithmetic block; using AND would over-report the deficit |
| The per-county alternative thresholds are bundled | ❌ | **DHCS Attachment B is a PDF and `dhcs.ca.gov` 403s a server-side fetch** (§4 cmd 17); statewide defaults ship with their authority printed, and the wizard offers a custom threshold |
| A no-vehicle household variable is bound | ❌ | ACS `B08201` redirected on a keyless call (§3); until wired, **every travel number says "car assumed"** |
| An AAS docket is bound as data | ❌ | DHCS publishes no machine-readable exception register; the ghost rule is a **labelled scenario** or a user-supplied filing |

**On-par-or-better:** matches the geographic core of what the incumbent charges for — time and
distance on the road network, per service line, against a stated standard — on open, keyless data,
while adding the two things none of Quest Analytics, Esri's Access to Care stack, UDS Mapper or
HealthLandscape ships: **the population distribution around the threshold** (so you can see how much
is sitting just outside), and **the exception case rendered beside the rule** (so "compliant" and
"covered on paper" are visibly different states). It also reaches the personas the incumbents do not
sell to at all — the county health officer, the health-centre planner and the advocate on the other
side of the filing. **Honest gap:** no certification, no contracted-network view without an upload,
no appointment availability or wait times, no service-line status (nobody has it), no ambulance
response modelling, no vehicle-availability variable, and no transit-routed travel time.

## 7. Harvest (gaps → strata-core)

Log as strata-core issues: a **`coverage-curve`** widget (a weighted cumulative distribution with
labelled, draggable thresholds and a confidence band — **this design's harvest candidate, and a
numbered template `threshold-curve` if it earns reuse twice**; it generalises immediately to
emissions limits, utility response-time standards, service guarantees and wait-time targets); an
**`access-engine`** / travel-matrix widget (origins × destinations → nearest → verdict, with a
pluggable engine — the reusable "how far is everyone from the nearest X" primitive that every sector
re-invents); a **population-weighted-centroid analyze op** (`popCentroid(polygons, weightSource)` —
the missing prerequisite for every honest accessibility measure, and the one this recipe proves is
load-bearing); a **travel-matrix batching helper** honouring per-engine coordinate caps; a
**typed-field coercion helper** for the String-typed-numeric ESRI column (this recipe needed `CAST`
for `AREA_SQMI`, site-selection for `AHEAD_AADT`, aviation `TRIM()` — **three recipes, one missing
"field hygiene" layer**); and a **vintage/provenance popup element** (value + source + as-of +
confidence interval, rendered consistently). Bigger asks: **transit-routed travel time** (blocked on
a GTFS routing engine, not on code) and a **2SFCA/E2SFCA analyze op** for teams that want the
academic index beside the regulatory threshold.

## 8. Sources

**Market & competitive (fetched 2026-07-29)**
- Quest Analytics — [Network adequacy & provider network analysis](https://questanalytics.com/how-we-help/qes/adequacy/) ·
  [Quest Enterprise Services (QES)](https://questanalytics.com/how-we-help/qes/) ·
  [Provider data management for payers & health plans](https://questanalytics.com/who-we-help/payers-plans/) ·
  [CMS renews the network-adequacy contract (Jan 2026)](https://questanalytics.com/news/quest-analytics-centers-for-medicare-and-medicaid-services-cms-renew-contract-measure-network-adequacy/)
- Compliance tier — [How health plans can meet 2026 network adequacy requirements (Verisys)](https://verisys.com/blog/how-health-plans-can-meet-network-adequacy-standards/) ·
  [Network adequacy 2026: Medicare & Medicaid requirements (Atlas Systems)](https://www.atlassystems.com/blog/network-adequacy-requirements-2026)
- Esri — [Solutions for access to health care](https://www.esri.com/en-us/industries/health/focus-areas/access-to-care) ·
  [To survey health-care adequacy, ArcGIS GeoAnalytics Engine simplifies analysis (ArcNews, Winter 2025)](https://www.esri.com/about/newsroom/arcnews/to-survey-health-care-adequacy-arcgis-geoanalytics-engine-simplifies-analysis) ·
  [OD cost matrix (ArcGIS Pro)](https://pro.arcgis.com/en/pro-app/latest/help/analysis/networks/od-cost-matrix-tutorial.htm)
- Public / non-profit tier — [UDS Mapper (HealthLandscape)](https://healthlandscape.org/portfolio/uds-mapper/) ·
  [HealthLandscape](https://healthlandscape.org/) · [PolicyMap — HRSA data](https://www.policymap.com/data/sources/hrsa) ·
  [Mapping tools (Build Healthy Places Network)](https://buildhealthyplaces.org/tools-resources/measure-up/mapping-tools/)
- Method — [An enhanced two-step floating catchment area (E2SFCA) method — Luo & Qi](https://pubmed.ncbi.nlm.nih.gov/19576837/) ·
  [Spatial accessibility of primary health care utilising the 2SFCA method (Int. J. Health Geographics)](https://link.springer.com/article/10.1186/1476-072X-11-50) ·
  [Evaluation of access to health care in rural areas using E2SFCA (J. Transport Geography)](https://www.sciencedirect.com/science/article/abs/pii/S0966692316304495) ·
  [A geoprocessing toolbox for spatial accessibility analysis (JMIR Formative Research, 2024)](https://formative.jmir.org/2024/1/e51727)

**Standards & designation**
- HRSA — [What is shortage designation?](https://bhw.hrsa.gov/workforce-shortage-areas/shortage-designation) ·
  [Scoring shortage designations](https://bhw.hrsa.gov/workforce-shortage-areas/shortage-designation/scoring) ·
  [Health workforce shortage areas (data.hrsa.gov)](https://data.hrsa.gov/topics/health-workforce/shortage-areas) ·
  [HPSA score (NHSC)](https://nhsc.hrsa.gov/scholarships/requirements-compliance/jobs-and-site-search/hpsa-score-class-year)
- California — [DHCS network adequacy](https://www.dhcs.ca.gov/forms-laws-publications/network-adequacy/) ·
  [DHCS Attachment B — time and distance standards](https://www.dhcs.ca.gov/wp-content/uploads/2025/10/Attachment-B-Time-and-Distance-Standards.pdf) ·
  [APL 23-001](https://www.dhcs.ca.gov/formsandpubs/Documents/MMCDAPLsandPolicyLetters/APL2023/APL23-001.pdf) ·
  [Medi-Cal managed care time and distance standards (Disability Rights California)](https://www.disabilityrightsca.org/publications/medi-cal-managed-care-time-and-distance-standards-for-providers) ·
  [DMHC timely access to care](https://www.dmhc.ca.gov/Portals/0/Docs/DO/TAC_accessible.pdf) ·
  [28 CCR §1300.67.2.2 (Cornell LII)](https://www.law.cornell.edu/regulations/california/28-CCR-1300.67.2.2) ·
  [Network adequacy in Medi-Cal managed care (NHeLP, 2024)](https://healthconsumer.org/wp/wp-content/uploads/2016/10/Network-Adequacy-in-Medi-Cal_NHelP_2024.pdf)
- **The exception finding** — [Exceptions to network adequacy rules may exacerbate health disparities in Medi-Cal managed care (National Health Law Program, 31 Jul 2019)](https://healthlaw.org/exceptions-to-network-adequacy-rules-may-exacerbate-health-disparities-in-medi-cal-managed-care/)

**The closure story (why the `closure` page exists)**
- [SB 1300 — health facility closure: public notice (leginfo)](https://leginfo.legislature.ca.gov/faces/billNavClient.xhtml?bill_id=202320240SB1300) ·
  [Sen. Cortese on SB 1300](https://sd15.senate.ca.gov/news/sen-corteses-statement-sb-1300-closure-inpatient-psychiatric-and-maternity-ward-services) ·
  [New California law mandates notice for hospital unit closures (Becker's)](https://www.beckershospitalreview.com/legal-regulatory-issues/new-california-law-mandates-notice-for-hospital-unit-closures/)
- [No deliveries: maternity care deserts in California (CalMatters series)](https://calmatters.org/series/no-deliveries-maternity-care/) ·
  [Maternity care deserts: can CA lawmakers keep labor wards open? (CalMatters)](https://calmatters.org/health/2024/04/maternity-care-deserts/) ·
  [Corona Regional Medical Center closing labor and delivery ward in 2026 (CBS LA)](https://www.cbsnews.com/losangeles/news/corona-regional-medical-center-closing-labor-and-delivery-ward) ·
  [Nowhere to go: maternity care deserts across the US, 2024 (March of Dimes)](https://www.marchofdimes.org/sites/default/files/2024-09/2024_MoD_MCD_Report.pdf) ·
  [Where you live matters: maternity care access in California (March of Dimes PeriStats)](https://www.marchofdimes.org/peristats/reports/california/maternity-care-deserts)

**Data**
- HCAI — [Healthcare facility attributes](https://hcai.ca.gov/data/data-resources/healthcare-facility-attributes/) ·
  [Licensed and certified healthcare facility listing (CHHS Open Data)](https://data.chhs.ca.gov/dataset/healthcare-facility-locations) ·
  ArcGIS org `services5.arcgis.com/fMBfBrOnc6OOzh7V` (MSSA, PCSA, facilities, FQHC, HPSA points)
- HRSA HPSA boundaries — Federal AGOL org `services2.arcgis.com/FiaPA4ga0iQKduv3`
- CDC — [PLACES: local data for better health](https://services3.arcgis.com/ZvidGQkLaDJxRSJ2/arcgis/rest/services/PLACES_LocalData_for_BetterHealth/FeatureServer)
- Routing — [Project OSRM](https://router.project-osrm.org) · [Valhalla (FOSSGIS/OSM Germany)](https://valhalla1.openstreetmap.de)

**Internal** — `DESIGN-PROPOSAL.md` · `build_access_catalog.py` → `access-catalog-ca.json` ·
`../APP-TEMPLATE-LIBRARY.md` (assignment: `open-design` "coverage-curve"; `nearby-finder` released) ·
`../../data_sources/data_sources_ca.md` · `strata/recipes/COMPONENT-MANIFEST.md` (§8 modernization,
§10 freestyle charter) · `strata/docs/guide/app-design.md` ·
`strata/docs/reference/human-language.md`

---

## Modernization (parity release)

> Applies the Experience-Builder-parity features (Phases 1–7). See `strata/recipes/MODERNIZATION.md`
> + `COMPONENT-MANIFEST.md` §8. Cross-cutting: a structured **`theme`**, app-shell
> (`header`/`footer`/`splash`), and `dataSource.{ sourceId }` linking.

- **Source linking is the architecture, not a garnish:** one `access` output feeds `curve`,
  `kpi-outside`, `kpi-worst`, `gauge-deficit`, `cohorts`, `cohortbars` and `evidence` through
  `dataSource.fromWidget` — **seven widgets, zero `connections`** — and `kpi-unins`/`kpi-senior` bind
  to the shared `places-tracts`/`enriched-tracts` sources so the `filter` → `store.setDefinition`
  path updates them for free. **Nine of the app's widgets are alive with no wiring at all.**
- **Layout nodes:** `splitter` ×2 (curve | evidence | strip, and map | cohort list — the user decides
  how much argument vs. how much map), `panel` (the evidence strip, dock-bottom → **dock-bottom sheet
  on phones** for the pickers), **`window`** (the hearing packet, `open:false`, opened by `showHide`;
  and the catalog drawer), `views` + `mapState` (the `closure` page's before / after / who-moves
  tabs each re-filter the facilities layer and fly the map), `accordion` (the phone collapse),
  `section` `mode:"fixed"` (the map box), `grid` (the phone KPI grid).
- **Data-source kinds:** **`FileDataSource`** is load-bearing in a way it is not in most siblings —
  the candidate-site list, the closure list **and a plan's own contracted-provider roster** all
  arrive through it, which is how the app serves the plan persona at all. `RestDataSource` for
  `access-catalog-ca.json`, the OSRM matrix and the Valhalla contour. **No `StreamDataSource`** —
  nothing here ticks; a tract's population does not update on a poll.
- **Motion:** `fly` + `stagger` on the KPI row so a re-run **reads as a recomputation** rather than a
  repaint; `fade` on the curve section. Nothing on the map container.
- **Theme:** one structured `ThemeSpec` derives hover/active/focus states and the type scale, with
  the semantic roles carrying inside / covered-as-granted / outside; `theme-switch` swaps the default
  **light** public-document mode ⇄ a dark analyst mode and pairs the basemap to it; the on-map
  keyless gallery can then override that pairing. A `kpi` `overrides` block gives the numerals a
  tabular mono face.
- **i18n:** EN + AR/RTL via `@strata/i18n` + `lang-switch`, logical CSS properties throughout — the
  curve, its rules and the cohort list mirror as a unit.
- **Writes 🔶:** a county's own verification note on a place ("confirmed with the hospital
  2026-08-01", "AAS contested") via `updateRecord` behind `assertEsriBackend`; everything else is
  read-only and works unchanged on Strata Serve.
