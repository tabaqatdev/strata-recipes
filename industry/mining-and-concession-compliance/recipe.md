# Recipe — Mining & Concession Compliance: **"The Rule Cascade"** (Industry & Manufacturing)

> **RESEARCHED 2026-08-05.** First write-up of this solution folder. §1 is page-verified, §3/§4 are
> curl-verified live against `gis.conservation.ca.gov`, and the full evidence — every count, sum, field
> list, value distribution and trap — lives in the shared catalog,
> **`../../data_sources/data_sources_ca.md` § *MOL — the statewide mine register***. The design record
> and harvest notes live in the shared **`../APP-TEMPLATE-LIBRARY.md`** entry. This file is
> self-sufficient: silhouette, rejected rivals, `AppLayout`, connections, capability sweep and risks are
> all below.

Whether a mine may lawfully sell to a California public agency — and what the public record cannot show.

---

## 0.1 The working app in this folder — **built and driven**

```
cd app && node server.mjs      # → http://localhost:8031/
node test-cascade.mjs          # 122 assertions — the arithmetic, the live data, the chrome, the theme
node test-render.mjs           #  40 assertions — boots the shipped script and drives the loop
```

**162 assertions, all green 2026-08-05.** Self-contained, MapLibre from a CDN, no build step:

| File | |
|---|---|
| `app/index.html` | the app — layout, theme, two maps, chrome, rendering |
| `app/cascade.mjs` | **the pure logic** — config, the four rungs and their predicates, statistics, control tests, the evidence horizon, the glossary, the haul-radius maths |
| `app/layers.mjs` | basemaps, context layers, per-mode point colours, the popup field model — presentation kept out of the statutory logic |
| `app/test-cascade.mjs` | 122 assertions: live data, static guards, chrome, header, theme contrast |
| `app/test-render.mjs` | 40 assertions: boots the shipped script against a DOM + MapLibre stub and drives the loop |
| `app/server.mjs` | zero-dependency static server |

`cascade.mjs` is imported **unchanged** by the browser and the suite, so no number can reach the screen
that the suite has not asserted. See `app/README.md` for running, configuring and debugging it.

It fetches the register live (`MOLMines/0`, 1,975 rows in one request — CORS-open, no proxy) and carries
the PRC §2717(b) rulebook as config, not as literals. **Nothing is synthesized**: there is no upload leg
and no sample data, because the whole app is public-record.

The strongest assertions are §2 of the suite: for each of the four rungs it runs the rung's SQL against
the live service **and** the rung's client-side predicate over the fetched rows, and asserts the two
agree. That is what makes the cascade on screen the cascade the statute describes, rather than a
hand-tuned funnel.

**Driving it produced three corrections the written design had wrong.**

1. **The aggregate floor was measured against the wrong denominator.** The design said `PriProduct` is
   null on *23 of the 130*. It is null on **29** of the 130 — 23 is the figure over the *active-only* 95,
   carried onto the statutory denominator by mistake. The suite caught it on the first run. Corrected
   everywhere; the honest line is now **"71 of 130 declare an aggregate commodity; 29 declare none"**
   (the shared catalog, trap **M6**).
2. **The keyless-basemap guard was tripped by our own CSS.** The suite greps the shipped HTML for
   provider names (`mapbox`, `googleapis`, `api_key`, …) to prove no keyed basemap ever appears — and
   matched the app's own `.mapbox` layout class. Renamed to `.mapframe`. A guard that searches source
   text for a vendor's name will also find your own vocabulary; name UI classes so they cannot collide
   with the thing you are policing. The guard was kept at full strength rather than narrowed.
3. **A map-registration race dropped the first hydration.** `maps[id] = m` was assigned *after* the
   event handlers were registered, so a `styledata` that fired before `initMap` returned found no map in
   the registry and returned early, with nothing to retry. The map is now registered before any handler
   is attached, and hydration additionally runs on **`idle`** — so a style that resolves late, or never
   fires `load`, still gets its sources.

Running it in a browser found a fourth, and it was the worst of them:

4. **⚠ The splash could not be dismissed.** Reported as *"press 'Understood — show the cascade' or Esc
   and it doesn't close"* — the app was unusable past the first screen. Two causes, one of them a rule
   this codebase already knew:
   **(a)** `#splash` sets `display:flex`, and an **author `display` rule beats the UA's
   `[hidden]{display:none}`** — so setting `.hidden = true` changed the attribute and nothing else. The
   stylesheet already carried `.page[hidden]{display:none}` for exactly this reason on the page
   containers; the companion rule for the splash was simply never written. One line: `#splash[hidden]`.
   **(b)** **Esc was never wired at all.** The design said "dismissible"; nobody implemented the keyboard
   path. There is now one `closeSplash()` with three ways in — the button, `Escape`, and a backdrop click
   that ignores clicks on the card — plus focus moved to the card's button on open and back to the nav on
   close.
   The regression guard is **general, not a spot fix**: `test-cascade.mjs` §12 finds every element in the
   markup carrying a `hidden` attribute, works out whether any CSS rule sets a non-`none` `display` on its
   id or classes, and fails unless a matching `[hidden]` rule exists. Verified to bite — reverting the one
   CSS line turns the suite red. Adding a fourth dismissible element cannot repeat this.

A second browser pass reported three more, and they forced a **second test harness**:

5. **⚠ The map read as empty on first open.** Reported as *"no point on the map is shown"*. The wiring
   was never broken — `test-render.mjs` §1 confirms the source receives all 130 features on boot. It was
   **visual**: the map opened on a fixed centre/zoom with 3.6 px dots, so 130 operations spread over the
   whole state rendered as specks, and on a pale or slow-to-paint basemap that reads as nothing at all.
   Three changes: the map now **fits its data** the first time it becomes paintable, points are sized up
   (5.5 px at z4, 1.6 px halo), and the status readout carries a **drawn count** so an empty map *says*
   it is empty instead of merely looking broken.
6. **A rung was a one-way selection.** Clicking a rung narrowed the app; clicking it again did nothing,
   so there was no way back to the register except reloading. A rung is now a **toggle** — clicking the
   active one clears back to rung ①, matching the agency bars and the legend chips, which already
   toggled. (`test-render.mjs` §2.)
7. **A ledger row did not complete the master–detail move.** It jumped the camera and filled the
   dossier, but did not flash the operation or open its popup — so on a dense map you lost track of which
   point you had just selected. A row (in the ledger, the haul-radius table, or the search results) now
   **pans, flashes and opens the popup**; a map *click* still only opens the popup, because the user's eye
   is already on the point. Motion honours `prefers-reduced-motion`. (`test-render.mjs` §3.)

**None of items 5–7 were visible to `test-cascade.mjs`**, which greps the shipped markup but never runs
it. `test-render.mjs` boots the real inline module against a minimal DOM + MapLibre stub and drives the
loop, so "the map has features", "a rung toggles" and "a row flashes" are now assertions. All three were
verified to fail when their fix is reverted.

The map was **still empty** after item 5, because item 5 fixed the wrong cause. The real one:

8. **⚠ Hydration re-entered on every `idle`, so the map never settled.** Correction 3 had added `idle`
   as a third hydration trigger to close a hole where a late-resolving style never fired `load`. But
   `hydrate()` ends in `paint()`, `paint()` calls `setData`, `setData` makes the map re-render, and a
   finished render fires **`idle`** — which called `hydrate()` again. An unbounded loop: the map
   re-rendered forever, never settled, and therefore never painted. It looked like an empty map with a
   hot CPU, and *nothing about the symptom pointed at the hydration fix that caused it.*
   `installed[id]` now allows exactly one install per style, and `setBasemap` clears the flag because
   `setStyle` wipes every custom source and layer. A `m.resize()` on install also covers a container
   laid out after construction.
   **The first render harness could not see this either** — its stub fired `idle` once from a timer, so
   there was no render loop to enter. The stub now fires `idle` after every `setData`, exactly as a real
   map does, which is what makes the bug reproducible offline at all. Measured: **43 `setData` calls on
   boot without the guard (capped), 1 with it.**
9. **A ledger row was a one-way selection**, the same shape of bug as the rung in item 6 — clicking the
   selected row again did nothing. Rows in the ledger, the haul-radius table and the search results now
   toggle: a second click clears the highlight, closes the popup, empties the dossier and removes the
   flash ring. (`test-render.mjs` §4.)

10. **The map was reported empty a third time — basemap drawing, no operations.** At this point three
    hypotheses had each been asserted green offline and each failed to fix the browser, so the honest
    move was to stop hypothesising and make the app **say what is wrong**:
    - **Bounded self-heal.** `ensureDrawn()` re-installs and repaints at **250 / 800 / 2000 / 4500 ms**,
      then stops. Every hydration trigger has a hole (a late style, a container laid out after
      construction, a source dropped by a swap); a handful of timed checks closes all of them without
      adding a fourth event — which is precisely how the loop in item 8 got in. `add()` is idempotent,
      so a retry is free.
    - **`hydrate` can no longer fail silently.** It is wrapped; a throw now renders a dismissible
      banner *in the map frame* naming the error, instead of leaving an empty map and a console line
      nobody reads.
    - **The status readout reports what the map actually holds** — `painted[id]`, the count last handed
      to the source — as `"130 drawn"`, `"130 drawn of 874"` or `"layer not installed"`. Those three
      strings distinguish *no data*, *stale data* and *no layer*, which is the diagnosis the previous
      two rounds were missing.
    - **`window.__cascade`** exposes `state()` (per map: installed · styleLoaded · hasLayer · hasSource ·
      inSource · shouldHave · zoom · canvas size) and `redraw()`.

    **Stated plainly: the root cause in the reporting browser is still unconfirmed.** Items 5 and 8 were
    both real defects and both are fixed, but neither has been shown to be *the* cause of this symptom.
    What ships now is a build that recovers from the whole class of failure and reports the state needed
    to close it if it recurs.

A header/theme review found two more:

11. **The header had drifted from its own spec, and the ASCII skeleton had drifted from the JSON.**
    §2.1's `AppLayout` specifies the header filter over **three** fields — `LAS_Name`,
    `PriProduct`, `MineStatus` — and the build shipped two; **status was missing**. Added, stacking with
    the rung like the other two. Separately, §3's ASCII skeleton drew `[county ▾]` where the JSON says
    lead agency — there is no county field in the register at all, so the skeleton was describing a
    column that does not exist. Corrected to `[lead agency ▾][commodity ▾][status ▾]`, and the header
    is now asserted field-by-field against §4 (`test-cascade.mjs` §15).
12. **⚠ Dark mode was broken, and the fix exposed a colour-vision defect in *both* modes.**
    The map's point colours were **hardcoded light-mode ink**. On CARTO Dark Matter the eligible green
    (`#2e6f40`) measures **2.81:1** against the tiles — below the 3:1 WCAG non-text minimum, i.e.
    effectively invisible. Dark mode now takes the dark palette's lifted `success`/`danger` roles
    (5.92:1 and 5.96:1) and flips the halo from paper-white to panel-dark; the legend reads the same
    table so it cannot drift from the map.
    Writing the contrast test then surfaced a bigger problem: barred red and eligible green sit at
    **1.08:1 luminance contrast** — the textbook red/green confusion pair, indistinguishable under
    deuteranopia or protanopia (~8 % of men) and identical in greyscale print. The recipe's own exhibit
    is a **printed procurement record**, so this was a real failure, not a theoretical one. Barred and
    eligible are now separated by **fill density as well as hue** — barred solid, eligible a hollow ring
    — in the map paint and in the legend swatch. The assertion checks for the non-colour channel, not a
    luminance ratio between two hues, because a ratio is the wrong instrument for that question.

