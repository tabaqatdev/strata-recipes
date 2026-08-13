# Recipe — Public Information Map (Local Government)

A reproducible path to a **citizen-facing public information map** on strata-core that answers the
question every municipal map viewer leaves out: *"there is a crew outside with a saw — what is it,
how long will it be there, and **is there anything I can still do about it?**"* It takes everything a
city publishes about an address and sorts it not by department or layer but by **the reader's own
recourse** — speak · object · plan around it · call someone · report it · nothing, and here is why —
giving each rung its own clock, and printing the rule and the statute beside every assignment.

> **Scope (honest).** An **informational projection of what a city already publishes** — not a system
> of record, not the notice of record, and not legal advice. The legal notice remains the mailed
> **Gov. Code §65091** letter, the §65090 newspaper publication and the posted agenda; this map
> *shows* them, it does not *give* them. **The `Object` rung is empty in the demo city** — Los Angeles
> publishes no machine-readable hearing calendar, and a `data.lacity.org` search for *hearing* or
> *notice* returns nothing relevant — so the app renders the empty rung and **names the absence**
> rather than filling it. **Agenda items are never geocoded**: §54954.2 caps an item description at a
> "brief general description" (~20 words), so a meeting is placed at its body's boundary or its
> meeting address, never at an address parsed out of a title. **The 300-foot ring is drawn from parcel
> geometry, not from the assessment roll** — the city publishes the parcels, not the owner names, so
> the app counts parcels inside the ring and never claims to know who was mailed. **Contacts are
> republished exactly as the city publishes them, including the 16.4 % that are blank.** **No write
> path at all**, so it runs identically on Strata and ArcGIS. Deploy **on-prem** where a city binds
> its own case system.

Run the §5 prompt-script on a fresh strata-core project (or via `/recipe
local-government_public-information-map`). ESRI Web Map JSON is the map contract; ingest is via
`/convert` → `/publish` to Strata Serve.

---

## 0. Provenance

| | |
|---|---|
| **Source** | `https://tabaqat.net/solutions` — **Local Government** section |
| **Name on site** | Public Information Map |
| **Tagline** | "What the city is doing at your address — and what you can still do about it" |
| **Researched** | 2026-08-09 |
| **Template** | **`open-design`** ("recourse-ladder") — see `DESIGN-PROPOSAL.md` §2 for the three candidates and the anti-collision check |
| **Sector status** | **First researched recipe in Local Government** (all 16 folders were empty), so nothing is inherited from the template library and nothing is released back |
| **Catalog** | `pim-catalog-ca.json` — 2,360 CA services inventoried, 250 role-tagged across 8 roles, 18 external feeds (16 verified-live). Rebuild with `python build_pim_catalog.py` |
| **Presentation** | `presentation/index.html` (12-slide deck, tabaqat house style) + `presentation/linkedin-article.md` (~980-word article + teaser + a claims note giving the source for every figure and an explicit do-not-claim list) |
| **Built** | **2026-08-09** — `app/` (self-contained, MapLibre from CDN; `node server.mjs` → :8037). **343 assertions green** across `test-standing.mjs` (234, live data) and `test-render.mjs` (109, boots the shipped page and drives every gesture), plus a **real-browser pass** (headless Chrome). See `app/README.md` |

---

## 1. Study — how the market frames this

**The question the buyer asks.** Two buyers, one counter apart. **The resident:** *"What is happening
outside my window, and can I do anything about it?"* **The city** — a PIO, a GIS manager, a Council
chief of staff: *"Can we answer that without a phone call, and can we show we told them?"*

Every product in this market answers the first half. The municipal viewers (**NavigateLA**, SF's
**Property Information Map**, **PortlandMaps**, NYC's **ZoLa**) are layer browsers organised by
department. The engagement platforms (**Granicus EngagementHQ**, **PublicInput**, **Social Pinpoint**,
**coUrbanize**, **Konveio**, **Decidim**) own the consultation but not the street. The permitting
portals (**Accela**, **Tyler EnerGov**, **OpenGov**, **CentralSquare**, and the aggregator
**AgencyCounter** — formerly **buildingeye**) own the record and publish it as a *status*, never as a
*deadline*. **Chicago Cityscape** is the sharpest thing in the category and is a private product
selling permit alerts to brokers and zoning attorneys — on the other side of the counter from the city.

### The finding this recipe is built on (verified live 2026-08-09, not asserted)

**California's public estate does not contain this app's subject.** The 8-role classifier in
`build_pim_catalog.py`, run over the **2,360** services in `../../data_sources/data_sources_ca.md`:

| role | layers in the crawled CA estate |
|---|---|
| `works-and-closures` — what is happening to your address | **0** |
| `decision-in-progress` — what is being decided about it | **0** |
| `notice-channel` — who you could tell | **0** |
| `service-request` — how you report it | **0** |
| context roles (`reporting-frame` 103 · `address-frame` 57 · `hazard-context` 54 · `service-entitlement` 36) | 250 role-tagged |

Structural, not an oversight: the catalogue inventories **state** agencies, and every fact this app
needs is held by one of California's **483 incorporated cities** — of which it holds **none**. Sixteen
services matched the vocabulary and were rejected *by hand, with reasons recorded in the build script*
(8 as CAL FIRE incident names — the **Park Fire** and the **District Fire**; 8 inspected and written
up, including DWR's permits-to-encroach-on-DWR's-own-levees and Caltrans' `D4_Right_of_Way_Boundary`,
which is a land interest **and returns `Token Required`**).

**So this recipe found a municipal server.** `https://maps.lacity.org/lahub` — City of Los Angeles
(NavigateLA / Bureau of Engineering), **35 services, CORS-open**, absent from `data_sources_ca.md`.
It publishes the **Public Way Reservation System**, the city's cross-agency registry of everything
that has reserved a piece of the public way. Measured live, fully counted:

| | points | lines | polygons | **total** |
|---|---|---|---|---|
| reservations | 11,283 | 7,843 | 43,180 | **62,306** |
| **in progress right now** | 10,259 | 3,037 | 29,778 | **43,074** |
| future / completed | 667 / 357 | 4,403 / 403 | 13,249 / 153 | 18,319 / 913 |
| **no contact email** | 4,259 | **5,897 (75.2 %)** | 37 | **10,193 (16.4 %)** |
| `EndDate` = 2099 sentinel | 124 | 228 | 187 | **539** |
| in progress, ending ≤ 30 d | 806 | 75 | 4,347 | 5,228 |

**Four facts, and the app is built on all four:**

1. **Not one of the 62,306 records carries a comment, objection, appeal or hearing date.** The
   24-field schema has `StartDate`, `EndDate`, `EnterDate` and `ActiveStatus` — and no field in which
   a right could be expressed. It is a **notification system with no reply address**.
2. **The reply address is missing exactly where it matters.** 16.4 % name no email — but **75.2 % of
   *line* features** do, against **0.09 %** of polygons. A line is a length of somebody's street.
   Whether a resident can find out who is digging depends on the **geometry type**, which no resident
   could guess.
3. **`EventType` and `StreetName` are blank on all 62,306 rows.** The two fields a person would search
   by are empty on every record — so a map is not the preferred interface here, it is the only
   possible one.
4. **Only 5,228 of 43,074 in-progress items end within 30 days** (12,835 within 90), and 539 run to
   **2099**. `ActiveStatus` is derived honestly — zero rows are "In Progress" with a past end date —
   but it inherits the sentinels, so a large share of what the map calls *now* has no foreseeable end.

**And the counterweight, on the same server:** LA's **99 certified Neighborhood Councils** — every one
a Brown Act body — publish **99/99 phone, 99/99 website, 98/99 email**; the **15 Council Districts**
name the sitting member. *The mechanism for speaking is fully attributed. The record of what is being
done to you is not.* **That asymmetry is the product.**

### Why the gap exists — the statutory frame

- **Gov. Code §65091(a)(3)** — hearing notice is *"mailed or delivered at least 10 days prior … to all
  owners of real property as shown on the latest equalized assessment roll within 300 feet."* **The
  mailbox is the interface and it is addressed to owners**; the section makes **no provision for
  occupants or tenants**. In Los Angeles County **54.1 %** of occupied units are renter-occupied (ACS
  B25003 via Esri Living Atlas, `GEOID='06037'`, measured 2026-08-09).
- **And it stops at the top:** above **1,000** owners an agency may substitute **a single ⅛-page
  newspaper advertisement** — individualised notice ends precisely at the largest projects.
- **One clock always exists: Gov. Code §54954.2(a)(1)** — the agenda is posted **at least 72 hours**
  before a regular meeting, *"on the local agency's internet website"*, with a *"brief general
  description"* (~20 words) of each item; §54954.3 carries the right to address the body. This binds
  every legislative body without exception, including all 99 Neighborhood Councils — which is why this
  app's top rung is built on it rather than on any one city's case system.

A public information map is therefore **the channel the statute does not provide.** That is the
sentence a city buys it on.

### Reference solutions (benchmark + coexist, never copy)

- **Esri** — [My Neighborhood Services](https://solutions.arcgis.com/local-government/help/my-neighborhood-services/)
  (the closest named analogue: a public app to find facilities and services by address) ·
  [Capital Project Coordination](https://doc.arcgis.com/en/arcgis-solutions/latest/reference/introduction-to-capital-project-coordination.htm)
  + its Capital Project Locator · ROW Permitting · Special Event Permitting ·
  [ArcGIS Hub](https://www.esri.com/arcgis-blog/products/constituent-engagement/constituent-engagement/arcgis-hub-for-community-engagement)
  as the front door. All organised **by department**; none by the reader's standing.
- **Municipal incumbents** — NavigateLA · SF Property Information Map · PortlandMaps · NYC ZoLa ·
  Shaping Seattle. Excellent at *what*; silent on *by when*.
- **Permit records & aggregation** — Accela Citizen Access · Tyler EnerGov · OpenGov · CentralSquare ·
  **[AgencyCounter](https://www.agencycounter.com/)** (ex-buildingeye), which unifies all four into one
  public portal — the clearest possible statement that this market's problem is fragmentation, from a
  vendor that solved the fragmentation and left the deadline out.
- **Civic alerting (non-Esri, the sharpest)** — **[Chicago Cityscape](https://www.chicagocityscape.com/)**
  (development map + daily permit-alert newsletters) · **PlanningAlerts** (OpenAustralia Foundation) ·
  BuildZoom.
- **Engagement** — Granicus (EngagementHQ, govDelivery) · PublicInput · CivicPlus · Social Pinpoint ·
  coUrbanize · Konveio · Decidim · Bang the Table.
- **311** — SeeClickFix (CivicPlus) · QScend QAlert · Granicus GORequest · Salesforce Public Sector.
- **Meetings** — Granicus/**Legistar** · CivicClerk · PrimeGov · BoardDocs. Legistar exposes the Brown
  Act clock as a keyless API and **no public information map uses it**.

### Our edge

**Nobody in this market sells the deadline, and most of them structurally cannot.** The permitting
vendors sell the system of record and cannot publish a right their customer has not configured; the
engagement platforms are only invited once a city has decided to consult; the alerting products sell
subscriptions over the city's own data and sit on the other side of the counter. **An open, MIT,
on-prem tool has no record to protect and no subscription to sell**, so it can put *"you cannot do
anything about this one, and here is why"* on the screen — the sentence every commercial product here
has a reason to omit. Plus the standing edges: AI-authored, runs on **Strata *or* ArcGIS**, sovereign/
on-prem, cross-widget interactivity on the first build, ESRI Web Map JSON in and out.

### Standards, programmes & organizations to speak fluently

- **Ralph M. Brown Act** (Gov. Code §54950 et seq.) — **§54954.2** 72-hour agenda · **§54954.3** right
  to address · **§54956** 24 h for a special meeting.
- **Planning & Zoning Law** — **§65090** (newspaper publication, 10 days) · **§65091** (300-ft mailed
  notice to owners; ⅛-page substitution above 1,000 owners) · §65094 (definition of notice).
- **CEQA** — NOP / NOI / NOD, and the fact that a **ministerial** approval has none.
- **Ministerial by-right approvals** — **SB 9**, **SB 35 / SB 423**, **AB 2011**, ADU law: a growing
  class of housing approvals with **no discretionary hearing and therefore no §65091 notice at all**.
  A public information map that only shows hearings is silent on the fastest-growing category — the
  app's rung ⑥ is where these land, labelled.
- **AB 2234** — post-entitlement permit transparency and online checklists.
- **California Public Records Act** (Gov. Code §7920 et seq.) — the fallback when the map has nothing.
- **ADA / WCAG 2.1 AA · Gov. Code §7405** — a public app carries an accessibility obligation.
- **Orgs:** the City Clerk · Bureau of Engineering · **EmpowerLA** (the 99 Neighborhood Councils) ·
  Caltrans districts · the regional COG · GovLoop / ELGL / Code for America.
- **ESRI Web Map / `drawingInfo` / `popupInfo` JSON** — this repo's rendering contract.

**Honest scope** — see the blockquote at the top; repeated in the app's `splash` and footer.

---

## 2. UI design spec (front-loaded)

### 2.1 Layout (Template: `open-design` — "recourse-ladder")

- **Template `open-design`** under the freestyle charter (`strata/recipes/COMPONENT-MANIFEST.md` §10).
  Full derivation, three candidate silhouettes and the anti-collision check: **`DESIGN-PROPOSAL.md`**.
- **Why not a library template:** the library's `sidebar-explorer` is literally described as "the
  public information map", and it is already assigned to
  `emergency-management_public-crisis-information-map`. More to the point, a legend-as-filter sidebar
  organises by **layer**, and the subject of this app is **the reader's standing**, which no layer
  expresses. Local Government has no prior assignments, so the check ran across every other sector.
- **The silhouette:** a **six-rung ladder ordered by what the reader can still do** is the interface —
  *Speak · Object · Plan around it · Call someone · Report it · Nothing, and here is why.* Each rung
  carries a live count, a **clock** (soonest deadline on that rung) and a **reachability bar** (share
  of its items naming a human). Empty rungs render **their reason**, not nothing.
- **Signature loop:** **type an address → the ladder assembles → click a rung → the whole app adopts
  it** — the map repaints to exactly those items, the answer card rewrites with the action and the
  statute, the item table re-scopes, the URL updates.
- **Two grips on one rule.** **Address mode** is the resident's (one address, the 300-ft ring).
  **Area mode** is the *staff console* — a PIO, a council chief of staff and a neighborhood-council
  board member have a **district**, not an address, so the same rung rule runs over a whole council
  district (15) or neighborhood council (99). CD13 over 90 days is **4,302 items**. The area boundary
  is fetched generalised and sent as the query geometry (§3 *query capabilities*), which costs a
  ±~55 m boundary error the app discloses in the answer rail and repeats in every export.
- **The working surface** (area mode earns these; address mode gets them too): **filter chips** for
  every narrowing (scope · horizon · rung · kind · agency · quick · pinned), each removable — *a
  filter you cannot see is a bug*; a **"who is doing it"** agency list carrying each agency's
  unreachable count; **three quick filters** — *ends within 7 days · names nobody · no end date*;
  a **sortable table** (5 keys, both directions, `aria-sort`); **hover a row → the item lights up on
  the map**; **↑ ↓ / Enter / P** to walk, select and pin without a mouse; and a **pin tray** →
  *copy all emails · export packet · zoom to pinned*.
- **Resizable panels.** Three drag seams — ladder ⇄ map, map ⇄ answer, and the table's height — each
  an ARIA `separator`, keyboard-operable (arrows, ⇧ for a bigger step) and double-click-to-reset,
  calling `map.resize()` so the canvas never keeps a stale size. A fixed rail decides for the reader
  which pane matters; a staffer working the table wants it tall and a resident reading the ladder
  wants it wide.
- **Map chrome — one control vocabulary, on the map.** MapLibre's own cluster is hidden and replaced
  by a single top-right stack of 32 px inline-SVG buttons — zoom in · zoom out · **fit** · **L**ayers ·
  **B**asemap · Le**g**end — with drawers opening *beside* it, never over it, and `L`/`B`/`G`
  shortcuts. The basemap drawer shows **a real tile of the current location in each style**, because
  a colour swatch cannot tell Positron from Voyager. Identical vocabulary to
  `marketing_catchment-and-market-share-analyzer` — a suite of apps should not each invent its own map
  furniture.
- **Escape is priority-ordered:** splash → open drawer → selection → adopted rung. Escaping past a
  modal to clear a filter behind it is the wrong instinct. The splash also dismisses on a backdrop
  click and says "or press Esc".
- **Signature accent:** the **300-foot statutory ring**, drawn as a **stroke only, never a fill**, so
  it can never be misread as a hazard zone — with the count of parcels inside it and the sentence
  that the statute addresses their **owners**.
- **Wiring floor:** **14 authored `connections`** (the guideline's floor is 3), plus **9 widgets
  linked by a shared `sourceId` with no wiring at all**.

```
┌ HEADER ── Public Information Map ····· City of Los Angeles ·············································┐
│ [Address│Area]  [ 🔍 6801 Hollywood Blvd ]  within [300ft§│500│1000]  next [30d│90d│18mo│all]  [Print][☀] │
│        area mode swaps the address box for:  [Council district ▾] [13 - …  ▾]                           │
├ THE LADDER ─ 340 px ─────────┬─ MAP (dominant, full-bleed) ──────────┬─ THE ANSWER ─ 380 px ───────────┤
│ WHAT CAN I DO ABOUT IT?      │                        ┌─┐ + − ⌂      │  ┌───────────────────────────┐  │
│ ① SPEAK              3  ●    │     ╭──── 300 ft ────╮ │+│ − ⌂       │  │  4 things · 1 door open   │  │
│   ████████████ 100 % reach   │    ╱   ⌂ your address ╲│L│ B  G      │  │  next deadline: 2 days    │  │
│   ⏱ agenda due in 2 days     │   │    ▨ ▨      ▨      ││ │ ☰ G       │  └───────────────────────────┘  │
│   Arleta NC · Wed 6:30 pm    │    ╲    ▨   ━━━━━     ╱ └─┘           │  ① SPEAK — you may address      │
│ ② OBJECT             0  ○    │     ╰───────────────╯                 │     ARLETA NC, Wed 6:30 pm      │
│   ░░░░░░░░░░░░ no data       │                                       │     agenda due Sun 6:30 pm      │
│   ⚠ this city publishes no   │   ▨ work area  ━ street reservation   │     ☎ 213-978-1551 ✉ ANC@…      │
│     hearing calendar as data │   ● capital project ⬤ meeting body    │  ───────────────────────────    │
│ ③ PLAN AROUND        2  ●    │                                       │  THE DERIVATION — printed       │
│   ██████░░░░░░  50 % reach   │  ┌ LEGEND — every row filters ──────┐ │   §54954.2(a)(1): agenda ≥72 h  │
│   ⏱ starts in 14 days        │  │ Excavation permit      2  ◱      │ │   §65091: mail to OWNERS ≤300ft │
│ ④ CALL SOMEONE       2  ◐    │  │ Street resurfacing     1         │ │   ⚠ 54.1 % of homes here rent   │
│ ⑤ REPORT IT          —       │  │ Capital project        1         │ │  ───────────────────────────    │
│ ⑥ NOTHING — AND WHY  1  ○    │  └──────────────────────────────────┘ │  ▸ Your services (bin day: THU) │
│   names nobody, no end date  │                          EPSG:4326 ▸  │  ▸ What this map cannot show    │
├══════════════════════════════╧═══ ⇕ drag ════════════════════════════╧═════════════════════════════════┤
│ SHOWING  [Area 13 — …] [Next 90 days] [Rung Call someone ×] [Agency BOE ×] [Pinned 4 ×]                 │
│ QUICK    ( ends within 7 days 12 ) ( names nobody 3 ) ( no end date 14 )                                │
│ ITEMS ▲▼ what · who · when · who-to-call · rung · 📌   hover → map · ↑↓ walk · Enter select · P pin     │
│ TRAY   4 pinned · 1 distinct contact   [Copy all emails] [Export packet] [Zoom to pinned] [Clear]       │
└────────────────────────────────────────────────────────────────────────────────────────────────────────┘
   ▲ click a rung: the whole app adopts it     ⇕ all three seams drag   footer: sources · vintage · scope
```

**A second page, `dossier`** (`type:"scroll"`) is the printable address record — the property-report
silhouette, correctly demoted from the navigation to the artifact.

**Map chrome.** MapLibre's native cluster is hidden and replaced with one top-right cluster — zoom in ·
zoom out · **fit to the adopted rung** · **L**ayers · **B**asemap · Le**g**end — drawers opening beside
it, never over it. Same vocabulary as `industry_mining-and-concession-compliance` and
`telecom_underserved-gap-and-take-rate`.

**The legend is a control surface, not a caption.** Each row filters by **kind** with a live count.
Filtering by kind deliberately does **not** change the adopted rung, and a hint line says so.

**Responsive.** Below 900 px the three columns become one. **The ladder stays first and stays a
ladder** — full-width rows, never a dropdown, because a ladder that has become a picker is no longer
an argument about what you can do. The map drops to 45 vh, the answer card follows, the item table
goes last. The ring and the address pin survive at every width.

### 2.2 Theme

Structured `ThemeSpec`, **`mode: "light"`**. **One hue means one thing: `primary #1e6b52` (green) is
a door that is still open**; `warning #a4490a` is a clock running out; `secondary #475569` (slate) is
information you cannot act on; `danger #8a3324` is reserved for the single real failure state — a
record that names nobody and never ends. A reader learns the distinction from colour before reading a
label, and no element may use one to mean another.

**The light/dark call is measured, not asserted.** Contrast was computed for all six rung roles
against each theme's panel. Light's worst was the amber at **4.37:1** — under WCAG AA — while dark
cleared AA on everything (worst 5.34:1). That reads as an argument for dark until the distribution is
inspected: **every other light role sat at 5.40–7.08:1**, so the amber was the outlier, not the theme.
It was darkened `#b45309 → #a4490a` (**5.17:1** on panel, 5.89:1 on page) and **both modes now clear
AA on every rung**, which the suite asserts by computing the ratios rather than trusting the palette.
Light then stands on fitness for purpose: the audience is the public on a phone in daylight; the
artifact leaves as **paper** into a council packet and should be the same document on screen and on
the page; and the map is a **locator, not a choropleth**, so there is no dense surface a dark basemap
would flatter. Dark stays first-class via `theme-switch` — a council office does run this on a wall.

`fonts.scale "default"` · **mono override on `kpi` and `table`** with `tabular-nums` so a countdown in
days never jitters · motion **160 ms**, short, because every animation here is a deadline moving.

**Why light, deliberately:** the audience is **the public on a phone in daylight** (and
`app-design.md` §5 reserves light for public and civic apps); the artifact **leaves as paper** — the
printed address record goes into a Neighborhood Council packet or a Council office file, and it should
be the same document on screen and on the page; and **the map is a locator, not a choropleth**, so
there is no dense surface a dark basemap would flatter. **Dark stays first-class** via `theme-switch`
(a Council office does run this on a wall) and an explicitly chosen basemap survives a theme swap.

Basemap **keyless**: CARTO Positron (light) / Dark Matter (dark) from `OPEN_BASEMAPS`, paired via
`basemapForTheme`. Work-area fills alpha ≈ 40/255; the **300-ft ring is stroke-only**. **EPSG:4326
throughout** — every LA City service is `wkid 102100`, so `outSR=4326` on ingest.

**i18n — and here it is not decorative.** EN + **ES** is the pairing this audience needs, and
`@strata/i18n` ships `en`/`ar` only, so **an `es` dictionary is a required deliverable of this recipe**
(§7). AR/RTL is wired and works and is kept for the platform, but shipping AR and not ES for a Los
Angeles public map would be equity theatre.

### 2.3 KPI cards

Three, all live `kpi`/`gauge` `stat: {field, op}` bound to the shared `items` source, so they recompute
on every rung adoption **with no `connections`**:

**Things at this address** (`item_id`, count — the headline, with the ring radius beneath it) ·
**Next deadline** (`clock_days`, min, in days, amber) · and a radial `gauge` for **items naming a
human** (`has_contact`, avg; thresholds 50/85), which across LA's whole registry reads **83.6 %** and
across its line features **24.8 %**.

Two further metrics — **items with a comment period** and **items whose owners were mailed** — render
**greyed with an explicit reason** ("this city publishes no hearing calendar as data" / "the owner
roll is not published"), and turn live if a city binds its case export via `FileDataSource`.

### 2.4 Charts & table

- **The ladder** — the signature. A new `recourse-ladder` widget (§10.3 block in `DESIGN-PROPOSAL.md`
  §9) rendering six rungs with count + clock + reachability bar; **named fallback: a `table`** over the
  same `rungs` source whose `rowSelect` carries the identical payload, so **the signature loop survives
  unchanged**. **Ship the fallback on day one.** A `stacked-bar` (6 series) is the visual stand-in.
- **Item table** — `AttributeTablePanel` over `items`: `kind` · `agency` · `title` · `start` · `end` ·
  `contact_name` · `contact_phone` · `rung`. Sortable, per-column filter, server-paged, row → `zoomTo`
  + `flash`, CSV/GeoJSON export. **Rows with `has_contact == false` are tinted** — the absence is
  visible in the table too, not only in the gauge.
- **No chart rail.** A resident does not want a distribution of permit types, and charting this would
  make a civic notice look like analytics. The `chart-board` silhouette belongs to other recipes.

### 2.5 Capabilities to use (Phases 0–7)

- **`/analyze` at build time:** `buffer` (the 300-ft ring), `pointsWithin` (items ∩ ring),
  `spatialJoin` (items × council district × neighborhood council), `dissolve`/`aggregate` (the rung
  roll-up). **Not exposed as a widget** — an analysis panel on a public map invites a resident to
  produce a finding the city must then defend. **`hexbin`/`hotspot` rejected** — density answers
  "where is there a lot of this"; the question here is about *one address*. **`weighted-overlay`
  rejected** — a composite "disruption score" is the invented number this app refuses.
- **WIF `connections`** — 14 (`DESIGN-PROPOSAL.md` §5), signature loop first.
- **DataSource linking** — 9 widgets on shared `sourceId`s (`items`/`rungs`/`bodies`/`services`);
  without it the connections table would be ~20 rows longer.
- **`FileDataSource`** — the city's own planning-case export (CSV with hearing dates) turns rung ②
  from empty to live with **no schema change**. **`RestDataSource`** — Caltrans LCS + proxied Legistar.
- **`arcade`** — one `valueExpression` deriving `rung` on the live renderer, so the horizon control
  repaints without a rebuild.
- **Deliberately NOT used:** `filter`/`query` builders (a free-form WHERE lets a resident construct a
  class of item the ladder cannot then assign a right to — the rung rule must own the classification),
  `add-data` (a public app must not let a reader add a layer and mistake it for the city's word),
  `chart`/`carto`/`sparkline`, `swipe`/`bookmarks`/`controller`, `timeslider` (18 months of start dates
  is not an animation; the horizon control is the honest form), `routing`/isochrones (recourse is not
  a drive time), `measure`/`draw`/`elevation`, `views` (the rung **is** the state machine), and
  **editing — no write path at all**. Full sweep: `DESIGN-PROPOSAL.md` §8.
- **Composed export** — the **printed address record** (map + ring + ladder + bodies + statute
  citations + vintage), a **per-Neighborhood-Council atlas** (the 99-page book a Council office
  actually wants), a per-item **feature report**, CSV/GeoJSON, and `exportSpec` so the record
  round-trips to ArcGIS. **One CSV path** serves both the working list and the pinned packet, so the
  stamp — scope, vintage, *"not the notice of record"*, and the boundary generalisation when area
  mode is on — cannot be forgotten on one of them.
- **The staff console (shipped).** Area mode over council districts / neighborhood councils; filter
  chips; an agency breakdown carrying each agency's unreachable count; three quick filters; a
  sortable table with hover→map and ↑↓/Enter/P; a pin tray with a call list and a packet; three
  resizable seams. All of it reads the same `items` population and the same rung rule — none of it
  is a second model.

---

## 3. Data sources

All EPSG:4326 (reproject on ingest — every LA City service is `wkid 102100`). Every row below was
`curl`'d on **2026-08-09**; counts are the literal `returnCountOnly` / `returnIdsOnly` / `outStatistics`
responses. Full provenance, schemas and traps: **`pim-catalog-ca.json`** (18 external feeds, 16
verified-live).

**The discovery.** California's crawled public estate — all **2,360** services in
`data_sources_ca.md` — contains **zero** works-and-closures layers, **zero** decision-in-progress
layers and **zero** notice-channel layers, because it holds **no municipal server at all** (the state
has 483 incorporated cities). But LA's public map viewer, **NavigateLA**, is served from an ArcGIS
server that is not in the catalogue:

> **`https://maps.lacity.org/lahub/rest/services`** — City of Los Angeles, **35 services**,
> `Map,Query,Data`, **CORS open** (`Access-Control-Allow-Origin` reflects the Origin — browser-direct,
> no proxy). Public-way reservations, council districts, 99 neighborhood councils, capital projects,
> parcels, collection days.

| Role | City of Los Angeles (the discovered server) | California (public estate) | National / statutory |
|---|---|---|---|
| **Works — what is happening** | `Public_Way_Reservations/MapServer` **33** points (11,283) · **34** lines (7,843) · **35** polygons (43,180) · **37** One-Year Moratorium (292) · **38** Proposed Resurfacing (5,531) · **40** Proposed Slurry (6,585) · `Citywide_CIP/MapServer/0` (863) | ⛔ **none** | **Caltrans LCS** `cwwp2.dot.ca.gov/data/d{N}/lcs/lcsStatusD{NN}.json` — keyless, current + 7 days |
| **Decision — what is being decided** | ⛔ **none** — no hearing calendar as data; `data.lacity.org` search for *hearing*/*notice* returns nothing relevant | ⛔ **none** | **Legistar** `webapi.legistar.com/v1/{client}/Events` — keyless, **12 CA clients**, ⚠ **no CORS**. **Gov. Code §54954.2** (72 h) · **§65091** (300 ft, owners) |
| **Channel — who you can tell** | `Boundaries/MapServer` **18** Neighborhood Councils (**99**, `DWEBSITE`/`DEMAIL`/`DPHONE`, 98/99 email) · **13** Council Districts (15, member named) · **9** Community Plan Areas (36 — ⚠ count endpoint lies) | ⛔ none (6 Cal OES inter-agency rosters, rejected) | — |
| **Geocoding an address** | `centerlineLocator/GeocodeServer/findAddressCandidates` — the city's own centerline file, keyless, CORS-open, returns a `score`. ⚠ **pass `outSR={"wkid":4326}`** (trap L16) | ⛔ none | Nominatim (`@strata/plugin-search`) as a fallback outside the city |
| **Address frame — the unit + the ring** | `Landbase_Information/MapServer` 0–9 — Parcels, Parcels (LA County APN), Building Footprints (LARIAC5), Centerlines, Tracts | statewide parcels `UCD_Parcels` — **display-only, `query` returns 400** (catalogued) | — |
| **Service entitlement** | `Boundaries/MapServer` **22** Solids Collection Service District Days (49) · 19 Neighborhood Service Areas · 20 Sanitation Maintenance Districts | 36 parks/libraries/facilities layers | — |
| **Service request (outbound)** | `data.lacity.org` MyLA311 (Socrata, CORS `*`) | ⛔ none | — |
| **Reporting frame** | `Boundaries/MapServer` 2/10/13/23/26 (Assembly, Congressional, Council, Senate, Zip) | 103 frame layers · `California_Incorporated_City_Boundaries` (483) | Census TIGERweb |
| **Hazard context** | `Geotechnical_and_Hydrological_Information` · `Special_Areas` | 54 hazard layers (FHSZ, floodplain, seismic) | FEMA NFHL *(non-CORS)* |
| **The renter share §65091 misses** | — | ⛔ none | **Esri Living Atlas ACS** `ACS_Housing_Tenure_by_Race_View_Boundaries/FeatureServer/1` — keyless; LA County **45.9 % own / 54.1 % rent** |

**CORS:** `maps.lacity.org` reflects the request Origin and `data.lacity.org` is `*` — **browser-direct,
no proxy** for the demo city's own data. **Legistar is the exception: GET 200 keyless but with no
`Access-Control-Allow-Origin`, and the OPTIONS preflight returns 403** → dev-proxy, a reference proxy,
or build-time ingest. Carry City of Los Angeles (NavigateLA/BOE), Caltrans, US Census/ACS and
OpenStreetMap attribution plus "no warranty", and state on screen that **this is not the notice of
record**.

### Query capabilities — what makes "my whole district" answerable

Three things this server supports, verified 2026-08-09, which together turn a district question from
impossible into one request per layer. Also recorded in `data_sources_ca.md` § [LG].

| Capability | Why it matters |
|---|---|
| **`POST` on `/query`** | the endpoint takes a form-encoded body, so a district polygon can be sent **as** the query geometry; a GET would exceed the URL limit and there is no other way to ask |
| **`maxAllowableOffset` on the boundary** | CD13 at offset `0.0005` drops from **2,479 vertices / 104 KB to 107 / 2.3 KB** — small enough to *be* that geometry. Costs ±~55 m of boundary error, which the app discloses on screen and in every export |
| **`CURRENT_TIMESTAMP + INTERVAL 'N' DAY`** | the horizon runs server-side: CD13 unfiltered **4,717**, over 90 days **4,302**. End to end a whole district is ~**1.4 MB in ~6 s**, with item geometry generalised at `0.00005` |

### The asymmetry, measured

| | value |
|---|---|
| Reservations of the public way | **62,306** |
| …in progress right now | **43,074** |
| …carrying a comment / objection / appeal / hearing date | **0** |
| …naming no contact email | **10,193 (16.4 %)** |
| …**line features** naming no contact email | **5,897 of 7,843 (75.2 %)** |
| …with `EventType` blank | **62,306 (100 %)** |
| …with `StreetName` blank | **62,306 (100 %)** |
| …ending in the year 2099 | **539** |
| …in progress and ending within 30 / 90 days | 5,228 / 12,835 |
| Neighborhood Councils (Brown Act bodies) | **99**, in 12 regions |
| …publishing a phone / website / email | **99 / 99 / 98** |

### Traps that bite (all reproduced, not inferred)

| # | Trap |
|---|---|
| **L1** | **`EventType` and `StreetName` are blank on all 62,306 PWRS rows.** The two fields a resident would search by. Any text-search UI over this registry returns nothing, forever, with no error. The map is the only interface. |
| **L2** | **Contact completeness varies by GEOMETRY TYPE**: 0.09 % of polygons lack an email, against **75.2 %** of lines. Sampling one layer and generalising gives a number that is wrong by two orders of magnitude. Measure per layer. |
| **L3** | **`returnCountOnly=true` returns `{"count":0}` on `Boundaries/9` (Community Plan Areas) while `returnIdsOnly=true` returns 36 ids.** The count endpoint lies, with a 200. Every other layer tested agrees — so it is per-layer, which is worse: one KPI silently reads zero. **Assert `returnCountOnly == len(returnIdsOnly)` on any layer whose count reaches a KPI.** |
| **L4** | **`Boundaries/13.NAME` is the councilmember's personal name, not the district's** (`NAME`='Eunisses Hernandez', `District`=1). A label bound to `NAME` prints a person where a reader expects a place, and goes stale at the next election, silently. Bind labels to `District`. |
| **L5** | **PWRS layers 0/8/16/32/36 are GROUP layers** — `?f=json` returns a name with `geometryType: null` and no `fields`, and `/query` returns **HTTP 400 "Invalid or missing input parameters"**. The data is in the point/line/polygon children (1/2/3, 9/10/11, 17/18/19, 33/34/35). |
| **L6** | **539 records carry `EndDate` = 2099-01-01**, a sentinel for "open-ended". `ActiveStatus` is derived correctly from the dates (zero rows are "In Progress" with a past end), so the sentinel silently inflates *In Progress* rather than corrupting it. Bucket sentinels explicitly; never quote "43,074 things are happening now" without them. |
| **L7** | **The earliest in-progress `StartDate` is 1999-01-30**, and **69** in-progress rows started before 2010 (43 points · 8 lines · 18 polygons). The registry has no retirement policy, so "In Progress" includes reservations older than most of the app's users' tenancies. |
| **L8** | **Legistar has no CORS.** Keyless GET returns 200 with **no `Access-Control-Allow-Origin`**; the OPTIONS preflight returns **403 "Key or Token is required"**. Server-side verification succeeds and the browser still fails — the exact shape of bug that ships. |
| **L9** | **The City of Los Angeles is not a Legistar client** (`lacity` 500s). **`lacounty` and `metro` are** — so an LA resident's County and transit agendas are machine-readable while their City's are not. Confirmed CA clients: `metro`, `lacounty`, `culver-city`, `beverlyhills`, `longbeach`, `sanjose`, `oakland`, `sacramento`, `fresno`, `stockton`, `chulavista`, `sanbernardino`. |
| **L10** | **Agenda item titles are not addresses.** §54954.2 caps them at a "brief general description" (~20 words) and most are `Roll Call` / `Consent Calendar`. Do **not** geocode them — place a meeting at its body's boundary or meeting address. |
| **L11** | **`api.census.gov` is key-gated** — a bare call returns HTTP 200 whose body is an HTML page titled *Missing Key*. Third recipe to record it (catchment 2026-08-06, gap 2026-08-09, here). Use the Esri Living Atlas ACS republication, which is keyless. |
| **L12** | **`D4_Right_of_Way_Boundary` returns `Token Required`** — an endpoint listed in `data_sources_ca.md` that is not publicly readable. Catalogue correction. |
| **L13** | **CAL FIRE names one service per wildfire** (`FS_<INCIDENT>_<YEAR>_<UNIT>_<N>`), so a civic-vocabulary sweep matches the 2024 **Park Fire** and the 2025 **District Fire**. Match the naming signature, not a word blocklist — the next fire called "Council" walks straight in. |
| **L14** | **The statewide parcel layer is display-only** (`UCD_Parcels`: `query` → 400, `maxRecordCount` 100). A 300-ft ring cannot be computed statewide from the state estate; it needs the city's or county's parcels. |
| **L15** | **`maxRecordCount` is 1,000–20,000 depending on the service** on this server (Street_Information 1,000; PWRS 20,000; CIP 2,000). Page everything and check `exceededTransferLimit`; an un-paged query **understates** the count. |
| **L16** | **The city geocoder's default output SR is Web Mercator, not 4326.** `centerlineLocator/GeocodeServer/findAddressCandidates` without `outSR` returns `{"spatialReference":{"wkid":102100},"candidates":[{"location":{"x":-13162802,"y":4036039}}]}` — metres. Fed to a 4326 map that address lands in the Gulf of Guinea. The response declares its own `spatialReference`, so this is silent only if you don't read it. Always pass `outSR={"wkid":4326}`. (Both `Street=` and the advertised `Single Line Input=` work as the query parameter.) |

---

## 4. Verify each URL first (terminal)

```bash
UA='Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 Chrome/126.0 Safari/537.36'
B=https://maps.lacity.org/lahub/rest/services

# ── THE DISCOVERY: a municipal ArcGIS server absent from data_sources_ca.md ──────────────────
curl -s -A "$UA" "$B?f=json" | python -c "import json,sys;d=json.load(sys.stdin);print(len(d['services']),'services')"
                                                                            # → 35 services

# ── CORS: decisive — can a browser reach it at all? ──────────────────────────────────────────
curl -s -I -A "$UA" -H "Origin: https://example.org" "$B/Boundaries/MapServer?f=json" | grep -i access-control
                                              # → Access-Control-Allow-Origin: https://example.org

# ── THE REGISTRY. NOTE layers 32/36 are GROUP layers and 400 on /query (trap L5) ─────────────
P=$B/Public_Way_Reservations/MapServer
q(){ curl -s -A "$UA" -G "$1/query" --data-urlencode "where=$2" \
     --data-urlencode "returnCountOnly=true" --data-urlencode "f=json"; }
for L in 33 34 35; do q "$P/$L" "1=1"; done            # → 11283 · 7843 · 43180   (62,306)
for L in 33 34 35; do q "$P/$L" "ActiveStatus='In Progress'"; done   # → 10259 · 3037 · 29778 (43,074)
q "$P/32" "1=1"                          # → {"error":{"code":400,...}}  ← group layer. L5.

# ── FACT 1: no field in the schema can express a right ───────────────────────────────────────
curl -s -A "$UA" "$P/35?f=json" | python -c "
import json,sys;print([f['name'] for f in json.load(sys.stdin)['fields']])"
 # → [...'ContactName','ContactPhone','ContactEmail','EnterDate','StartDate','EndDate',
 #     'ActiveStatus',...]   NO comment / objection / appeal / hearing field. 24 fields.

# ── FACT 2: the contact gap, and that it varies by GEOMETRY TYPE (trap L2) ───────────────────
for L in 33 34 35; do q "$P/$L" "ContactEmail IS NULL OR ContactEmail=''"; done
                                                     # → 4259 · 5897 · 37     = 10,193 (16.4%)
                                                     #   lines: 5897/7843 = 75.2%

# ── FACT 3: the two fields a resident would search by are empty on EVERY row (trap L1) ───────
for L in 33 34 35; do q "$P/$L" "EventType IS NULL OR EventType=''"; done     # → 11283·7843·43180
for L in 33 34 35; do q "$P/$L" "StreetName IS NULL OR StreetName=''"; done   # → 11283·7843·43180

# ── FACT 4: the 2099 sentinel, and how little of "now" ends soon (traps L6/L7) ───────────────
for L in 33 34 35; do q "$P/$L" "EndDate >= DATE '2098-01-01'"; done          # → 124 · 228 · 187 (539)
for L in 33 34 35; do q "$P/$L" "ActiveStatus='In Progress' AND EndDate <= DATE '2026-09-08'"; done
                                                                             # → 806 · 75 · 4347 (5,228)
for L in 33 34 35; do q "$P/$L" "ActiveStatus='In Progress' AND EndDate < CURRENT_TIMESTAMP"; done
                                             # → 0 · 0 · 0   ActiveStatus is DERIVED and honest.

# ── THE COUNTERWEIGHT: the channel, and that it IS complete ──────────────────────────────────
N=$B/Boundaries/MapServer/18
q "$N" "1=1"                                                    # → 99 neighborhood councils
q "$N" "DEMAIL IS NULL OR DEMAIL=''"                            # → 1
q "$N" "DPHONE IS NULL OR DPHONE=''"                            # → 0
q "$B/Boundaries/MapServer/13" "1=1"                            # → 15 council districts
curl -s -A "$UA" -G "$B/Boundaries/MapServer/13/query" --data-urlencode "where=District=1" \
  --data-urlencode "outFields=NAME,District" --data-urlencode "f=json" | head -c 140
                   # → "NAME":"Eunisses Hernandez"  ← the MEMBER, not the district. Trap L4.

# ── TRAP L3: the count endpoint lies on exactly one layer ────────────────────────────────────
q "$B/Boundaries/MapServer/9" "1=1"                             # → {"count":0}
curl -s -A "$UA" -G "$B/Boundaries/MapServer/9/query" --data-urlencode "where=1=1" \
  --data-urlencode "returnIdsOnly=true" --data-urlencode "f=json" \
  | python -c "import json,sys;print(len(json.load(sys.stdin)['objectIds']),'ids')"    # → 36 ids

# ── THE OTHER RUNGS ─────────────────────────────────────────────────────────────────────────
q "$B/Citywide_CIP/MapServer/0" "1=1"                           # → 863 capital projects
q "$P/37" "1=1"; q "$P/38" "1=1"; q "$P/40" "1=1"               # → 292 · 5531 · 6585
q "$B/Boundaries/MapServer/22" "1=1"                            # → 49 collection-day districts

# ── THE STATUTORY CLOCK: keyless, and NOT CORS-reachable (traps L8/L9) ──────────────────────
for c in metro lacounty longbeach sanjose oakland sacramento fresno stockton \
         chulavista sanbernardino culver-city beverlyhills lacity; do
  printf "%-16s %s\n" "$c" "$(curl -s -A "$UA" "https://webapi.legistar.com/v1/$c/Bodies?\$top=1" | head -c 1)"
done            # → "[" for 12 clients; lacity returns an error object. L9.
curl -s -D - -o /dev/null -X OPTIONS -H "Origin: https://example.org" \
  -H "Access-Control-Request-Method: GET" "https://webapi.legistar.com/v1/sacramento/Bodies" | head -1
                                                    # → HTTP/1.1 403 Key or Token is required. L8.
curl -s -A "$UA" "https://webapi.legistar.com/v1/sacramento/Events?\$top=3&\$filter=EventDate%20gt%20datetime'2026-08-09'&\$orderby=EventDate" \
 | python -c "
import json,sys
for e in json.load(sys.stdin): print(e['EventBodyName'],e['EventDate'][:10],e.get('EventTime'),bool(e.get('EventAgendaFile')))"

# ── THE RENTER SHARE §65091 DOES NOT ADDRESS (keyless; api.census.gov is gated — L11) ───────
curl -s -A "$UA" -G "https://services.arcgis.com/P3ePLMYs2RVChkJx/arcgis/rest/services/ACS_Housing_Tenure_by_Race_View_Boundaries/FeatureServer/1/query" \
  --data-urlencode "where=GEOID='06037'" \
  --data-urlencode "outFields=NAME,B25003_calc_pctOwnE,B25003_calc_pctRentE" \
  --data-urlencode "returnGeometry=false" --data-urlencode "f=json"
                                # → Los Angeles County  45.9 own / 54.1 rent
curl -s -L "https://api.census.gov/data/2023/acs/acs5?get=NAME,B25003_001E&for=place:44000&in=state:06" \
  | head -c 60                  # → <html ... <title>Missing Key</title>          L11

# ── AREA MODE: the three capabilities that make a whole district answerable ──────────────────
B13=$B/Boundaries/MapServer/13/query
# full resolution vs generalised — the reason a district polygon can be a query geometry at all
for OFF in "" "&maxAllowableOffset=0.0005"; do
  curl -s -A "$UA" "$B13?where=District%3D13&returnGeometry=true&outSR=4326&f=json$OFF" \
  | python -c "
import json,sys
g=json.load(sys.stdin)['features'][0]['geometry']
print(' vertices', sum(len(r) for r in g['rings']), '| bytes', len(json.dumps(g)))"
done                                    # → 2479 / 104276   then   107 / 2341

# POST the generalised polygon as the query geometry (a GET would blow the URL limit)
python - <<'PY'
import json, urllib.request, urllib.parse
UA="Mozilla/5.0"; B="https://maps.lacity.org/lahub/rest/services"
def get(u,p): return json.load(urllib.request.urlopen(urllib.request.Request(
    u+"?"+urllib.parse.urlencode(p), headers={"User-Agent":UA}), timeout=90))
g=get(f"{B}/Boundaries/MapServer/13/query",{"where":"District=13","returnGeometry":"true",
      "outSR":"4326","maxAllowableOffset":"0.0005","f":"json"})["features"][0]["geometry"]
geom=json.dumps({"rings":g["rings"],"spatialReference":{"wkid":4326}})
# the horizon as SERVER-side SQL — INTERVAL works in a WHERE on this server
H=("(EndDate >= CURRENT_TIMESTAMP OR EndDate IS NULL) "
   "AND StartDate <= CURRENT_TIMESTAMP + INTERVAL '90' DAY")
for L in (33,34,35):
    for where,label in (("1=1","all "),(H,"90d ")):
        d=urllib.parse.urlencode({"geometry":geom,"geometryType":"esriGeometryPolygon","inSR":"4326",
            "spatialRel":"esriSpatialRelIntersects","where":where,"returnCountOnly":"true",
            "f":"json"}).encode()
        r=urllib.request.urlopen(urllib.request.Request(
            f"{B}/Public_Way_Reservations/MapServer/{L}/query", data=d,
            headers={"User-Agent":UA,"Content-Type":"application/x-www-form-urlencoded"}),timeout=180)
        print(f"  CD13 L{L} {label}", r.read().decode()[:40])
PY
 # → L33 all 952 / 90d 915 · L34 all 431 / 90d 204 · L35 all 3334 / 90d 3101
 #   whole district: 4,717 unfiltered → 4,302 in a 90-day window

# ── THE GEOCODER — and the default SR that will put your address in the Gulf of Guinea (L16) ──
G=$B/centerlineLocator/GeocodeServer/findAddressCandidates
curl -s -A "$UA" -G "$G" --data-urlencode "Street=200 N Spring St" --data-urlencode "f=json" \
 | python -c "import json,sys;d=json.load(sys.stdin);print(' default SR',d['spatialReference'],d['candidates'][0]['location'])"
                     # → wkid 102100 · x -13162802, y 4036039   ← METRES
curl -s -A "$UA" -G "$G" --data-urlencode "Street=200 N Spring St" \
 --data-urlencode 'outSR={"wkid":4326}' --data-urlencode "f=json" \
 | python -c "import json,sys;d=json.load(sys.stdin);print(' with outSR ',d['candidates'][0]['location'])"
                     # → x -118.2435, y 34.0539

# ── THE STATUTES (read them; they are the rung rule) ─────────────────────────────────────────
# Gov. Code §54954.2  https://leginfo.legislature.ca.gov/faces/codes_displaySection.xhtml?lawCode=GOV&sectionNum=54954.2
# Gov. Code §65091    https://leginfo.legislature.ca.gov/faces/codes_displaySection.xhtml?lawCode=GOV&sectionNum=65091

# ── CALIFORNIA'S PUBLIC ESTATE: prove the absence that justifies this recipe ─────────────────
python build_pim_catalog.py
 # → library 2360 · role-tagged 250 · hand-rejected 8 (reasons recorded) · noise 8
 #   ASSERTED EMPTY: works-and-closures, decision-in-progress, notice-channel — holds
```

**Real field names that drive symbology / rungs / KPIs.** PWRS: `PWRS_ID`, `AutoID`, `AgencyName`,
`AgencyFullName`, **`Category`**, `Type`, `Entity`, `ContactName`, **`ContactPhone`**,
**`ContactEmail`**, `EnterDate`, **`StartDate`**, **`EndDate`**, `StartDateFormatted`,
`EndDateFormatted`, **`ActiveStatus`**, `NLA_URL`, `REPORT_URL` — and the two dead ones, `EventType`
and `StreetName`. Moratorium: `SECT_ID`, `Street`, `PGM_YR`, `CLOSEDT`, `StreetAgeDays`, `ST_FROM`,
`ST_TO`, `AssetID` (the OID — there is no `OBJECTID`). Resurfacing: `SECT_ID`, `STREET`, `FISCALY`
(a String), `SCHEDULE`, `COMPLETION_DT`, `Bureau_Construction_Date`. CIP: `ProjectTitle`,
`ProgramName`, `PM_Name`, `PM_Phone`, `PM_EMail`, `ConsStartDate`, `ConsEndDate`, `CouncilDistrict`,
`ConstructionCost`, `ConstPercComp`. Neighborhood Councils: `NAME`, `NC_ID`, `WADDRESS`, `DWEBSITE`,
`DEMAIL`, `DPHONE`, `SERVICE_RE`, `CERTIFIED`. Council Districts: **`District`** (label on this, not
`NAME`), `District_Name`. Collection days: `District`, `Day`, `ShortDay`, `District_Name`. Legistar:
`EventId`, `EventBodyName`, `EventDate`, `EventTime`, `EventLocation`, `EventAgendaFile`. ACS:
`B25003_calc_pctRentE`.

---

## Guided wizard — **the prompts that assign the app's defaults**

Claude runs this like an ArcGIS Instant-App wizard: ask each group, **apply the default so "accept
all" builds a complete app**, confirm a one-line summary, then run §5. Launch with
`/recipe local-government_public-information-map`. Every answer *sets an application default* baked
into `layers.json` / the `AppLayout`.

| # | Wizard question | Options → **default** | The default it assigns |
|---|---|---|---|
| 1 · App | Title & city? | free text → **"Public Information Map"**, **City of Los Angeles** | header title + `strata:notes.asOf` + initial extent |
| 2 · Works source | What is happening on the street? | **the city's public-way / ROW permit registry** · capital projects only · none (rung ③ renders empty with a reason) | the `items` layer and rungs ③/④ |
| 3 · **Decision source** | **Does the city publish hearing dates as data?** | **no → rung ② renders empty and names the absence** · yes, a FeatureServer · yes, a CSV I will upload (`FileDataSource`) | whether rung ② is live; the §65091 10-day clock |
| 4 · Channel | Which bodies can a resident address? `[multi]` | **neighborhood/community councils + the district member** · council only · + County / transit via Legistar | the `bodies` layer + rung ①'s 72-hour clock; Legistar needs the proxy (trap L8) |
| 5 · Ring | Draw the statutory notice radius? | **yes — 300 ft, stroke only, §65091 cited** · 500 ft · 1,000 ft · off | the `ring` layer + the "addressed to owners" sentence |
| 6 · The renter line | Show who the mailed notice does not reach? | **yes — the county renter share, cited** · no | the ACS callout beneath the ring |
| 7 · Horizon | How far ahead? | **30 days · 90 days · 18 months (switchable, default 90)** | the `date-filter` presets + `clock_days` |
| 8 · Sentinels | Records ending in 2099 / started before 2010? | **bucket into rung ⑥ and say why** · treat as in progress · hide | rung ⑥'s membership rule (trap L6/L7) |
| 9 · Contacts | Republish blank contacts? | **yes — show "nobody is named", it is the finding** · hide blank rows | the gauge, the tinted table rows, rung ⑥ |
| 10 · Services | Include the ordinary answers? `[multi]` | **collection day + districts** · + facilities · none | the `services` accordion |
| 11 · Report | Outbound 311 link? | **yes, deep-link the city's 311 with the item id** · no | rung ⑤ |
| 12 · Export/Theme | Artifacts? · theme/language? | **printed address record + per-council atlas**; **light / EN + ES** · dark · EN only | `/export report` + atlas; `ThemeSpec` mode + `lang-switch` |

**Then:** Claude echoes *"City of Los Angeles · PWRS works · no hearing feed (rung ② empty, reason
shown) · 99 NCs + district member · 300 ft ring · renter line on · 90-day horizon · sentinels to rung
⑥ · blank contacts shown · collection day · 311 deep-link · address record + council atlas · light /
EN+ES"* and, on confirmation, runs §5 — so the app opens **fully configured**.

---

## 5. Prompt-script (run in order)

```
A. /new-app — a "Public Information Map" open-design app ("recourse-ladder", see DESIGN-PROPOSAL.md
   §3-§6). LIGHT ThemeSpec with a one-hue-one-meaning system: primary #1e6b52 (A DOOR STILL OPEN),
   warning #b45309 (a clock running out), secondary #475569 (information you cannot act on),
   danger #8a3324 (RESERVED: names nobody / never ends), fonts.scale default, mono+tabular-nums
   override on kpi and table, motion 160ms. App-shell: header (title + address search + ring radius
   + horizon + print + lang-switch + theme-switch), footer (attribution + share + "not the notice of
   record"), splash stating the scope in two sentences. Two pages: `now` (fixed) and `dossier`
   (scroll). Install deps + give me the run command.

B. Build `items` — the union that is the whole design. /convert each LA City service to EPSG:4326
   GeoParquet (they are wkid 102100 — request outSR=4326), then UNION into one layer with a common
   schema (item_id, kind, agency, title, start, end, contact_name, contact_phone, contact_email,
   source_url):
     • Public_Way_Reservations/MapServer 33 + 34 + 35   -> kind from `Category`  (62,306)
       ⚠ layers 32/36 are GROUP layers and 400 on /query — use the point/line/polygon children (L5)
       ⚠ `EventType` and `StreetName` are blank on EVERY row — do not map them, do not search on
         them, and do not offer a text search over this layer (L1)
     • Public_Way_Reservations/MapServer 37 (moratorium, 292) + 38 (resurfacing, 5,531)
       + 40 (slurry, 6,585)
     • Citywide_CIP/MapServer/0 (863) -> contact from PM_Name/PM_Phone/PM_EMail
     • Caltrans LCS cwwp2.dot.ca.gov/data/d7/lcs/lcsStatusD07.json -> the state-highway seam
   Add `has_contact` (false on the 10,193 with no email — do NOT drop them), `is_sentinel`
   (EndDate >= 2098-01-01, 539 rows), and `clock_days`. Page EVERYTHING with resultOffset and assert
   exceededTransferLimit is false — maxRecordCount varies 1,000-20,000 on this server (L15).
   /publish (folder=losangeles, service=pim).

C. Build `bodies` and `rungs`.
   `bodies` = Boundaries/18 (99 neighborhood councils; DWEBSITE/DEMAIL/DPHONE) + Boundaries/13
     (15 council districts — ⚠ label on `District`, NEVER on `NAME`, which is the member's personal
     name and goes stale at the next election, L4) + optionally Legistar events for `lacounty` and
     `metro` (⚠ NO CORS: proxy or build-time ingest, L8; the City of LA is not a client, L9).
     Compute `next_meeting` and `agenda_due` = next_meeting - 72h (Gov. Code 54954.2(a)(1)).
   `rungs` = the 6-row roll-up of `items` + `bodies` by the rule in DESIGN-PROPOSAL.md §1.5:
     speak / object / plan / call / report / nothing, each with n, soonest_clock, pct_reachable and
     — when empty — a `reason` string. ASSERT the marginals against §4: 62,306 items, 43,074 in
     progress, 10,193 with no email, 0 with any comment/appeal date.
   ⚠ ASSERT returnCountOnly == len(returnIdsOnly) on every layer whose count reaches a KPI — on
     Boundaries/9 the count endpoint returns 0 against 36 real ids, with a 200 (L3).

D. Build `ring` and `services`. /analyze buffer 300 ft around the searched address (Gov. Code
   65091(a)(3)) -> pointsWithin against Landbase_Information parcels -> `parcel_n`. STROKE ONLY,
   never a fill — a filled ring reads as a hazard zone. The app counts parcels inside it and must
   NEVER claim to know who was mailed (the owner roll is not published). `services` =
   Boundaries/22 collection days (49) + district lookups. Add the ACS renter callout from Esri
   Living Atlas ACS_Housing_Tenure_by_Race_View_Boundaries/1 (LA County 54.1% renter) — api.census.gov
   is key-gated (L11).

E. /symbology + /popup — genuine ESRI JSON on verified fields only.
   items: uniqueValue on `rung` (an arcade valueExpression derives it from the dates + has_contact) —
     open-door green #1e6b52, clock amber #b45309, slate #475569, and #8a3324 ONLY for rung 6;
     fill alpha 40/255; esriSMSCircle for points, NEVER esriSMSPath.
   ring: stroke-only outline, no fill. bodies: light outline + label on `District` / NC `NAME`.
   /popup items in PLAIN LANGUAGE: what it is, who is doing it, when it starts and ends, who to call
     — and where there is no contact, the literal line "This record does not name anyone to contact."
     Add "Dates are as published; some records carry an open-ended end date of 2099."

F. /panel statistics as the answer rail: "Things at this address" (count), "Next deadline"
   (min clock_days, amber), and a radial gauge "Items naming a human" (avg has_contact, thresholds
   50/85 — LA reads 83.6% overall and 24.8% on line features). All as kpi/gauge `stat:{field,op}`
   bound to sourceId `items` — no connections. "Items with a comment period" and "owners mailed"
   render GREYED with their reason until a city binds a case export via FileDataSource.

G. The ladder + the rail.
   ① SHIP THE FALLBACK: /panel table over `rungs` (6 rows: label, n, soonest_clock, pct_reachable,
      reason) whose rowSelect carries {field:"rung_key"}. Register the app-local `recourse-ladder`
      widget later (DESIGN-PROPOSAL.md §9) — the loop is identical.
   ② interactive legend over `items.kind` with live counts, and the hint line that filtering by kind
      does NOT change the adopted rung.
   ③ accordion: the derivation (the rung rule + the statute text) · your services · what this map
      cannot show.
   ④ /panel table `items` (server-paged, CSV/GeoJSON, rows tinted where has_contact=false).
   ⑤ page 2 `dossier`: the printable address record — map + ring + bodies table + items table.

H. WIF + controls + export: author AppLayout.connections — the 14 in DESIGN-PROPOSAL.md §5, with the
   SIGNATURE LOOP first (ladder categorySelect -> filter on map + item table + answer card +
   setUrlParam). ONE map-control vocabulary: hide MapLibre's own cluster and mount a single
   top-right stack (zoom in / out / fit / Layers / Basemap / Legend) with drawers opening BESIDE it
   and L/B/G shortcuts; the basemap drawer shows a REAL TILE of the current location per style.
   status-bar EPSG:4326. Wire /export report (the printed address record: map + ring + ladder +
   bodies + the statute citations + the data vintage), a per-neighborhood-council atlas (99 pages),
   /export image, /export layer csv, and a share deep-link carrying the address + adopted rung.
   Verify responsive.small collapses the splitter and that THE LADDER SURVIVES INTACT (full-width
   rows, never a dropdown).

I. THE STAFF CONSOLE — a PIO and a council chief of staff have a DISTRICT, not an address.
   ① Area mode: a picker over Boundaries/13 (15 council districts) and /18 (99 neighborhood
      councils). Fetch the chosen boundary with maxAllowableOffset=0.0005 and POST it as the query
      geometry; apply the horizon SERVER-side with
      "(EndDate >= CURRENT_TIMESTAMP OR EndDate IS NULL) AND StartDate <= CURRENT_TIMESTAMP +
       INTERVAL 'N' DAY"; generalise item geometry at 0.00005. ⚠ The boundary is GENERALISED — print
      the ±55 m caveat in the answer rail AND in every export. Carry the body's own contacts through
      with its geometry so rung ① is never an empty card.
   ② Filter chips for every narrowing (scope · horizon · rung · kind · agency · quick · pinned), each
      removable. A filter the user cannot see is a bug.
   ③ An agency list ("who is doing it") carrying each agency's unreachable count, and three quick
      filters: ends within 7 days · names nobody · no end date.
   ④ Table: sortable on 5 keys with aria-sort, hover -> highlight the item on the map, ↑↓/Enter/P to
      walk / select / pin.
   ⑤ A pin tray -> copy all emails · export packet · zoom to pinned. ONE csv path for the list and
      the packet so the stamp (scope, vintage, "not the notice of record", and the generalisation
      when area mode is on) can never be forgotten.
   ⑥ Three drag seams (ladder ⇄ map ⇄ answer, table height) as ARIA separators — keyboard-operable,
      double-click resets, and each drag calls map.resize().
   ⑦ Escape is PRIORITY-ORDERED: splash -> open drawer -> selection -> adopted rung.
```

---

## 6. Verify (benchmark to Esri My Neighborhood Services / AgencyCounter / Chicago Cityscape)

**Researched AND built 2026-08-09** — every endpoint in §3 curl'd, every count in §1/§3 reproduced,
all 16 traps reproduced rather than inferred, `pim-catalog-ca.json` generated with its emptiness
assertion holding, and `app/` shipped with **343 assertions green** plus a real-browser pass.

| Check | Pass |
|---|---|
| A municipal CA ArcGIS server discovered, enumerated (35 services) and CORS-verified | ✅ verified 2026-08-09 — `maps.lacity.org/lahub`, absent from `data_sources_ca.md` |
| California's crawled estate proven empty of works / decision / notice-channel layers (2,360 services swept) | ✅ asserted in `build_pim_catalog.py`; build fails if it changes |
| 16 vocabulary collisions inspected by hand and rejected **with reasons recorded**, not by narrowing a regex | ✅ 8 CAL FIRE incident names + 8 in the `REJECTED` table |
| The registry counted in full: 62,306 reservations, 43,074 in progress | ✅ live, per geometry type |
| **Zero of 62,306 records carry a comment, objection, appeal or hearing date** | ✅ the 24-field schema is reproduced in §4 |
| The contact gap measured, and shown to vary by geometry type (16.4 % overall, 75.2 % on lines) | ✅ live, per layer |
| `EventType` and `StreetName` blank on 100 % of rows | ✅ 62,306 / 62,306 on both |
| The 2099 sentinel (539) and the honesty of `ActiveStatus` (0 stale) both reproduced | ✅ |
| The counterweight measured: 99 Brown Act bodies, 99/99 phone, 98/99 email | ✅ |
| `returnCountOnly` proven to lie on exactly one layer (0 vs 36 ids) | ✅ trap L3 |
| Legistar proven keyless-but-CORS-blocked, and 12 CA clients enumerated | ✅ traps L8/L9 |
| The statutes read and quoted, not paraphrased (§65091 300 ft / owners / ⅛-page; §54954.2 72 h) | ✅ both fetched from leginfo |
| The renter share measured from a keyless source after confirming ACS is gated | ✅ 54.1 %, Living Atlas; `api.census.gov` → *Missing Key* |
| Every `layerId` + field verified against the live service (§4); no field written from memory | ✅ |
| **Application built and driven** | ✅ `app/` — **343 assertions** (`test-standing.mjs` 234 live-data + `test-render.mjs` 109 shipped-page), all green |
| Area mode: a whole council district on the same rung rule | ✅ CD13/90 days = **4,302 items**, partitioned by the four item rungs; POST + generalised boundary + server-side horizon all asserted |
| The staff surface — chips · agency filter · quick filters · sort · hover · keyboard · pin tray · packet | ✅ each driven in `test-render.mjs` against the shipped page |
| Three panels resize, by mouse AND keyboard, and reset on double-click | ✅ asserted as ARIA separators calling `map.resize()` |
| Escape is priority-ordered (splash → drawer → selection → rung) | ✅ the ORDER is asserted, not just the presence |
| WCAG AA on every rung colour, both themes | ✅ contrast computed in the suite: light 5.17–7.08:1, dark 5.34–7.23:1 (the amber was fixed `#b45309 → #a4490a`, not argued around) |
| The signature loop works end to end | ✅ driven in both suites and in headless Chrome: click a rung → map payload, table, answer card and URL all adopt it |
| Rung ② renders its reason as visible content, not a null state | ✅ asserted in the DOM, not just in the model |
| A 2099 sentinel never reaches the screen as a date | ✅ asserted over the rendered table + rails |
| The ring is stroke-only (no `ring-fill` layer exists) | ✅ asserted against the MapLibre layer set |
| Real-browser pass (headless Chrome, live data) | ✅ all six rungs render, map canvas mounts, deep-linked `?rung=` adopts, zero app console errors |
| Printed address record / per-council atlas | ⛏ **not built** — belongs to the `<StrataApp>` `/export` path |
| `<StrataApp>` / `AppLayout` rendering | ⛏ **authored in §2, unrendered** — `strata/node_modules` is absent from this checkout |

**On-par-or-better:** matches the address-lookup surface of Esri's My Neighborhood Services and the
cross-department unification AgencyCounter sells, and the deadline-awareness Chicago Cityscape charges
for — and **exceeds all of them on the one axis none of them competes on: sorting what is happening
by what the reader can still do about it, and printing the rule and the statute beside every
assignment, including the rungs that are empty.** Plus the AI-authored build, the sovereign/on-prem
posture, MIT, on Strata or ArcGIS. **Honestly less than:** Accela / Tyler / OpenGov on the permit
record itself (this app has no case management and no submission path); Granicus / PublicInput on
running an actual consultation; Chicago Cityscape on alert subscriptions and per-user monitoring
(this app has no notification channel at all — see §7); Esri on enterprise integration. We ship one
city's published record, sorted by recourse, with the derivation printed — and we say so on screen.

---

## 7. Harvest (gaps → strata-core)

Log as strata-core issues:

- **An `es` (Spanish) dictionary for `@strata/i18n`.** The package ships `en`/`ar` only. This is the
  **second** recipe to ask (`telecom_underserved-gap-and-take-rate`, 2026-08-09), which under the
  harvest rule makes it a real gap rather than a one-off. For a California public-facing app, shipping
  AR and not ES is equity theatre. One file; it unblocks the entire US public-sector catalog.
- **`recourse-ladder` as a core widget** (`DESIGN-PROPOSAL.md` §9) — a rail whose rows are *what the
  reader may still do*, each with a count, a clock and a reachability bar, and whose **empty rows
  render their reason**. The generic form — *"classify a population by the action available to the
  reader, and make the classification the navigation"* — would serve benefits eligibility, appeals,
  applicant status and consultation portals. Promote after a second use.
- **A first-class "empty with a reason" state** for data-bound widgets. Three recipes have now
  hand-rolled "this bucket is empty and here is why, in a sentence" (`telecom` band 0 / no-record row,
  `energy_pipeline` never-evaluated, this one's rung ②). `kpi`/`table`/`chart` should take an
  `emptyReason` prop and render it as content, not as a null state.
- **A count-integrity probe in `/add-data`.** `returnCountOnly` returns a confident `0` against 36
  real object ids on one layer of this server. `/add-data` should assert
  `returnCountOnly == len(returnIdsOnly)` on registration and warn on disagreement — this is the same
  class of trap as the gap recipe's hidden-`Status` field, now on a fourth server.
- **A group-layer guard.** PWRS layers 0/8/16/32/36 return a name with `geometryType: null` and 400 on
  `/query`. `LayerPanel` and `/add-data` should recognise a group layer and offer its children rather
  than registering a layer that cannot be queried.
- **A CORS probe surfaced at add-data time.** Legistar returns a keyless 200 to curl and fails in the
  browser (no `Access-Control-Allow-Origin`, preflight 403). Server-side verification passing while the
  browser fails is the exact bug that reaches production; a one-line preflight check at `/add-data`
  time would catch it.
- **Per-widget "assumption" affordance** — a first-class way to attach a printed derivation + source
  to any KPI or rung. This is now the **fifth** recipe to hand-roll it (`education_campus-operations`,
  `real-estate_site-selection`, `marketing_catchment`, `telecom_gap`, this one). It is a pattern.
- **A `notice-ring` helper.** "Buffer a point by a statutory distance, count the parcels inside, and
  render stroke-only with the citation" is three lines of Turf and a rendering rule, and it will recur
  in every US local-government recipe (§65091 here; setback, buffer-zone and notification radii
  elsewhere).
- **An `area-scope` primitive** — *"pick an administrative area, fetch its boundary generalised, POST
  it as a query geometry, push the time filter server-side, and disclose the generalisation error."*
  This app needed all five steps to answer "my district", every one is generic, and the disclosure is
  the part a hand-rolled version will skip. Reusable by any recipe whose buyer thinks in wards,
  basins, beats or service areas.
- **A `selection-tray` primitive** — pin → *copy all contacts* → *export a stamped packet*. The
  artifact a public-sector user actually leaves with is a call list and a document, and today every
  recipe rebuilds both. `data-actions` covers zoom/flash/export on a selection but has no notion of an
  accumulating set that survives a filter change.
- **A global `[hidden]` guard in the app scaffold.** An author `display:flex|grid` beats the UA
  stylesheet's `[hidden]{display:none}`, so `el.hidden = true` silently does nothing. It bit this app
  **twice** (the splash, then the radius control). `/new-app` should emit
  `[hidden]{display:none !important}` in the base stylesheet so nobody meets it a third time.
- **A `dataset`-aware test scaffold.** The render harness had to grow a live `data-*` view, compound
  class selectors and `append`/`classList.toggle` before it could drive real UI. If the shipped
  scaffold offered a minimal DOM fake, every recipe would not rebuild one badly.
- **An alerting seam.** The single feature every commercial rival has and this app does not is *"email
  me when something lands near my address."* strata-core has no notification primitive, and it should
  not grow one in core — but a documented seam (the app supplies `onWatch(address, rungs)`) would let a
  deployer wire it to their own govDelivery/SMTP without forking.

Editing is not applicable — this app has **no write path by design**.

---

## 8. Sources

- **Statute (read, not paraphrased):**
  [Gov. Code §54954.2 — the 72-hour agenda](https://leginfo.legislature.ca.gov/faces/codes_displaySection.xhtml?lawCode=GOV&sectionNum=54954.2) ·
  [Gov. Code §65091 — the 300-foot mailed notice](https://leginfo.legislature.ca.gov/faces/codes_displaySection.xhtml?lawCode=GOV&sectionNum=65091) ·
  Gov. Code §65090 · §54954.3 · §54956 · California Public Records Act (Gov. Code §7920 et seq.)
- **Why the mailed channel misses people:**
  [Einstein, Glick & Palmer, *Neighborhood Defenders* (Cambridge UP)](https://www.cambridge.org/core/books/neighborhood-defenders/0677F4F75667B490CBC7A98396DD527A) ·
  [their public-participation paper (PDF)](https://maxwellpalmer.com/docs/articles/Einstein_Glick_Palmer_Public_Participation.pdf) —
  participants in planning and zoning meetings are demographically unrepresentative and
  overwhelmingly opposed; the notice channel selects the audience.
- **Ministerial approvals — the decisions with no hearing at all:**
  [SF Planning Director Bulletin No. 9 — ministerial approval processes](https://sfplanning.org/resource/planning-director-bulletin-no-9-ministerial-approval-processes-mixed-income-housing) ·
  [Allen Matkins — streamlining middle housing](https://www.allenmatkins.com/real-ideas/streamlining-middle-housing-the-latest-in-small-site-development-reforms.html) ·
  [Brownstein — what's new in California housing law](https://www.bhfs.com/insight/whats-new-in-california-housing-law-an-overview-of-the-latest-signed-bills/)
- **How cities actually notice people:**
  [SF Planning — Neighborhood Notification](https://sfplanning.org/resource/neighborhood-notification) ·
  [San José — Guide to Public Hearings](https://www.sanjoseca.gov/your-government/departments-offices/planning-building-code-enforcement/planning-division/commissions-hearings-and-developers-roundtable/guide-to-public-hearings) ·
  [LA City Planning — Public Hearings](https://planning.lacity.gov/project-review/public-hearings)
- **Esri (nominative use):**
  [My Neighborhood Services](https://solutions.arcgis.com/local-government/help/my-neighborhood-services/) ·
  [Introduction to Capital Project Coordination](https://doc.arcgis.com/en/arcgis-solutions/latest/reference/introduction-to-capital-project-coordination.htm) ·
  [Capital Project Locator](https://solutions.arcgis.com/local-government/help/capital-project-locator/) ·
  [ArcGIS Hub for community engagement](https://www.esri.com/arcgis-blog/products/constituent-engagement/constituent-engagement/arcgis-hub-for-community-engagement)
- **Non-Esri platforms:**
  [AgencyCounter (formerly buildingeye)](https://www.agencycounter.com/) ·
  [Chicago Cityscape](https://www.chicagocityscape.com/) + its
  [development map](https://www.chicagocityscape.com/maps/development.php) ·
  [Granicus civic engagement alternatives (SoftwareWorld)](https://www.softwareworld.co/competitors/granicus-civic-engagement-platform-alternatives/) ·
  PublicInput · CivicPlus · Social Pinpoint · coUrbanize · Konveio · Decidim · SeeClickFix ·
  QScend QAlert · Accela · Tyler EnerGov · OpenGov · CentralSquare
- **Data endpoints (all curl-verified 2026-08-09 — see §4):**
  City of Los Angeles `maps.lacity.org/lahub/rest/services` (35 services, CORS-open) ·
  [data.lacity.org](https://data.lacity.org) (Socrata, CORS `*`) ·
  [Legistar Web API](https://webapi.legistar.com/v1/sacramento/Events) (keyless, no CORS, 12 CA clients) ·
  Caltrans CWWP2 LCS `cwwp2.dot.ca.gov/data/d{N}/lcs/lcsStatusD{NN}.json` ·
  Esri Living Atlas ACS `services.arcgis.com/P3ePLMYs2RVChkJx` ·
  [Census API key signup](https://api.census.gov/data/key_signup.html) (now required)
- **Internal:** `DESIGN-PROPOSAL.md` (this recipe's full design record) · `pim-catalog-ca.json` +
  `build_pim_catalog.py` · `../APP-TEMPLATE-LIBRARY.md` · `../DESIGN-CONTEXT.md` ·
  `../../data_sources/data_sources_ca.md` (§ *Local Government — [LG]*, added by this recipe) ·
  `../telecom_underserved-gap-and-take-rate/RECIPE.md` (format exemplar; its ACS and
  hidden-field findings are reconfirmed here) ·
  `../emergency-management_public-crisis-information-map/RECIPE.md` (the nearest cousin — the
  differentiation is set out in `DESIGN-PROPOSAL.md` §2.4) ·
  `strata/recipes/COMPONENT-MANIFEST.md` (§10 freestyle charter) · `strata/docs/guide/app-design.md`

---

## Modernization (parity release)

> Applies the Experience-Builder-parity features (Phases 1–7). See `strata/recipes/MODERNIZATION.md` +
> `COMPONENT-MANIFEST.md` §8. Cross-cutting: a structured **`theme`**, app-shell
> (`header`/`footer`/`splash`), and `dataSource.{ sourceId }` linking (widgets sharing a source link
> with **no `connections`**).

- **DataSource linking is the backbone, not a nicety.** Nine widgets — the two KPIs, the reachability
  gauge, the answer card, the services table, the ladder's fallback, the bodies table, the item table
  and the legend — bind to `sourceId` `items`/`rungs`/`bodies`/`services` and relink on every rung
  adoption with **zero** wiring. Without it this app would need roughly 20 more connections.
- **Structured `ThemeSpec` doing semantic work.** One hue means one thing — `primary` is *a door still
  open*, `warning` is *a clock*, `secondary` is *information you cannot act on*, and `danger` is held
  back for the single real failure state. A theme carrying an argument, not a palette.
  `overrides.kpi`/`overrides.table` put the countdown in mono with `tabular-nums`.
- **App-shell + `splash`.** The splash states the scope — *"this is not the legal notice of record"* —
  before the first render, because this app's honesty has to land before its numbers do.
- **`splitter`** once (ladder ↔ map ↔ answer) with `responsive.small` collapse — and the explicit rule
  that **the ladder survives the collapse intact**, never becoming a dropdown.
- **`panel`** for the ladder (dockable, collapsible) and **`accordion`** for the three secondary
  panels (derivation · services · what-this-cannot-show), so the answer stays above the fold.
  **`window`** for the body detail / add-to-calendar modal, opened by `showHide`.
- **Second page as `type:"scroll"`** with `scroll-reveal` — the printable address record is editorial,
  not operational, and gets the editorial silhouette.
- **`arcade`** one `valueExpression` deriving `rung` from the dates + `has_contact` on the live
  renderer, so the horizon control repaints without a rebuild.
- **`FileDataSource`** is the rung-② path: a CSV of planning cases with hearing dates turns the empty
  rung live with no schema change. **`RestDataSource`** carries Caltrans LCS and the proxied Legistar
  events.
- **Phase-2 triggers/actions used declaratively:** `search → zoomTo` + `refresh`, `rangeSelect →
  selectByGeometry` (the ring), `buttonClick → showHide` (the window). No `timer` — these are
  day-to-month clocks, and a ticking dashboard would misrepresent them.
- **Freestyle charter §10.2** invoked exactly once, for `recourse-ladder`, with a named `table`
  fallback that preserves the signature loop.