13. **⚠ The points vanished on every basemap and theme change.** `setStyle` fires `styledata`
    **synchronously and then drops every custom source and layer**. The persistent `styledata` →
    `hydrate` wire therefore re-installed *during* the swap, marked itself `installed`, and was
    immediately wiped — so the later, safe pass returned early on that flag and nothing came back.
    Fixed twice over, deliberately: a **`swapping[id]`** flag suppresses hydration while `setStyle` is
    in flight, and the **`once("idle")`** handler resets `installed` before re-installing, so whatever
    the swap left behind is rebuilt from scratch. The two are redundant — removing *either* alone still
    passes; removing **both** fails (38/2) — which is how the guard was confirmed to detect the bug
    rather than merely coexist with it. `ensureDrawn()` also stands down while swapping so the bounded
    retries cannot fight the transition. (`test-render.mjs` §5, with the stub firing `styledata`
    mid-swap exactly as MapLibre does.)

**The lesson worth keeping:** five of these thirteen corrections (3, 8, the `[hidden]` half of 4, and the
harness gap behind 10) were caused by *fixing the previous one*. Each fix was correct in isolation and
wrong in composition. Two follow-ons: a static suite cannot see a feedback loop between a fix and the
thing it fixed — hence the render harness; and when three offline-green hypotheses in a row fail against
the real thing, the next commit should add **observability**, not a fourth hypothesis.

`../APP-TEMPLATE-LIBRARY.md` carries an older entry for this recipe describing a build with 289
assertions. Its statutory and data claims were re-verified from scratch and reproduce, **except the
cascade population**, which this pass corrects (§1.2 and §1.2). That build is not in this
checkout; the app here is new.

**What is not built:** the `<StrataApp>` path. §2 specifies a full `AppLayout` (registry widgets,
`connections`, `sourceId` linking) and §2.1 serializes it — but rendering it needs
`strata/packages/*` installed and built, and `strata/node_modules` is absent from this checkout. The app
here is the standalone implementation of that same design, exactly as the sibling recipes ship theirs.
The `AppLayout` is authored and unrendered; the two must be reconciled before this recipe is called done.

---

## 0. Provenance

| | |
|---|---|
| **Source** | `https://tabaqat.net/solutions` — **Industry & Manufacturing** section |
| **Name on site** | Mining & Concession Compliance |
| **Folder** | `industry_mining-and-concession-compliance` |
| **Researched & designed** | 2026-08-05 |
| **Template** | **`open-design` — "rule-cascade"**. `zone-lookup` released back to the library (the site-lookup question survives as the subordinate page-2 dossier) |
| **Demo region** | California, statewide — extent `-124.16195784, 32.77639383 → -114.63140052, 41.88333836` |
| **Backend** | read-only. Strata Serve **or** ArcGIS. No write path anywhere in this app, by design |

---

## 1. Study — how the market frames this

### 1.1 The question the buyer asks

> *"We have a paving contract, the aggregate is coming from a pit up the valley, and procurement wants a
> sign-off before the trucks roll. Can this mine legally sell to a public agency today — and if not,
> what exactly is missing, and can I prove I checked?"*

That question has a statutory answer in California, and the answer is a **list**.

### 1.2 The rules that decide it

| Rule | Authority | Decides |
|---|---|---|
| A **state** agency shall not acquire or use minerals from an operation not on the List | **PCC §10295.5** | whether a state purchase is lawful |
| Contractors and operators may not sell sand, gravel, aggregate or other mined materials to a **local** agency if the operation is not on the List | **PCC §20676** | whether a local purchase is lawful |
| The Division of Mine Reclamation shall publish, **at least quarterly**, a list of all operations *reporting as **newly permitted, active, or idle*** meeting **all six** conditions | **PRC §2717(b)** | who is on the List |
| An appeal pending before the board < 180 days keeps an operator on the List **despite** failing (1) and (2) | **PRC §2717(c)** | why the List is not a pure conjunction |
| Annual report + fees | **PRC §2207** | conditions (4) and (5) |
| Lead agency's notice of completion of inspection | **PRC §2774(b)** | condition (3) — owned by one of **140** lead agencies |

**The six conditions of §2717(b)**, verbatim in §1.2 below:
(1) reclamation plan approved · (2) financial assurance mechanism approved and ≥ the current estimate ·
(3) FACE submitted per the lead agency's notice of completion of inspection · (4) annual report
submitted · (5) all fees, penalties and interest paid · (6) not out of compliance with an order to
comply.

**The detail most summaries drop, and it moves the numbers.** §2717(b) scopes the List to operations
*"reporting as newly permitted, active, or idle"* — **three** reporting statuses. Reading it as
*active* alone under-counts the barred population by **35 operations** (19 idle + 16 newly permitted)
and **$2,615,514.96** of financial assurance. This recipe ships the statutory reading.

### 1.3 The cascade, measured

Every count is a live query on 2026-08-05 (§1.3):

```
①  1,975   the register                        1=1
②    874   PRC §2717(b) preamble               MineStatus IN ('ACTIVE','IDLE','NEWLY PERMITTED')
③    160   PCC §10295.5 / §20676 — off list    … AND ABLIST LIKE 'OFF%'
④    130   filed a current report, barred      … AND ReportYear >= 2025

   the 130:  $12,950,937.46 financial assurance · 2,498.17 disturbed acres
             43 lead agencies · 71 of 130 declare an aggregate commodity (29 declare none)
             ACTIVE 95 · IDLE 19 · NEWLY PERMITTED 16
```

**Control tests — the cascade is not tautological.** **44** operations sit *on* the List with a stale
annual report, so the List is **not** a report-currency check. `RP_Number` is populated at **21.8 %** on
the listed side and **17.2 %** on the barred side, so it is **not** a plan-approval check — it measures
extract completeness, and was rejected as a cascade rung for that reason.
The flag is not reducible to either field — which is the whole reason the app renders the published
flag rather than recomputing it.

### 1.4 The evidence horizon

Of the six conditions, the public record makes **exactly one** directly readable — (4), the annual
report, via `ReportYear`/`ReportType`. Conditions (3), (5) and (6) live in the Division of Mine
Reclamation's own folder, which is token-gated:

```
$ curl -s "https://gis.conservation.ca.gov/server/rest/services/DMR?f=json"
{"error":{"code":499,"message":"Token Required","details":[]}}
```

(2) is an amount, not evidence of approval; (1) has no reliable field. **The app cannot recompute the
List, and says so on its face.** That admission is the product's most credible feature, not its
weakness — it tells the buyer exactly when to stop trusting the screen and call the lead agency.

### 1.5 Reference solutions (benchmark + coexist, never copy)

**The incumbent is the regulator's own app.** DOC/DMR publishes **Mines Online (MOL)** at
`maps.conservation.ca.gov/mol/` — active and historic mines, status, commodity, and a note of AB 3098
compliance, searchable by mine name, mineral or county. It is the authoritative public face of the
register and it is excellent at *"what is this mine's status?"*.

**Esri** positions GIS across the whole mine lifecycle (`esri.com/en-us/industries/mining`) —
exploration, permitting and land-use planning, operations, EHS, closure — with **Dashboards + Experience
Builder + StoryMaps** as the regulator pattern (New Mexico EMNRD's Mining and Minerals Division is the
cited reference: permit processing, activity tracking, reclamation enforcement). The shape on offer is a
permitting console, not a purchasing bar.

**Non-Esri — where this market actually is.** Land-tenure GRC and mine-planning suites:

| Vendor / product | Owns | Blind to |
|---|---|---|
| **Trimble Landfolio** (ex-Spatial Dimension) — the de-facto mining-cadastre standard; national cadastres plus the corporate side for Rio Tinto, Newmont, OZ Minerals, Samancor; built on the Esri platform | application → evaluation → grant → **compliance monitoring** of mineral, surface and water rights | it tracks the *licence*; a downstream purchasing prohibition is not a tenure object |
| **K2fly / Decipher** — enterprise SaaS *Natural Resource Governance*; SAP-endorsed, Esri partner | land access, ground disturbance, cultural heritage, tailings, rehabilitation obligations, auditable disclosure | operator-side only; nothing about a *buyer's* statutory bar |
| **Datamine · Maptek · Seequent · RPMGlobal · MICROMINE** | geology, resource modelling, scheduling, survey, financial modelling | compliance is peripheral; no public-register view |
| **Benchmark Gensuite · iPoint · Fair Supply · BanQu** | EHS/ESG compliance, conflict-minerals due diligence, supply-chain traceability | commodity-level ESG, not a jurisdiction's eligibility list |
| **Public cadastre portals** — Queensland **GeoResGlobe** (600+ layers, free, no registration) · WA **TENGRAPH** + MINEDEX + GeoVIEW + Mineral Titles Online · Peru **GEOCATMIN** (INGEMMET) | first-class *additive* tenement browsing — find a title, stack layers, check overlaps | all are lookup surfaces; you arrive knowing what to find |

**The gap.** Every product above — MOL included — is **additive**. Not one is **subtractive**: none
starts from the whole register and eliminates it down to the barred population, citing the rule that
performs each removal. So the oversight question ("how much supply does the bar remove, and who owns the
failure?") has no surface at all, and the procurement officer answers theirs one mine at a time with no
record that the check happened.

### 1.6 Our edge

AI-authored, MIT/open, runs on **Strata *or* ArcGIS**, sovereign/on-prem. And the structural one: **we
can show both the arithmetic and the gap, because we have nothing proprietary hidden inside either.** A
vendor whose compliance engine *is* the product cannot ship a panel saying "five of these six conditions
are not in public data". For a statute-driven bar, that is the most valuable thing on the page.

Coexistence: MOL stays the authoritative register view; Landfolio keeps the tenure; DMR keeps the List.

### 1.7 Standards, statutes & organizations to speak fluently

- **SMARA** — Surface Mining and Reclamation Act of 1975, PRC **§§2710–2796**. **§2717(b)/(c)** (the
  List), **§2207** (annual report + fees), **§2736** (financial assurance), **§2773.4** (FACE),
  **§2774(b)** (notice of completion of inspection), **§2770(e)** (appeals), **§2774.4** (board as lead
  agency).
- **Public Contract Code §10295.5** (state) and **§20676** (local) — the purchasing bar. *(Some
  secondary sources cite §20679; §2717(b) itself cross-references §10295.5 and §20676.)*
- **AB 3098 (1992)** — the bill that created the List and the name everyone uses for it.
- **DOC / DMR** — Department of Conservation, **Division of Mine Reclamation**: publishes the List, runs
  Mines Online. **SMGB** — State Mining and Geology Board: regulations, appeals, and **lead agency of
  last resort** (it is the lead agency on 9 of the 130). **CGS** — California Geological Survey: mineral
  land classification, Production-Consumption Regions.
- **Lead agencies** — 140 city/county jurisdictions administering SMARA locally; they own condition (3).
- **California Regulatory Notice Register** — where the List is actually published, at least quarterly.
- **Caltrans** — Construction Manual §3-6 *Control of Materials* points staff to the "AB 3098 SMARA
  Eligible List"; the separate **Aggregate Prequalification Program** and Authorized Materials List are
  a *different* test (materials quality) and must not be conflated with AB 3098 (legal eligibility).
- Adjacent vocabularies worth knowing but **out of scope here**: NI 43-101 / JORC (resource reporting),
  IRMA, TSM, the Copper Mark (responsible-sourcing assurance), EITI (revenue transparency).

### 1.8 Honest scope — what this is **not**

- **Not the AB 3098 List.** It renders the register's `ABLIST` flag as of the extract date. DMR's
  quarterly publication governs.
- **Not a compliance determination.** Five of six conditions are not public (§1.4).
- **Not a tenure or cadastre system.** The register publishes **points**; `PermitAcre` is a `String`. No
  permit boundary is drawn anywhere in this app.
- **Not a siting or hazard tool.** Farmland, Williamson Act and seismic layers sit in the same server and
  are deliberately excluded — different question.
- **Not positionally precise.** DMR cannot guarantee operator-reported coordinates; no distance claim is
  made tighter than a haul-radius screen.
- **Not a materials-quality check.** AB 3098 is about legal eligibility, not aggregate specification.

---

## 2. UI design spec (front-loaded)

### 2.1 Layout (Template: **`open-design` — "rule-cascade"**)

- **Template** `open-design` under the freestyle charter (`strata/recipes/COMPONENT-MANIFEST.md` §10):
  registry widgets and manifest config keys only, plus **one** §10.3 app-local widget with a named
  fallback (§2.6). **Never** fall back to `split-dashboard`.
- **Why this shape:** the purpose sentence is a *subtraction*, so the navigation is a subtraction — a
  **rail of four rungs, each one a `definitionExpression` that cites the rule performing the removal**,
  showing survivors, the count removed, and the SQL. The app teaches itself in about five seconds
  because the first screen already reads 1,975 → 874 → 160 → 130.
- **Signature loop:** `rung → map repaints + ledger narrows + agency bars recompute → click the tallest
  agency bar → its barred mines flash → click a row → dossier → export the CSV as the procurement
  record.`
- **The signature accent is the terminal panel** — the **evidence horizon**, which names the five
  §2717(b) conditions the public record cannot see and shows the `499 Token Required` that proves it.
  The app ends in an admission, not a score.
- **Two pages:** `cascade` (the population — three of four personas) and `supply` (the haul-radius
  dossier — the one-mine procurement question).
- **Three drag handles** per page (rail ⇄ map, map ⇄ horizon, board ⇄ ledger).
- **Wiring floor:** **21 connections** authored, ≥3 required (`docs/guide/app-design.md` §3). Seven more
  widgets link with **zero** connections through `dataSource.sourceId`, two through `fromWidget`.
- **Responsive:** `responsive.small` collapses both splitters to a column, rail first (`dock:"top"`,
  rungs as a `flow-row` of chips), map at 45 vh, KPI strip to chips. The map keeps a **240 px floor** —
  below that MapLibre can silently fail to finish loading a style, and a blank map in a compliance tool
  reads as "no data" rather than "no room".

#### The two rivals, rejected on integrity grounds

Both were designed for this business and both die on the data, not on taste.

- **`permit-envelope`** — each operation drawn as its permitted area, coloured by list position. The most
  legible shape and the closest to what mining buyers expect from Landfolio or GeoResGlobe. **It dies
  because the register publishes *points*.** There is no permit polygon anywhere in `MOL`, and
  `PermitAcre` is a `String(200)`. Building the envelope means parsing that string and buffering a
  self-reported point into a circle — fabricating the geometry the entire app rests on, on a layer whose
  own publisher disclaims coordinate accuracy.
- **`custody-chain`** — six lanes, one per §2717(b) condition, each mine a token on the lane where its
  compliance breaks. Maps directly onto the statute and would be the most actionable view for an
  operator. **It dies because five of the six conditions are unreadable.** Every token would be placed
  by inference, and an interface that guesses which legal obligation an operator breached is
  defamatory-by-interface. The idea survives, inverted, as the evidence horizon.

#### ASCII skeleton — page 1 `cascade` (desktop)

```
┌ HEADER ────────────────────────────────────────────────────────────────────────────────────┐
│ Mining & Concession …the rule cascade  [lead agency ▾][commodity ▾][status ▾] ⟨1⟩⟨2⟩ ☀/☾ │
│ AB 3098 flag as published on the MOL register · extract 2026-08-05 · republished ≥ quarterly│
├──────────────┬─────────────────────────────────────────────┬───────────────────────────────┤
│ CASCADE RAIL │                                             │  EVIDENCE HORIZON             │
│  (the nav)   │                 MAP  (boxed)                │  PRC §2717(b) — 6 conditions  │
│              │                                             │  (1) recl. plan      ✕ not in │
│ ① 1,975      │   survivors solid · eligible hollow ring    │  (2) fin. assurance  ✕  public│
│   register   │   removed 16% ghost · click → popup         │  (3) FACE/inspection ⛔ DMR 499│
│ ② 874  −1,101│                                             │  (4) annual report   ✅ ReportYr│
│   §2717(b)   │   ┌ legend ─────────┐        ┌ ctl cluster ┐│  (5) fees paid       ⛔ DMR 499│
│ ③ 160   −714 │   │ ● barred    130 │        │ + − ⌂ ≡ ▤ ☰ ││  (6) order to comply ⛔ DMR 499│
│   §10295.5   │   │ ○ eligible    0 │        └─────────────┘│  (c) appeal <180d    ✕        │
│   /§20676    │   │ ◌ removed 1,845 │                       │  ── 1 of 6 readable ──        │
│ ④ 130    −30 │   └─────────────────┘   lon,lat · z · EPSG  ├───────────────────────────────┤
│   §2717(b)(4)│                                             │  CONTROL TESTS · THE STATUTE  │
├──────────────┴─────────────────────────────────────────────┴───────────────────────────────┤
│  130 barred │ $12,950,937 assurance │ 2,498.17 ac │ 43 agencies │ 71 of 130 aggregate │ 18.3%│
├──────────────────────────────┬──────────────────────────────────────────────────────────────┤
│ BY LEAD AGENCY (bar, top 12) │ LEDGER — Mine_ID · Name · Operator · Status · ABLIST · Report │
│ San Bernardino ████ 16       │  Year · FaceAmount · Acres_Dist · LAS_Name · PriProduct       │
│ Kern           ███  12       │  row → pan + flash + popup · click again to clear · CSV export│
└──────────────────────────────┴──────────────────────────────────────────────────────────────┘
```

Page 2 `supply` — the haul-radius dossier: `job panel | map + ring | dossier` over a
sources-within-radius table, eligible first.

**Phone (`responsive.small`).** Both splitters collapse to a column. Page 1 order: rail (as
`panel dock:"top"`, rungs as a `flow-row` of chips) → map at 45 vh → KPI chips → horizon → ledger. The
map keeps a **240 px floor** — below that MapLibre can silently fail to finish loading a style, and a
blank map in a compliance tool reads as "no data" rather than "no room".

#### The `AppLayout` JSON (authored; see §0.1 for what is and is not rendered)

```jsonc
{
  "version": "1",
  "theme": { /* §2.2 */ },
  "splash": { "title": "This is not the AB 3098 List",
    "body": "It renders the ABLIST flag published on the state mine register, extracted 2026-08-05. The Division of Mine Reclamation republishes the List at least quarterly in the California Regulatory Notice Register, and that document governs. Five of the six PRC §2717(b) conditions are not in public data — the app names them rather than guessing.",
    "dismissible": true, "once": true },
  "pages": [
    { "id": "cascade", "type": "fixed",
      "header": { "kind": "row", "gap": 12, "children": [
        { "kind": "widget", "widget": { "id": "hdr", "type": "text",
            "props": { "content": "Mining & Concession Compliance — the rule cascade", "as": "h3" } } },
        { "kind": "widget", "widget": { "id": "hdr-vintage", "type": "text",
            "props": { "content": "AB 3098 flag as published on the MOL register · extract 2026-08-05", "as": "body2" } } },
        { "kind": "widget", "widget": { "id": "flt", "type": "filter",
            "props": { "layerId": "mines", "fields": [
              { "name": "LAS_Name",   "label": "Lead agency", "type": "string" },
              { "name": "PriProduct", "label": "Commodity",   "type": "string" },
              { "name": "MineStatus", "label": "Status",      "type": "string" } ] } } },
        { "kind": "widget", "widget": { "id": "nav",      "type": "page-nav",     "props": { "variant": "tabs" } } },
        { "kind": "widget", "widget": { "id": "theme-sw", "type": "theme-switch", "props": { "initial": "light" } } }
      ]},
      "root": { "kind": "splitter", "orientation": "v", "sizes": [66, 34], "minSizes": [30, 15], "children": [
        { "kind": "splitter", "orientation": "h", "sizes": [22, 52, 26], "minSizes": [14, 30, 16], "children": [
          { "kind": "panel", "dock": "left", "title": "The cascade", "width": 300,
            "responsive": { "small": { "dock": "top" } }, "children": [
            { "kind": "widget", "widget": { "id": "cascade", "type": "rule-cascade",
                "dataSource": { "sourceId": "mines" },
                "props": { "layerId": "mines", "showSql": true, "asOf": "2026-08-05", "steps": [
                  { "id": "all",     "label": "The register", "cite": "—", "where": "1=1" },
                  { "id": "universe","label": "Newly permitted, active or idle", "cite": "PRC §2717(b)",
                    "where": "MineStatus IN ('ACTIVE','IDLE','NEWLY PERMITTED')" },
                  { "id": "offlist", "label": "Off the AB 3098 List", "cite": "PCC §10295.5 · §20676",
                    "where": "ABLIST LIKE 'OFF%'" },
                  { "id": "filed",   "label": "Filed a current annual report — and barred anyway",
                    "cite": "PRC §2717(b)(4)", "where": "ReportYear >= 2025" } ] } } } ]},
          { "kind": "section", "mode": "fixed", "children": [
            { "kind": "widget", "widget": { "id": "map", "type": "map",
                "props": { "config": { "$ref": "layers.json" },
                  "controls": { "navigation": true, "scale": true, "legend": true,
                                "basemapSwitcher": true, "layerList": true } } } },
            { "kind": "widget", "widget": { "id": "statusbar", "type": "status-bar",
                "props": { "crs": "EPSG:4326", "precision": 5 } } } ]},
          { "kind": "panel", "dock": "right", "title": "Evidence horizon", "width": 340, "children": [
            { "kind": "widget", "widget": { "id": "horizon", "type": "table",
                "props": { "oidField": "id", "viewportHeight": 240,
                  "columns": ["cond", "requirement", "public evidence", "readable"],
                  "rows": [ /* the seven rows of §1.4 — six conditions + the §2717(c) override */ ] } } },
            { "kind": "accordion", "titles": ["Control tests", "The statute, verbatim"], "children": [
              { "kind": "widget", "widget": { "id": "controls", "type": "text", "props": { "as": "body2", "content": "…" } } },
              { "kind": "widget", "widget": { "id": "statute",  "type": "text", "props": { "as": "body2", "content": "…" } } } ]} ]}
        ]},
        { "kind": "column", "gap": 8, "children": [
          { "kind": "flow-row", "gap": 8, "children": [
            { "kind": "widget", "widget": { "id": "kpi-barred", "type": "kpi",
                "props": { "label": "Barred", "stat": { "field": "OBJECTID", "op": "count" } },
                "dataSource": { "sourceId": "mines" } } },
            { "kind": "widget", "widget": { "id": "kpi-assurance", "type": "kpi",
                "props": { "label": "Financial assurance", "unit": "$", "stat": { "field": "FaceAmount", "op": "sum" } },
                "dataSource": { "sourceId": "mines" } } },
            { "kind": "widget", "widget": { "id": "kpi-acres", "type": "kpi",
                "props": { "label": "Disturbed acres", "stat": { "field": "Acres_Dist", "op": "sum" } },
                "dataSource": { "sourceId": "mines" } } },
            { "kind": "widget", "widget": { "id": "kpi-agencies", "type": "kpi",
                "props": { "label": "Lead agencies", "stat": { "field": "LAS_CODE", "op": "count" } },
                "dataSource": { "sourceId": "mines" } } },
            { "kind": "widget", "widget": { "id": "gauge-share", "type": "gauge",
                "props": { "label": "% of the §2717(b) universe barred", "thresholds": [10, 25],
                           "stat": { "field": "OBJECTID", "op": "count" } },
                "dataSource": { "sourceId": "mines" } } },
            { "kind": "widget", "widget": { "id": "acts", "type": "data-actions", "props": { "hideWhenEmpty": true } } } ]},
          { "kind": "splitter", "orientation": "h", "sizes": [32, 68], "children": [
            { "kind": "widget", "widget": { "id": "agencies", "type": "chart",
                "props": { "charts": [ { "id": "byagency", "title": "Barred by lead agency", "kind": "bar",
                  "source": { "layer_id": "mines", "field": "LAS_Name", "stat": "count" } } ], "limit": 12 },
                "dataSource": { "sourceId": "mines" } } },
            { "kind": "widget", "widget": { "id": "ledger", "type": "table",
                "props": { "virtualize": true, "viewportHeight": 260, "oidField": "OBJECTID",
                  "columns": ["Mine_ID","MineName","Operator","MineStatus","ABLIST","ReportYear",
                              "FaceAmount","Acres_Dist","LAS_Name","PriProduct"] },
                "dataSource": { "sourceId": "mines" } } } ]} ]}
      ]},
      "footer": { "kind": "row", "gap": 12, "children": [
        { "kind": "widget", "widget": { "id": "attrib", "type": "text", "props": { "as": "body2",
            "content": "Source: CA Dept. of Conservation, Division of Mine Reclamation — Mines Online register. The published quarterly AB 3098 List governs." } } },
        { "kind": "widget", "widget": { "id": "share-w", "type": "share" } },
        { "kind": "widget", "widget": { "id": "print-w", "type": "print" } } ]} },

    { "id": "supply", "type": "fixed",
      "root": { "kind": "splitter", "orientation": "h", "sizes": [24, 50, 26], "children": [
        { "kind": "panel", "dock": "left", "title": "Where is the job?", "children": [
          { "kind": "widget", "widget": { "id": "nearme", "type": "near-me", "props": { "title": "Haul radius", "defaultKm": 40 } } },
          { "kind": "widget", "widget": { "id": "buffer", "type": "analysis", "props": { "tools": ["buffer"] },
              "dataSource": { "sourceId": "mines" } } },
          { "kind": "widget", "widget": { "id": "kpi-radius", "type": "kpi",
              "props": { "label": "Sources in radius", "stat": { "field": "OBJECTID", "op": "count" } },
              "dataSource": { "fromWidget": "buffer" } } },
          { "kind": "widget", "widget": { "id": "q", "type": "query",
              "props": { "fields": [ { "name": "MineName", "label": "Mine name", "type": "string" },
                                     { "name": "Operator", "label": "Operator",  "type": "string" },
                                     { "name": "Mine_ID",  "label": "Mine ID",   "type": "string" } ] },
              "dataSource": { "sourceId": "mines" } } } ]},
        { "kind": "section", "mode": "fixed", "children": [
          { "kind": "widget", "widget": { "id": "map2", "type": "map",
              "props": { "config": { "$ref": "layers.json" },
                         "controls": { "navigation": true, "scale": true, "legend": true } } } } ]},
        { "kind": "panel", "dock": "right", "title": "Dossier", "children": [
          { "kind": "widget", "widget": { "id": "dossier", "type": "feature-info",
              "props": { "emptyText": "Pick a source in the radius, the table or the map." },
              "dataSource": { "sourceId": "mines" } } },
          { "kind": "widget", "widget": { "id": "conditions", "type": "table",
              "props": { "oidField": "id", "columns": ["cond", "what the record shows"] } } } ]}
      ]},
      "footer": { "kind": "row", "children": [
        { "kind": "widget", "widget": { "id": "within", "type": "table",
            "props": { "virtualize": true, "viewportHeight": 200, "oidField": "OBJECTID",
              "columns": ["Mine_ID","MineName","Operator","ABLIST","PriProduct","LAS_Name"] },
            "dataSource": { "fromWidget": "buffer" } } },
        { "kind": "widget", "widget": { "id": "print2", "type": "print" } } ]} }
  ],
  "connections": [ /* §2.7 */ ]
}
```

### 2.2 Theme

Structured `ThemeSpec`, **light** — a counter document, not a control room — with a real dark mode behind
`theme-switch`. Every exhibit prints light whichever mode is on screen.

```jsonc
{ "mode": "light",
  "colors": { "primary": "#3d5a6c", "secondary": "#6b7a83", "success": "#2e6f40", "info": "#3d5a6c",
              "warning": "#a86a00", "danger": "#b3261e", "light": "#f6f4f0", "dark": "#1d2429" },
  "fonts": { "scale": "default", "mono": "ui-monospace, Menlo, monospace" },
  "variables": { "--strata-radius-md": "4px", "--strata-motion-base": "140ms",
                 "--strata-elevation-1": "0 1px 2px rgba(29,36,41,.10)" },
  "overrides": { "kpi": { "--strata-mono": "ui-monospace, Menlo, monospace" } } }
```

Semantic status is load-bearing and binary: `success` = **on the List**, `danger` = **barred**. Those are
the only saturated colours on the page. Basemap keyless and mode-paired (`basemapForTheme` → CARTO
Positron / Dark Matter). **Eliminated rows stay on the map at ~16 % opacity** rather than vanishing —
subtraction has to be visible to be persuasive. `esriSMSCircle`/`esriSMSSquare` only, never
`esriSMSPath`; polygon context at ~40/255 alpha. Tabular monospaced numerals wherever a figure meets a
rule. **No container `animate`** — a legal instrument that slides is one you distrust. **English only**
(§2.5).

**Map ink is theme-aware, and the light values may not be reused on dark.** The point colours live in one
table (`POINT_COLORS` in `app/layers.mjs`), keyed by mode, and the legend reads the same table so it
cannot drift from the map:

| | barred | eligible | halo | ghost |
|---|---|---|---|---|
| **light** | `#b3261e` (5.74:1 vs Positron) | `#2e6f40` (5.32:1) | `#fffefb` | `#6b7a83` |
| **dark** | `#e8776d` (5.92:1 vs Dark Matter) | `#5aa970` (5.96:1) | `#11171b` | `#9aa8b0` |

Reusing the light green on a dark basemap measures **2.81:1** — below the 3:1 WCAG non-text minimum, i.e.
invisible. Each mode is asserted against its own basemap.

**Never encode barred vs eligible by hue alone.** Red and green sit at **1.08:1** luminance contrast —
the textbook red/green confusion pair, indistinguishable under deuteranopia or protanopia (~8 % of men)
and identical in greyscale. Since this recipe's exhibit is a **printed** procurement record, that is a
live failure, not a theoretical one. The two are therefore separated by **fill density as well as
hue** — barred solid, eligible a hollow ring — in the map paint *and* the legend swatch. The suite
asserts the non-colour channel exists, not a luminance ratio between two hues, because a ratio is the
wrong instrument for that question.

### 2.3 KPI cards

Six, all live `kpi.stat` / `gauge.stat` bound to the shared `dataSource.sourceId: "mines"`, so they
update on every rung and filter with **no `connections`**:

| Card | `stat` | At the terminal rung |
|---|---|---|
| **Barred** | `{ field: "OBJECTID", op: "count" }` | **130** *(label reads "Operations" off the terminal rung)* |
| **Financial assurance** | `{ field: "FaceAmount", op: "sum" }` | **$12,950,937.46** |
| **Disturbed acres** | `{ field: "Acres_Dist", op: "sum" }` | **2,498.17** |
| **Lead agencies** | `{ field: "LAS_CODE", op: "count" }` distinct | **43** |
| **Aggregate commodity** | allow-list over `PriProduct` | **71**, sub-labelled *"of 130; 29 declare none"* (M6) |
| **% of eligible universe barred** (`gauge`) | `{ field: "OBJECTID", op: "count" }`, `thresholds: [10, 25]` | **160 / 874 = 18.3 %** |

Plus `kpi-radius` on page 2, bound `fromWidget: "buffer"` — the count of sources inside the haul radius,
computed at runtime by the `analysis` buffer.

### 2.4 Charts & tables

- **`chart` — "Barred by lead agency"**: bar, `field: "LAS_Name"`, `stat: "count"`, `limit: 12`, over the
  `mines` source. Emits `categorySelect` → `filter` + `flash`. Verified top rows: County of San
  Bernardino **16** (446.3 ac) · County of Kern **12** (874.95 ac) · State Mining & Geology Board **9** ·
  Plumas / Inyo / Humboldt **7** each.
- **`table` — the ledger** (the protagonist): `virtualize: true`, `viewportHeight: 260`,
  `oidField: "OBJECTID"`, columns `Mine_ID · MineName · Operator · MineStatus · ABLIST · ReportYear ·
  FaceAmount · Acres_Dist · LAS_Name · PriProduct`. A row click is the full master–detail move —
  **pan + flash + popup + dossier** — and a **second click on the same row clears it** (highlight,
  popup, dossier and flash ring), matching the rungs, the agency bars and the legend chips. A map
  *click* only opens the popup: the user's eye is already on the point. **The CSV export is the
  procurement record** and carries the extract date and the active filter in its first line.
- **`table` — the evidence horizon**: static `rows` (no layer), columns `cond · requirement · public
  evidence · readable`. Seven rows: conditions (1)–(6) plus the §2717(c) appeal override.
- **`table` — sources within haul radius** (page 2): `fromWidget: "buffer"`, eligible first.
- **`carto`** is available and apt (commodity category + assurance formula) but **held back** so the
  ledger stays the protagonist — one brushable surface, not three.

### 2.5 Capabilities to use

Layout `splitter` + `panel` + `accordion` + `flow-row` + `section mode:"fixed"` · widgets `map legend
table chart filter query kpi gauge feature-info data-actions analysis near-me text status-bar share
print theme-switch page-nav` · triggers `filterChange categorySelect rowSelect featureSelect
extentChange recordsChange flash` · actions `filter zoomTo flash viewInTable showStatistics setUrlParam
message` · `dataSource.sourceId` linking (7 widgets, no wires) and `fromWidget` (2) ·
`@strata/processing` **buffer** · Arcade `valueExpression` on the mines renderer · structured theme ·
app-shell (`header`/`footer`/`splash`).

**Deliberately not used** — reasons, not omissions: **time slider** (`ReportYear` is a filing year, not
an event date — animating it would show mines "appearing" as paperwork lands); **`date-filter`** (same
reason: it is a `SmallInteger`, not a date field); **routing/isochrones** (haul *cost* needs a truck
model and a rate table we do not have — the radius ships labelled as a straight-line screen);
**search/geocoding** (you do not geocode a register); **`weighted-overlay`** (the weights are in statute,
not in the user's opinion); **hexbin/hotspot** (density over 130 points statewide is noise);
**overlay/dissolve/clip** (no polygons to combine); **`views`/`mapState`** (the cascade *is* the state
machine); **`RestDataSource`/`StreamDataSource`** (the List republishes quarterly; a live-looking widget
would be theatre); **`animate`/`autoPlay`**; **atlas**; **AR/RTL** (the audience is Californian
procurement and 140 SMARA lead agencies, and the binding statutes are English-only — the string table is
kept so every phrase lives in one place, but one locale ships).

**Not wired, and why** — `buttonClick`, `mapClick`, `sketchComplete`, `viewChange`, `pageChange`,
`timer`: the trigger types and dispatchers ship, but the **widget emitters are pending** (Manifest §4.1).
Consequently this design contains **no `window`** — a modal whose only opener is a `buttonClick` could
never be opened, so the statute text lives in an always-reachable `accordion`. Likewise no `navigate` /
`refresh` / `selectByGeometry` / `updateRecord`: the dispatchers ship but `<StrataApp>`'s callbacks are
pending. A connection that never fires is worse than none.

**No write path exists anywhere in this app**, by design — nothing here should ever write to a
compliance register. So there is no `assertEsriBackend` guard and no read-only degradation question: it
runs identically on Strata Serve and ArcGIS.

#### Capability sweep

| Capability | Where this design uses it — or why not |
|---|---|
| `splitter` | ✅ three, both pages (rail⇄map⇄horizon, board⇄ledger) |
| `panel` | ✅ cascade rail, evidence horizon, job panel, dossier — `dock:"top"` on phones |
| `accordion` · `flow-row` · `section mode:"fixed"` | ✅ control tests + statute · the KPI strip · the map box |
| `window` | ❌ its only opener (`buttonClick`) is a pending emitter — the statute text went to the accordion |
| `views` + `mapState` | ❌ the cascade **is** the state machine; a second one would fight it |
| `grid`, `card` | ❌ nothing here is a gallery |
| multi-page · scroll page | ✅ two (`cascade`, `supply`) · ❌ operational tool, not editorial |
| `animate` / `autoPlay` | ❌ deliberate: a legal instrument that slides is one you distrust |
| `map` | ✅ ×2, bus sink with `bus`+`store` |
| `table` | ✅ ×4 — ledger, horizon, within-radius, per-mine conditions |
| `chart` | ✅ barred-by-lead-agency bar, emits `categorySelect` |
| `carto` | ◻ available and apt (commodity category + assurance formula); held back so the ledger stays the protagonist — one brushable surface, not three |
| `filter` · `query` | ✅ header agency/commodity/status · page-2 name/operator/ID search publishing an output |
| `date-filter` | ❌ `ReportYear` is a `SmallInteger` filing year, not a date field — a calendar over it would be a lie |
| `kpi` + `gauge` (`stat`) | ✅ five, all live, all zero-connection |
| `sparkline`, `stacked-bar` | ❌ no honest time series (see time slider) |
| `feature-info` · `data-actions` | ✅ the dossier, tracking the source's selection · zoom/flash/export/clear |
| `legend`, `status-bar` | ✅ both. The legend **is** the layer switch — barred / eligible / removed, live counts, clickable. Status readout in EPSG:4326 with a drawn count |
| `basemap`, `layer-panel` | ✅ a drawer pair on the map's one control cluster (`L` / `B`); 5 keyless basemaps + 2 lazy-loaded context layers |
| popup | ✅ click a mine — a `popupInfo`-shaped field list that never shows `OBJECTID`, repeats the five-of-six caveat, links to the page-2 dossier |
| `share`, `print`, `theme-switch`, `page-nav`, `text` | ✅ all |
| `button` | ✅ ships **unwired**, used only as a link (`href`) — `buttonClick` is pending |
| `near-me` · `analysis` (buffer) | ✅ page-2 haul radius (callback-driven) · publishing its result as an output two widgets consume |
| `weighted-overlay` | ❌ the weights are in statute, not in the user's opinion |
| `swipe`, `bookmarks`, `controller` | ❌ no A/B question; two pages don't need a dock |
| `add-data`, `elevation`, `embed`, `video`, `image`, `menu`, `list`/`gallery` | ❌ not this app |
| `draw` / `measure` | ❌ measuring between self-reported points would imply a precision the source disclaims |
| **Triggers used** | `filterChange` · `categorySelect` · `rowSelect` · `featureSelect` · `extentChange` · `recordsChange` · `flash` |
| **Triggers not used** | `buttonClick` `timer` `viewChange` `pageChange` `sketchComplete` `mapClick` `countChange` — **emitters pending**; `rangeSelect`/`brush` — no continuous variable worth brushing; `hover` — no widget here emits it; `search` — you don't geocode a register |
| **Actions used** | `filter` · `zoomTo` · `flash` · `viewInTable` · `showStatistics` · `setUrlParam` · `message` |
| **Actions not used** | `panTo` (zoomTo is right for points) · `showHide` (no window) · `export` (native to `table`) · `navigate`/`refresh`/`selectByGeometry`/`updateRecord` — **`<StrataApp>` callbacks pending** |
| DataSource linking | ✅ `sourceId` on 7 widgets, `fromWidget` on 2 |
| `FileDataSource` | ◻ offered for the client's own approved-supplier list, to diff against the register — optional |
| `RestDataSource` / `StreamDataSource` | ❌ the List republishes **quarterly**; a live-looking widget would be theatre |
| `@strata/processing` | ✅ `buffer`. ❌ hexbin/hotspot (density over 130 points statewide is noise) · overlay/dissolve/clip (no polygons to combine) |
| Plugins | ✅ status bar. ❌ search/geocoding · routing/isochrones (haul *cost* needs a truck model and rate table we do not have) · **time slider** (`ReportYear` is a filing year — animating it would show mines "appearing" as paperwork lands) |
| Export / atlas / report | ✅ `print` exhibit + `table` CSV as the procurement record. ❌ atlas |
| Arcade | ✅ one `valueExpression` on the mines renderer, **trimming `ABLIST` first** (M1) |
| Structured theme · app-shell | ✅ full `ThemeSpec` · `header`/`footer`/`splash` |
| i18n EN + AR/RTL | ◻ **EN only** — the audience is Californian procurement and 140 SMARA lead agencies, and the binding statutes are English-only. The string table survives so every phrase lives in one place |
| Responsive | ✅ every side-by-side row has `responsive.small`; 240 px map floor |
| Writes / ESRI backend | ❌ **none, by design** — nothing here should ever write to a compliance register, so there is no `assertEsriBackend` path and no degradation question |

### 2.6 The one new widget (§10.2/§10.3) and its day-1 fallback

| Key | Purpose | Emits | Fallback that ships day 1 |
|---|---|---|---|
| **`rule-cascade`** | ordered rungs, each a named rule that *removes* rows — citation · survivors · removed · the cumulative `definitionExpression` | `filterChange` `{layerId, where}` (cumulative; `whereFromTrigger` derives every downstream filter with no extra options) | a **`chart`** (bar, one category per step, descending — already emits `categorySelect`) + a **`text`** block per step with citation and SQL + the header **`filter`** preloaded with the four expressions |

The evidence horizon, control tests and per-mine condition list use **shipped widgets only** (`table`
with static `rows` + `text` + `accordion`) — most "missing" widgets are an existing widget wired
differently, and those were.

**Full contract (§10.3):**

| | |
|---|---|
| **Registry key** | `rule-cascade` |
| **Purpose** | Ordered rungs, each a named rule that *removes* rows; shows the citation, the surviving count, the count removed, and the cumulative `definitionExpression` it produces. The navigation **is** the argument. |
| **Delivery** | App-local first — `<StrataApp registry={{ "rule-cascade": RuleCascade }}>` via `mergeRegistry`. No core change. |
| **Props** | `steps: { id, label, cite, where, note? }[]` (**required**) · `layerId` (**required**) · `sourceId?` · `showSql?` (default `true`) · `asOf?` (stamped on every rung) · `initialStep?` (default the last) |
| **Emits** | **`filterChange`** `{ layerId, where }` — the *cumulative* SQL for the active rung, `source` = its own `id`. `whereFromTrigger` already derives the target SQL from this payload, so every `filter` wire works with no extra options. |
| **Honors** | `filter` (external filters stack — the rung recomputes its counts within the incoming `where`), `clear` (returns to rung ①) |
| **dataSource** | `{ sourceId }` or `{ layerId }` — reads the bound source's filtered view to compute each rung's count; never fetches around the source |
| **Visual** | `--strata-*` tokens only; the bar uses `--strata-primary`, the removed count `--strata-secondary`, the terminal rung `--strata-danger`. RTL-safe (the bar mirrors with `direction`) |
| **Interaction** | A rung is a **toggle** — clicking the active one clears back to rung ① |
| **Fallback shipping day 1** | a **`chart`** (bar, one category per step, descending — it already emits `categorySelect`) beside a **`text`** block per step carrying the citation and the SQL, plus the header **`filter`** preloaded with the four expressions. Same four numbers, same wiring shape, none of the subtractive framing. |

### 2.7 `connections` (21 — every wire uses a shipped emitter)

| # | from | trigger | to | action | options | User-visible behaviour |
|---|---|---|---|---|---|---|
| 1 | `cascade` | `filterChange` | `map` | `filter` | `{layerId:"mines"}` | click a rung — the map keeps only the survivors |
| 2 | `cascade` | `filterChange` | `ledger` | `filter` | | the ledger narrows to the same rows |
| 3 | `cascade` | `filterChange` | `agencies` | `filter` | | the agency bars recompute for that rung |
| 4 | `cascade` | `filterChange` | — | `setUrlParam` | `{param:"where"}` | the rung is deep-linkable |
| 5 | `cascade` | `filterChange` | — | `message` | `{level:"info"}` | the applied SQL echoes in the status line |
| 6 | `agencies` | `categorySelect` | `map` | `filter` | `{layerId:"mines"}` | click *County of Kern* — map shows only Kern's barred set |
| 7 | `agencies` | `categorySelect` | `ledger` | `filter` | | ledger follows |
| 8 | `agencies` | `categorySelect` | `map` | `flash` | `{layerId:"mines"}` | that agency's mines pulse once |
| 9 | `flt` | `filterChange` | `map` | `filter` | `{layerId:"mines"}` | header agency/commodity/status filter drives the map; **stacks** with the rung |
| 10 | `flt` | `filterChange` | `ledger` | `filter` | | and the ledger |
| 11 | `flt` | `filterChange` | — | `setUrlParam` | `{param:"filter"}` | shareable |
| 12 | `ledger` | `rowSelect` | `map` | `zoomTo` | `{layerId:"mines"}` | pick a row — the map pans to that mine |
| 13 | `ledger` | `rowSelect` | `map` | `flash` | `{layerId:"mines"}` | and it pulses |
| 14 | `ledger` | `rowSelect` | `dossier` | `viewInTable` | | its dossier fills |
| 15 | `map` | `featureSelect` | `dossier` | `viewInTable` | | click a mine on the map — same dossier |
| 16 | `map` | `extentChange` | `kpi-barred` | `showStatistics` | | pan/zoom — the barred count recomputes for the view |
| 17 | `map` | `extentChange` | `kpi-acres` | `showStatistics` | | disturbed acres follow the extent |
| 18 | `acts` | `featureSelect` | `map` | `zoomTo` | `{layerId:"mines"}` | "Zoom to" on a selection |
| 19 | `acts` | `flash` | `map` | `flash` | `{layerId:"mines"}` | "Flash" |
| 20 | `buffer` | `recordsChange` | `kpi-radius` | `showStatistics` | | page 2: run the haul buffer — the in-radius count updates |
| 21 | `within` | `rowSelect` | `map2` | `zoomTo` | `{layerId:"mines"}` | pick a source in the radius — map 2 pans to it |

**Plus seven widgets that link with *zero* connections** via `dataSource.sourceId: "mines"` —
`kpi-barred`, `kpi-assurance`, `kpi-acres`, `kpi-agencies`, `gauge-share`, `dossier`, `ledger` — and two
via `fromWidget: "buffer"` (`kpi-radius`, `within`). Filter changes propagate to all of them through the
store, which is why the KPI strip has no wires of its own.

The spine:

```jsonc
{ "from": "cascade",  "trigger": "filterChange",   "to": "map",       "action": "filter", "options": { "layerId": "mines" } },
{ "from": "cascade",  "trigger": "filterChange",   "to": "ledger",    "action": "filter" },
{ "from": "cascade",  "trigger": "filterChange",   "to": "agencies",  "action": "filter" },
{ "from": "cascade",  "trigger": "filterChange",                      "action": "setUrlParam", "options": { "param": "where" } },
{ "from": "agencies", "trigger": "categorySelect", "to": "map",       "action": "filter", "options": { "layerId": "mines" } },
{ "from": "agencies", "trigger": "categorySelect", "to": "map",       "action": "flash",  "options": { "layerId": "mines" } },
{ "from": "flt",      "trigger": "filterChange",   "to": "map",       "action": "filter", "options": { "layerId": "mines" } },
{ "from": "ledger",   "trigger": "rowSelect",      "to": "map",       "action": "zoomTo", "options": { "layerId": "mines" } },
{ "from": "ledger",   "trigger": "rowSelect",      "to": "dossier",   "action": "viewInTable" },
{ "from": "map",      "trigger": "featureSelect",  "to": "dossier",   "action": "viewInTable" },
{ "from": "map",      "trigger": "extentChange",   "to": "kpi-barred","action": "showStatistics" },
{ "from": "buffer",   "trigger": "recordsChange",  "to": "kpi-radius","action": "showStatistics" }
```

### 2.8 As shipped — the surface the build added

§2.1–2.7 is what was specified. These are the capabilities the working app turned out to need. Each is
asserted in `app/test-cascade.mjs` (section in brackets).

| Shipped | Why it exists |
|---|---|
| **One map control cluster** — zoom · out · zoom-to-data · layers · basemap · legend *(§13)* | inline-SVG buttons replacing MapLibre's own zoom cluster, so there is exactly one set of controls. §8 had recorded `basemap`/`layer-panel` as covered by `map.controls.*`; the first build shipped neither, and a map you cannot re-basemap or re-layer is a screenshot |
| **Legend, and it is the layer switch** *(§13)* | barred · eligible · removed-by-the-cascade, each with a live count and each clickable to show/hide. The legend was the missing piece of readability — three colours on a map mean nothing unlabelled, and making the labels the controls costs no extra chrome |
| **Layers drawer** with two real context layers *(§13)* | **SMARA lead agencies** (140) answers "whose inspection is this?" and **aggregate Production-Consumption regions** (42) answers "is this even my market?" — aggregate is high-bulk and low-value, so supply is a regional question. Both lazy-load on first enable, and a failure turns the toggle back off with a plain-English message rather than a blank map |
| **Server-side generalization** *(§13)* | raw, those two layers are **21 MB and 33 MB** — unusable in a browser. `maxAllowableOffset=0.005` (~500 m) brings them to **268 KB and 76 KB**, which is the right trade for statewide context polygons. Asserted, so nobody removes it |
| **Basemap drawer** — 5 keyless styles *(§13)* | CARTO Positron/Voyager/Dark Matter · OpenStreetMap · OpenTopoMap. The suite asserts every tile URL is OSM-derived and carries no key, token or proprietary host — the house rule is enforced by the drawer offering nothing else. Switching the theme also switches the basemap (`basemapForTheme`) |
| **Popup on a mine** *(§13)* | the design had `feature-info` only on page 2, so page 1 had no way to read a single record. The popup is a genuine `popupInfo`-shaped field list, **never shows `OBJECTID`**, repeats the five-of-six caveat, and offers "Full record + the six conditions →" which crosses to page 2 |
| **Glossary — 15 terms, `ⓘ` in place** *(§14)* | ABLIST, SMARA, DMR, SMGB, FACE, RP_Number, lead agency, financial assurance, annual report, disturbed acres, aggregate, idle, newly permitted, P-C region, the List. One `GLOSS` table so a definition cannot drift between the rail, the ledger, the horizon and the popup. One tooltip element on `<body>`, because every panel is `overflow:auto` and would clip a CSS tooltip |
| **EPSG:4326 status readout, with a drawn count** *(§14, render §1)* | the design listed `status-bar`; the first build shipped a scale control instead and quietly dropped the CRS. It now reads `lon, lat · z · N drawn · EPSG:4326`, where `N` is what was **last handed to the map** — so `130 drawn`, `130 drawn of 874` and `layer not installed` distinguish *fine*, *stale* and *broken*. An empty map has to say it is empty rather than merely look it |
| **Keyboard: `L` layers · `B` basemap · `Esc` closes** *(§12)* | `Esc` closes the popup and both drawers as well as the splash |
| **Status filter in the header** *(§15)* | §2.1's `AppLayout` specifies the header filter over **three** fields; the build shipped two. `MineStatus` added, stacking with the rung like the other two |
| **Theme-aware map ink + a non-colour channel** *(§16)* | point colours were hardcoded light values, so dark mode drew an all-but-invisible green (2.81:1). Now one `POINT_COLORS` table per mode, read by both the map and the legend — and barred/eligible additionally differ by **fill density**, because red/green at 1.08:1 luminance is the colour-blindness failure case and this app's exhibit prints |
| **Everything is a toggle** *(render §2, §4)* | a rung, an agency bar, a legend chip and a table row all clear on a second click. The rail and the ledger were one-way selections with no route back but a reload |
| **Row click = pan + flash + popup** *(render §3)* | the full master–detail move; a map click stays popup-only. Motion honours `prefers-reduced-motion` — the pan becomes a jump, the flash a static ring |
| **Bounded self-heal + a visible failure** *(render §1)* | `ensureDrawn()` re-installs and repaints at **250 / 800 / 2000 / 4500 ms** then stops; `hydrate()` is wrapped so a throw renders a banner *in the map frame* naming the error; `window.__cascade.state()/.redraw()` expose per-map state. Added after three offline-green hypotheses failed to fix an empty map in a real browser — see §0.1 item 10 |
| **Style swaps rebuild the operation layers** *(render §5)* | `setStyle` fires `styledata` synchronously and *then* drops every custom source and layer, so hydration must stand down during the swap (`swapping[id]`) and rebuild from `once("idle")`. Without it the points vanish on every basemap and theme change — §0.1 item 13 |
| **`app/layers.mjs`** *(§13, §16)* | basemaps, context layers, point colours and the popup field model, kept out of `cascade.mjs` so the statutory logic carries no presentation |
| **`app/test-render.mjs`** *(40 assertions)* | boots the shipped inline script against a minimal DOM + MapLibre stub and drives the loop. `test-cascade.mjs` greps the markup and never runs it, so "the map is empty", "a rung toggles" and "a row flashes" were structurally invisible to it |

**Deliberate non-additions.** No `carto` widget — one brushable surface (the agency bars) keeps the
ledger the protagonist. No labels layer — text would need a glyph endpoint, and the honest keyless
options are all third-party; hover and click carry the names instead. The ledger **caps at 250 rows**
and says so rather than virtualizing, which is a real divergence from the `table` widget the design
specifies.

---

## 3. Data sources

**Full inventory, field lists, counts, distributions and ten traps: `../../data_sources/data_sources_ca.md` § *MOL — the statewide mine register*.** Everything below was
fetched live **2026-08-05**. All EPSG:4326 on ingest.

### 3.1 The headline — the catalogue had no mine data at all

`data_sources_ca.md` enumerates **2,569 endpoints**. A sweep for
`min|mineral|mining|quarr|aggregat|reclam|smara|abandon|geolog` returns **five genuine services, none of
them a mine record** — Caltrans `DEA_Mineral_Hazard` (asbestos screening), two DWR
`i08_Geologic*` basin layers, CAL FIRE `Geology_Points`, Cal OES `Geology1`. Everything else is noise on
the substring `min` (`NDFD_Minimum_Relative_Humidity`, `FS_MINDY_2025`, `CalOES_Admin_…`,
`OilRefineriesandTerminals`, `…_15min_…`).

The one server the catalogue **named but never enumerated** — `gis.conservation.ca.gov`, described only
as CGS seismic hazard — is the primary source for this entire vertical and carries the statewide mine
register **with the AB 3098 flag on it**.

| Catalogue said | Verified to exist |
|---|---|
| no mine register | **`MOL/MOLMines/FeatureServer/0`** — 1,975 points, 34 fields, `ABLIST` flag |
| no compliance flag | `ABLIST` `'ON    '` 716 / `'OFF   '` 1,259 (space-padded — **M1**) |
| no SMARA jurisdictions | **`MOL/MOLLeadAgency/MapServer/0`** — 140 polygons |
| (not known) | **`DMR/` is token-gated** — `499 Token Required` — the evidence horizon |

`data_sources_ca.md` has since been amended (§ *Mining, Extraction & Land Resources — [MIN]* and § *CA
Dept. of Conservation — server #7*). Two further corrections proposed in §7.7.

### 3.2 Role × region

*(Maryland was not swept — the demo region and the binding authority are both Californian. The AB 3098
purchasing bar is distinctively Californian; a second region would need its own statute, not just its
own data.)*

| Role | California (verified 2026-08-05) | National | Client-supplied |
|---|---|---|---|
| **Mine register + compliance flag** ★★ | `gis.conservation.ca.gov/…/MOL/MOLMines/FeatureServer/0` (**1,975**) | USGS MRDS (historic, no compliance) | — |
| Register **without** the flag | `…/MOL/MOLMinesNoAB/FeatureServer/0` (1,975) — **do not bind** (M3) | — | — |
| SMARA lead agencies | `…/MOL/MOLLeadAgency/MapServer/0` (**140**, MapServer only, no `objectIdField`) | — | — |
| Aggregate market geography | `…/CGS/IW_MineralResourcesProgram/FeatureServer/`**`4`** Production-Consumption Regions · **3** Production Areas · **7** MLC Reports | — | — |
| **Inspections, fees, orders to comply** | ⛔ `…/DMR/` — **499 Token Required** | ⛔ | — |
| The published List (legally operative) | ⛔ California Regulatory Notice Register + DMR PDFs — **not reachable server-side** (M7) | — | — |
| Approved-supplier list to diff | — | — | optional `FileDataSource` CSV |
| Farmland / Williamson Act (out of scope) | `…/DLRP/…` | — | — |
| Basemap | keyless OSM · CARTO Positron/Voyager/Dark · OpenTopoMap | | |

### 3.3 The demo region

**California, statewide.** No single county carries the story: the 130 barred operations span **43 lead
agencies**, and the largest single concentration — County of San Bernardino — holds only 16. The map
opens on the barred extent `-124.16195784, 32.77639383 → -114.63140052, 41.88333836`.

⚠ **The authoritative list is a document, not this feed.** `www.conservation.ca.gov` times out and
`maps.conservation.ca.gov` returns 403 to non-browser clients; **only `gis.conservation.ca.gov`
answers** (§3.3). So the app renders the register's flag as of its extract date, stamps that
date on every rung and every export, and states that DMR's quarterly publication governs. It must never
present itself as the List.

---

## 4. Verify each URL first (terminal)

```bash
# ── 1. THE SERVER. CA Dept. of Conservation, ArcGIS Server 11.5. CORS reflects Origin — no proxy.
B=https://gis.conservation.ca.gov/server/rest/services
curl -s "$B?f=json"
#  -> folders: Base CalGEM CGS CGS_Earthquake_Hazard_Zones DLRP DMR DO Hosted MOL Test Utilities WellSTAR
curl -sD- -o /dev/null -H "Origin: https://example.com" "$B/MOL/MOLMines/FeatureServer/0?f=json" | grep -i access-control
#  -> Access-Control-Allow-Origin: https://example.com     <-- browser-direct
curl -s "$B/MOL?f=json"
#  -> MOLLeadAgency(MapServer) · MOLMines(FS+MS) · MOLMinesNoAB(FS+MS)

# ── 2. THE EVIDENCE HORIZON. Confirm the gap BEFORE designing around it.
curl -s "$B/DMR?f=json"
#  -> {"error":{"code":499,"message":"Token Required","details":[]}}
#     Inspections, fee ledgers and orders to comply — conditions (3)(5)(6) — are NOT public.

# ── 3. THE REGISTER. 1,975 points, 34 fields, oid OBJECTID, SR 3857 -> always outSR=4326.
M=$B/MOL/MOLMines/FeatureServer/0
curl -s "$M?f=json" | head -c 400          # name MOL.DOC.allminesWGS84 · maxRecordCount 2500
curl -s "$M/query?where=1=1&returnCountOnly=true&f=json"                    #  -> {"count":1975}

# ── 4. THE SPACE-PADDING TRAP (M1). The single most likely way to ship a broken app.
curl -s -G "$M/query" --data-urlencode "where=1=1" --data-urlencode "f=json" \
  --data-urlencode 'outStatistics=[{"statisticType":"count","onStatisticField":"OBJECTID","outStatisticFieldName":"n"}]' \
  --data-urlencode "groupByFieldsForStatistics=ABLIST"
#  -> {"ABLIST":"OFF   ","n":1259}  {"ABLIST":"ON    ","n":716}
#     ArcGIS SQL ignores the padding: ABLIST='OFF' and ABLIST='OFF   ' both return 1259.
#     JAVASCRIPT DOES NOT: attributes.ABLIST === 'OFF' is false on EVERY row. Always trim() client-side.
#     Same on ReportType: "2 " -> 1962, "1 " -> 10.

# ── 5. THE CASCADE. Four steps; each is the definitionExpression of one rung.
curl -s -G "$M/query" --data-urlencode "where=1=1" --data-urlencode "returnCountOnly=true" --data-urlencode "f=json"
#  -> 1975                                                                       ① the register
E="MineStatus IN ('ACTIVE','IDLE','NEWLY PERMITTED')"
curl -s -G "$M/query" --data-urlencode "where=$E" --data-urlencode "returnCountOnly=true" --data-urlencode "f=json"
#  -> 874        <-- PRC §2717(b): "newly permitted, ACTIVE, or IDLE". NOT active-only (=748).
curl -s -G "$M/query" --data-urlencode "where=$E AND ABLIST LIKE 'OFF%'" --data-urlencode "returnCountOnly=true" --data-urlencode "f=json"
#  -> 160                                                                        ③ PCC §10295.5/§20676
curl -s -G "$M/query" --data-urlencode "where=$E AND ABLIST LIKE 'OFF%' AND ReportYear>=2025" \
     --data-urlencode "returnCountOnly=true" --data-urlencode "f=json"
#  -> 130        <-- filed a current annual report (condition 4) and barred anyway

# ── 6. THE TERMINAL POPULATION. These are the four KPI values.
curl -s -G "$M/query" --data-urlencode "where=$E AND ABLIST LIKE 'OFF%' AND ReportYear>=2025" --data-urlencode "f=json" \
  --data-urlencode 'outStatistics=[{"statisticType":"sum","onStatisticField":"FaceAmount","outStatisticFieldName":"assurance"},{"statisticType":"sum","onStatisticField":"Acres_Dist","outStatisticFieldName":"acres"}]'
#  -> {"assurance":12950937.46,"acres":2498.17}      43 distinct LAS_Name · 71 aggregate of 130
#     ⚠ PriProduct is NULL on 29 of the 130 — "71 aggregate" is a FLOOR, never a two-way split (M6).

# ── 7. THE CONTROL TESTS. Prove the cascade is not a restatement of something already in the extract.
curl -s -G "$M/query" --data-urlencode "where=ABLIST LIKE 'ON%' AND ReportYear<2025" --data-urlencode "returnCountOnly=true" --data-urlencode "f=json"
#  -> 44      44 mines sit ON the list with a STALE report => the list is NOT a report-currency check.
#             (restricted to MineStatus='ACTIVE' the same test gives 33.)
curl -s -G "$M/query" --data-urlencode "where=MineStatus='ACTIVE' AND ABLIST LIKE 'ON%' AND RP_Number IS NOT NULL AND RP_Number<>''" --data-urlencode "returnCountOnly=true" --data-urlencode "f=json"
#  -> 138 of 632 = 21.8 %      and OFF-list: 20 of 116 = 17.2 %
#     RP_Number measures EXTRACT COMPLETENESS, not plan approval. Rejected as a rung (the shared catalog, § *MOL* control tests).

# ── 8. THE DECOY LAYER (M3). Same 1,975 rows with the one column this app is about deleted.
curl -s "$B/MOL/MOLMinesNoAB/FeatureServer/0?f=json" | grep -c ABLIST      #  -> 0
curl -s "$B/MOL/MOLMinesNoAB/FeatureServer/0/query?where=1=1&returnCountOnly=true&f=json"  # -> 1975

# ── 9. THE LEAD-AGENCY LAYER (M4). MapServer only; objectIdField is undefined.
curl -s "$B/MOL/MOLLeadAgency/MapServer/0?f=json" | head -c 300
#  -> MOL.DOC.Lead_Agencies · esriGeometryPolygon · NO objectIdField · fields NAME_PCASE NUM PLACENAME LEADAGENCY
curl -s "$B/MOL/MOLLeadAgency/MapServer/0/query?where=1=1&returnCountOnly=true&f=json"     # -> 140

# ── 10. THE GEOMETRY YOU MUST NOT INVENT (M2).
curl -s "$M/query?where=1=1&outFields=PermitAcre,Permit_Num&resultRecordCount=3&returnGeometry=false&f=json"
#     PermitAcre is a String(200). The register publishes POINTS; there is no permit boundary anywhere
#     in MOL. Buffering a self-reported point by a parsed acreage fabricates the geometry the whole app
#     would rest on — on a layer whose publisher disclaims coordinate accuracy. Do not draw it.

# ── 11. THE EXTENT for initialState.viewpoint.
curl -s -G "$M/query" --data-urlencode "where=$E AND ABLIST LIKE 'OFF%' AND ReportYear>=2025" \
  --data-urlencode "returnExtentOnly=true" --data-urlencode "outSR=4326" --data-urlencode "f=json"
#  -> -124.16195784, 32.77639383 -> -114.63140052, 41.88333836

# ── 12. THE PUBLISHED LIST IS NOT FETCHABLE (M7). Confirm before designing a feed that cannot exist.
curl -s -o /dev/null -w "%{http_code}\n" "https://maps.conservation.ca.gov/mol/index.html"   # -> 403
curl -s -o /dev/null -w "%{http_code}\n" --max-time 45 "https://www.conservation.ca.gov/smgb/Pages/AB-3098-List.aspx"
#  -> 000 (timeout).  Only gis.conservation.ca.gov answers. The app renders the REGISTER'S FLAG and
#     says the quarterly publication governs. It never claims to be the List.
```

---

## Guided wizard — **the prompts that assign the app's defaults**

Launch with `/recipe industry_mining-and-concession-compliance`. Ask each group, **apply the default so
"accept all" builds a complete app**, confirm a one-line summary, then run §5.

| # | Wizard question | Options → **default** | The default it assigns |
|---|---|---|---|
| 1 · App | Title & extract date? | free text → **"Mining & Concession Compliance — the rule cascade"**, today | header title + the vintage line stamped on every rung and export |
| 2 · Universe | Which operations does the List cover? | **PRC §2717(b): newly permitted, active or idle (874)** · active only (748, the narrower reading) | rung ②'s `definitionExpression` |
| 3 · Report cycle | Which filing year counts as current? | **2025** (942 of 1,975 rows) · 2024 · custom | rung ④'s threshold — a **config constant**, never a literal (M5) |
| 4 · Commodity focus | Restrict to the commodities PCC §20676 names? | **no — show all, flag aggregate** · yes, aggregate only | header `filter` default; the "71 of 130" framing |
| 5 · Agency lens | Filter to one SMARA lead agency? | **no — statewide (43 in the barred set)** · pick one of 140 | `definitionExpression` on `LAS_Name`; map extent |
| 6 · Horizon | Show the evidence-horizon panel? | **yes** · no | page-1 right panel + the `DMR` 499 probe result |
| 7 · Controls | Show the control tests (44 / 21.8 % vs 17.2 %)? | **yes** | the accordion under the horizon |
| 8 · Page 2 | Include the haul-radius dossier? | **yes, 40 km straight-line screen** · no · custom radius | page `supply`; `near-me` default + the `analysis` buffer |
| 9 · Context | Overlay aggregate market geography? | **Production-Consumption Regions (layer 4)** · none · + Production Areas (3) | context layers at 40/255 alpha |
| 10 · Supplier list | Diff against your own approved-supplier CSV? | **skip** · CSV URL | optional `FileDataSource`; joined on `Mine_ID` |
| 11 · Look | Light or dark? | **light** (a counter document; prints anyway) · dark | `ThemeSpec.mode` + `basemapForTheme` |
| 12 · Export | What does the print button produce? | **the procurement record** (mine, flag, extract date, the six conditions) · summary only | `print` exhibit + `table` CSV |

---

## 5. Prompt-script (run in order)

```
A. /new-app — "Mining & Concession Compliance — the rule cascade", Template: open-design.
   Structured ThemeSpec (§2.2), light, primary #3d5a6c. App-shell: header (title · vintage line ·
   commodity/agency filter · page-nav · theme-switch), footer (attribution · share · print),
   splash titled "This is not the AB 3098 List". Two pages: `cascade`, `supply`.
   Register the one app-local widget (§2.6) with its fallback behind a flag.

B. /add-data — the verified spine, all outSR=4326:
     mines     MOL/MOLMines/FeatureServer/0            (1,975)  <-- NOT MOLMinesNoAB (M3)
     agencies  MOL/MOLLeadAgency/MapServer/0           (140, MapServer, no objectIdField)
     pcregion  CGS/IW_MineralResourcesProgram/FeatureServer/4   (Production-Consumption Regions)
   Set initialState.viewpoint to the barred extent (§4 step 11).

C. /symbology + /popup — genuine ESRI drawingInfo/popupInfo on verified fields only.
     mines     uniqueValue on an Arcade valueExpression that TRIMS ABLIST first (M1) and buckets
               list position x aggregate commodity. esriSMSCircle / esriSMSSquare — never esriSMSPath.
               Eliminated rows stay drawn at ~12% opacity: subtraction must be visible.
     agencies  simple fill, alpha 40/255, outline two tones darker.
     popupInfo mines: MineName, Mine_ID, Operator, MineStatus, ABLIST (trimmed), ReportYear,
               FaceAmount, Acres_Dist, LAS_Name, PriProduct. Never show OBJECTID.

D. /panel — the cascade rail (§2.6) with the four steps of §1.3 as its `steps` prop, each carrying its
   citation and `where`. Bind dataSource.sourceId = "mines". Stamp the extract date on every rung.

E. /panel table — the evidence horizon: 7 static rows, columns cond · requirement · public evidence ·
   readable. Paste the DMR 499 response verbatim into row (3)(5)(6). Then the control-test accordion.

F. /panel statistics + /panel chart + /panel table —
   KPIs: Barred, Financial assurance, Disturbed acres, Lead agencies, %-of-universe gauge (10/25).
   Chart: "Barred by lead agency" — bar, LAS_Name x count, limit 12.
   Table: the ledger (10 columns, virtualized, CSV = the procurement record).

G. /analyze — page 2: buffer the job centre by the wizard's haul radius, publish as an output; bind
   kpi-radius and the within-radius table with dataSource.fromWidget. Label it a straight-line screen,
   NOT a drive time.

H. WIF: author AppLayout.connections — the 21 rows in §2.7. Shipped emitters ONLY;
   no window, no buttonClick, no navigate. Verify the signature loop end to end.

I. Controls + composed export: scale, legend, basemapSwitcher, layerList; status-bar in EPSG:4326;
   print (the procurement record, with the extract date and the six conditions on it); share deep-link.
```

---

## 6. Verify

| Check | Pass |
|---|---|
| Silhouette is `rule-cascade`; distinct from every sibling and from every researched open-design | ✅ §2.1 anti-collision — no sibling navigates by elimination |
| ≥3 `connections` fire; the signature loop works end to end | ✅ 21 authored, all on shipped emitters; +7 widgets link via `sourceId`, +2 via `fromWidget` |
| Every `layerId` + field verified against the live service (§4) | ✅ 1,975 · 140 · 1,975(NoAB) · 34 fields listed in the shared catalog § *MOL* |
| No wire uses a pending emitter | ✅ `buttonClick`/`mapClick`/`sketchComplete`/`viewChange`/`pageChange`/`timer` unwired; **no `window`** as a result |
| The cascade reproduces from a cold terminal | ✅ 1,975 → 874 → 160 → 130 (§4 step 5) |
| The cascade is not tautological | ✅ 44 on-list-with-stale-report; `RP_Number` 21.8 % vs 17.2 % (§4 step 7) |
| `ABLIST` is trimmed everywhere it is compared client-side | ✅ M1 — renderer, popup, ledger, exports |
| The statutory universe is used, not active-only | ✅ 874 headline; 748/116/95 shown once as a labelled comparison |
| No number is derived from geometry the register does not publish | ✅ M2 — no permit boundary anywhere; `PermitAcre` never parsed |
| "Aggregate producers" is rendered as a floor | ✅ *"71 of 130 declare an aggregate commodity; 29 declare none"* (M6) |
| The app never claims to be the List | ✅ splash + header vintage + footer + every export: *"the published quarterly List governs"* |
| The evidence horizon names all five unreadable conditions | ✅ 7-row table incl. the §2717(c) appeal override |
| `responsive.small` collapses every side-by-side row | ✅ both splitters → column; rail → `dock:"top"`; KPIs → `flow-row`; 240 px map floor |
| Basemap keyless; everything EPSG:4326; `OBJECTID` is the OID | ✅ CARTO/OSM only; `outSR=4326` on every request |
| Runs on Strata **and** ArcGIS | ✅ read-only throughout — there is no write path to degrade |
| The report-cycle threshold is config, not a literal | ✅ wizard Q3 (M5); asserted in `test-cascade.mjs` §11 |
| **Each rung's client predicate reproduces its own SQL against the live service** | ✅ `test-cascade.mjs` §2 — 4 rungs × (server count, client count), all agree |
| The app fetches the register in one request, no paging, no proxy | ✅ 1,975 rows, ~1.2 MB, CORS-open (§1) |
| Every number on screen is asserted | ✅ `cascade.mjs` is imported unchanged by the browser and the suite |
| The suite fails if the register changes | ✅ deliberate — live-data assertions, no fixtures |
| The header matches §2.1's `AppLayout` field-by-field | ✅ `test-cascade.mjs` §15 — three filters, title, vintage, nav, theme switch |
| Point ink carries ≥3:1 against **its own** basemap in both modes | ✅ §16 — light 5.74/5.32:1, dark 5.92/5.96:1; the light-on-dark reuse (2.81:1) is asserted as the rejected case |
| Barred vs eligible is not encoded by hue alone | ✅ §16 — solid fill vs hollow ring, in the map paint and the legend |
| Anything toggled by `[hidden]` can actually be hidden | ✅ §12 — general guard over markup **and** JS-built elements; verified to fail without its CSS rule |
| The map is not empty on first open | ✅ `test-render.mjs` §1 — 130 features reach the source, fit-to-data runs, no error banner |
| Hydration cannot re-enter on `idle` | ✅ render §1 — 1 `setData` on boot; 43 without the guard |
| Rung, row, agency bar and legend chip all toggle off | ✅ render §2/§4 |
| A table row pans, flashes and opens the popup | ✅ render §3 |
| Only keyless, OSM-derived basemaps are offered | ✅ §13 — every tile URL checked for keys, tokens and proprietary hosts |
| Context layers are generalized before they reach the browser | ✅ §13 — `maxAllowableOffset=0.005` asserted (21 MB → 268 KB, 33 MB → 76 KB) |
| The popup never exposes `OBJECTID` and escapes its content | ✅ §13 |
| Every abbreviation the UI marks has a definition, and none is dead | ✅ §14 — 15 terms, all referenced |

---

## 7. Harvest (gaps → strata-core)

1. **`chart` has no `referenceLine` / threshold prop.** The agency bar chart wants the same rule marker
   the rail draws. Would also make `rule-cascade`'s fallback much closer to the real thing. *(Also
   requested by `education_campus-operations` §7.4 — second request, worth promoting.)*
2. **`table` has no `orderBy` / `sort` prop** in the manifest, so "largest disturbed acreage first" has
   to come from a filter or the chart. A declarative default sort would remove a widget from this design.
   *(Second request — also `education_campus-operations` §7.3.)*
3. **`button` does not emit `buttonClick`**, so a `window` cannot be opened declaratively. This design
   dropped its statute modal entirely and used an `accordion`. Manifest §4.1 lists the emitter as
   pending. *(Third request across researched recipes.)*
4. **No shipped widget renders a static "what we cannot show" panel.** Built here from `table` +
   static `rows` + `text`, which works, but the motif recurs — see the harvest note below.
5. **MapServer sources with no `objectIdField`** (`MOLLeadAgency`) have no documented handling in the
   manifest. Worth a line in §5 *Rules that bite*: read the layer JSON, and if `objectIdField` is absent,
   bind read-only and do not attempt OID-keyed selection.
6. **Harvest candidates:** **`rule-cascade`** (navigation by elimination — reusable by any recipe whose
   product is *who is excluded, and by which rule*: permit eligibility, benefit qualification, debarment)
   and the **evidence-horizon motif** (a terminal panel naming what the data cannot show, built from
   shipped widgets). Promote per the `APP-TEMPLATE-LIBRARY.md` harvest rule if reused twice.
7. **Catalogue corrections** (proposed text in §7.7): note that `ABLIST` is **space-padded** in
   the `[MIN]` row, and amend the cascade framing there from *active* to the statutory *newly permitted,
   active, or idle*.

---

## 7a. Presentation & article

| File | |
|---|---|
| `presentation/index.html` | 10-slide tabaqat deck, 16:9, keyboard + click nav, prints one slide per page. Opens on *"They filed the paperwork. They still can't sell you gravel."* The arc: the question → the six conditions → the cascade → the three-status correction → the evidence horizon + both control tests → honest scope |
| `presentation/linkedin-article.md` | ~1,300-word article, a ~180-word teaser, and a **27-row claims note** tracing every figure to the query or document it came from — plus an explicit **"Do not claim"** list and five gaps to volunteer if asked |

Both were verified against the **live register**, not against this file: all 19 figures re-queried and
matched, the deck checked for structure and for any named operator, and the article checked for the
framing that must survive editing.

**The framing is load-bearing.** The single biggest risk in publishing this is implying that 130 named
businesses are breaking the law. They are not: they are **absent from a published list**, and because
five of the six §2717(b) conditions are not public, the reason is not knowable from this data. Every
phrasing is *"off the list" / "barred from selling to a public agency"*, never *"in violation"*, and no
individual mine, operator or owner is named anywhere in the deck or the article. The lead is the
population and the statute, never a company.

---

## 7a-i. Open questions & risks

| # | Risk | Mitigation shipped |
|---|---|---|
| **R1** | **`ABLIST` is space-padded** — a JS `=== 'OFF'` silently classifies every mine as neither on nor off | every client-side read goes through `trim()`; the Arcade renderer expression trims first; the suite asserts a non-empty barred set (M1) |
| **R2** | **The app is not the List.** A user could treat a screen as the legal artifact | splash, header vintage line, footer attribution and every export carry *"the published quarterly List governs"*; the extract date is on every rung |
| **R3** | **"Current report" is a moving constant.** `ReportYear >= 2025` is right today and wrong next spring | a wizard answer and a `CONFIG` constant, never a literal; the header shows which cycle is in force (M5) |
| **R4** | **The active-only reading is seductive** and gives a smaller, tidier number | the statutory universe (874) ships as the headline; the active-only chain (748→116→95) appears once, labelled, as a comparison |
| **R5** | **`PriProduct` is null on 29 of 130** — an "aggregate producers" count could read as complete | rendered as *"71 of 130 declare an aggregate commodity; 29 declare none"*, never as a two-way split (M6) |
| **R6** | **Coordinates are operator-reported** and DMR disclaims them | no distance claim tighter than a haul-radius screen; the radius is labelled a straight-line screen, not a drive time; no boundary is ever drawn (M2, M8) |
| **R7** | **The register could add or rename a column**, or the DMR folder could open | the six-condition horizon is config; if `DMR` ever answers, conditions (3)(5)(6) become bindable and the horizon shrinks — the intended growth path, not a rewrite |
| **R8** | **`MOLLeadAgency` is MapServer-only with no `objectIdField`** | bound as read-only context; no OID-keyed selection attempted (M4) |
| **Q1** | Should the app ingest the client's **own approved-supplier list** and diff it against the register? A `FileDataSource` CSV would make the procurement officer's job one click. Offered as optional; needs a client to define the file. |
| **Q2** | Is a **haul-radius screen** the right page-2 frame, or a plain mine lookup? The radius is more useful and uses only shipped capability, but aggregate haul economics really want a road-network drive time we cannot honestly compute. Shipped as a straight-line screen, labelled. |
| **Q3** | **Sibling states?** Every state runs its own reclamation regime; the AB 3098 purchasing bar is distinctively Californian. A second region needs its own statute, not just its own data. |

---

## 8. Sources

**Internal** — `../APP-TEMPLATE-LIBRARY.md` · `../DESIGN-CONTEXT.md` · `../DESIGN-REQUEST-PROMPT.md` ·
`strata/recipes/COMPONENT-MANIFEST.md` (§3 registry, §4 triggers/actions, §10 freestyle charter) ·
`strata/docs/guide/app-design.md` · **`../../data_sources/data_sources_ca.md`** (the data evidence) ·
**`../APP-TEMPLATE-LIBRARY.md`** (the design record + harvest notes).

**Statute & regulation** (retrieved 2026-08-05)
- **PRC §2717** — full text incl. subdivisions (b)(1)–(6) and (c), via
  `california.public.law/codes/ca_pub_res_code_section_2717` (mirror of leginfo PRC §2717, updated
  2020-01-01); cross-checked against the SMARA amendments in the **AB 1142 (2016)** enrolled text at
  `leginfo.ca.gov/pub/15-16/…/ab_1142_bill_20160414_enrolled.htm`.
- **PCC §10295.5** (state purchases) and **§20676** (local purchases) — the AB 3098 bar.
- **SMARA** PRC §§2710–2796, esp. §2207 (annual report + fees), §2736, §2773.4, §2774(b), §2770(e),
  §2774.4. **AB 3098 (1992)**.
- **SMGB** *AB 3098 List* page and DMR's *AB 3098 Purchase Preference List Update for Lead Agencies*
  (both located; **neither reachable server-side** — see M7).
- **Caltrans** Construction Manual **§3-6** *Control of Materials* (points staff to the "AB 3098 SMARA
  Eligible List"); Aggregate Prequalification Program (a **different** test — materials quality).

**Competitive** (page-verified 2026-08-05)
- **DOC/DMR Mines Online** — `maps.conservation.ca.gov/mol/`, the incumbent public register viewer.
- **Esri** — `esri.com/en-us/industries/mining` (lifecycle positioning; exploration/permitting/EHS
  segments); New Mexico EMNRD Mining and Minerals Division as the cited regulator reference.
- **Trimble Landfolio** / Spatial Dimension — `landadmin.trimble.com`, *e-Gov for Mining*; corporate
  users incl. Rio Tinto, Newmont, OZ Minerals, Samancor.
- **K2fly / Decipher** — `k2fly.com`, Natural Resource Governance suite; Esri and SAP partnerships.
- **Datamine · Maptek · Seequent · RPMGlobal · MICROMINE**; **Benchmark Gensuite**, **iPoint**,
  **Fair Supply**, **BanQu**.
- **Public cadastre portals** — Queensland **GeoResGlobe**; Western Australia **TENGRAPH** / MINEDEX /
  GeoVIEW / Mineral Titles Online; Peru **GEOCATMIN** (INGEMMET).

**Data** — CA Dept. of Conservation, Division of Mine Reclamation, *Mines Online* register
(`MOLMines`, `MOLMinesNoAB`, `MOLLeadAgency`); California Geological Survey
*IW_MineralResourcesProgram*. All endpoints, counts, field lists and traps: **`../../data_sources/data_sources_ca.md` § *MOL***.

---

## Modernization (parity release)

> Applies the Experience-Builder-parity features (Phases 1–7). See `strata/recipes/MODERNIZATION.md` +
> `COMPONENT-MANIFEST.md` §8.

- **Structured theme** — the whole look from one `primary` hex, with `success`/`danger` carrying real
  semantics (on the List / barred) and a `kpi` override for tabular monospace.
- **App-shell** — `header` (title · extract vintage · filters · nav · theme), `footer` (attribution ·
  share · print), and a `splash` whose title is *"This is not the AB 3098 List"*.
- **DataSource linking** — `sourceId: "mines"` links the four KPIs, the gauge, the dossier and the
  ledger with **zero** `connections`; `fromWidget: "buffer"` feeds the haul-radius KPI and table. Live
  `kpi.stat` / `gauge.stat` throughout.
- **Layout nodes** — `splitter` ×3 per page, `panel` (rail and horizon, `dock:"top"` on phones),
  `accordion` (control tests + the statute), `flow-row` (the KPI strip). **No `window`** — deliberately,
  because its opener is a pending emitter.
- **`analysis` widget** — the `buffer` tool publishing its result as an output two widgets consume.
- **`query` widget** — page-2 name/operator/ID search publishing an output rather than a bare filter.
- **Deliberate abstentions** — no `animate`, no `autoPlay`, no time slider, no `date-filter`, no
  `RestDataSource`/`StreamDataSource`, no write path. An instrument that decides whether a purchase is
  lawful earns trust by holding still.
