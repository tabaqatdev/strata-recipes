# Recipe — Pipeline Integrity Evidence Pack: **"The Alignment Strip"** (Energy & Utilities)

> **RESEARCHED 2026-08-06.** First write-up of this solution folder, and the first researched recipe in the
> **Energy** sector. §1 is page-verified; §3/§4 are curl-verified live against
> `services.arcgis.com/BLN4oKB0N1YSgvY8` (Cal OES hosted), `services.gis.ca.gov` (State Geoportal) and
> `gis.conservation.ca.gov` (Dept. of Conservation). Every count, field list, distribution, measured
> mileage and trap lives in **`pipeline-catalog-ca.json`** beside this file and in
> `../../data_sources/data_sources_ca.md` § *Energy — CA hazardous liquid pipelines and the seismic
> evidence gap*. The design argument — three candidate silhouettes, the rejected rivals, the capability
> sweep, the §10.3 widget block — is **`DESIGN-PROPOSAL.md`** beside this file. This file is
> self-sufficient for building.

Milepost by milepost: what can be **proved** about this pipeline today, what no authority has ever
looked at, and which record only the operator holds.

---

## 0.1 The working app in this folder — **built and driven**

```
cd app && node server.mjs      # → http://localhost:8032/  (steps to the next free port if taken)
node test-strip.mjs            # 357 assertions — arithmetic, live data, traps, chrome, theme
node test-render.mjs           # 151 assertions — boots the shipped script and drives the loop
```

**508 assertions, all green 2026-08-06**, and **driven in a real browser** (Chrome via CDP) — which
found four defects that 297 passing assertions had not (§0.1 items 8–11), and a fifth on the operator-file
path (item 12). Self-contained, MapLibre from a CDN, no build step. See
`app/README.md` for running, method and debugging. `strip.mjs` and `layers.mjs` are imported
**unchanged** by both the browser and the suites, so no number can reach the screen that the suite has
not asserted. Assertions run against **live services** by design: if the register or the hazard layers
change, the suite goes red and the constants are known stale rather than quietly wrong.

**Nothing is synthesized** — no sample ILI, no mock CP survey, no fabricated HCA table. Nine of the
fourteen lanes render empty with their citation, because fabricating them in an app whose whole argument
is provenance would be self-defeating.

**Driving it produced eleven corrections the written design had wrong** — seven from building and
asserting it, and four more that appeared only once it was rendered in a browser (8–11 below).

1. **⚠ The headline figure for the demo line was wrong, and the research script hid it.** §3.3 said line
   0274 had *"eight unevaluated bands covering roughly 53 of its 91 miles"*. The truth is **91.05 of
   91.05 — 100 %**. The original figure came from a quick probe whose `try/except: continue` silently
   swallowed **5 of 13** polygons that failed `shapely` validity. The app, implemented independently,
   disagreed; an authoritative re-clip with `make_valid` confirmed the app. Corrected everywhere.
   *The lesson is not "use make_valid" — it is that a bare except in a measurement script is a data
   corruption that reports success.*
2. **⚠ `MAPNAME_1` does not name the quadrangle.** The design (and the catalogue amendment, and the
   design proposal) claimed the unevaluated band names the USGS 7.5′ quad a study must cover. The field
   is **whitespace-only on all 2,806 rows** — as are `LONGITUDE1` and `LATITUDE_S`. Verified:
   `where=MAPNAME_1 IS NOT NULL AND MAPNAME_1 <> ''` returns **0**. An unevaluated reach is *locatable*
   but not *nameable*; the dossier now exports its station range and extent instead, and says why. The
   evidence gap is one layer deeper than the research claimed (trap **P10**).
3. **⚠ Vertex-snapped band edges manufactured the one state that must never be manufactured.** The first
   implementation snapped every band edge to the nearest centerline vertex. Two abutting unevaluated
   quadrangles therefore each lost their shared edge, and the strip drew a hairline
   **"evaluated, and not zoned"** sliver between them — painting *an authority looked and found nothing*
   onto ground nobody has looked at. Measured: 88.98 mi unevaluated instead of 91.05. Band edges are now
   refined by **bisection** to about a metre, and the app reproduces an independent geodesic clip to
   0.01 mi on all three polygon lanes.
4. **"Resolved" and "evidenced" were being conflated.** Five lanes resolve — the register carries a
   commodity, a diameter and a length — but diameter and stationing are `unverifiable`, so only **three**
   are evidenced. The KPI was right at 3 of 14 *at that point*; a test asserting "3 resolved" was wrong.
   (Correction 9 later made this count **per line** — 5 on 0274, 4 on 1341 — with 3 as the structural
   ceiling.) `EVIDENCED_COUNT`
   is now **derived** from the lane states rather than hardcoded, so adding a lane cannot silently move
   the number on the splash screen.
5. **The keyless-basemap guard repeated the sibling recipe's exact mistake.** A grep for `proxy` over
   `server.mjs` matched the file's own explanatory prose. Narrowing the regex would have weakened the
   guard; instead it now asserts the **behaviour** — the server calls no `fetch(`, no `http.request`, no
   `createConnection`, and imports only `fs`/`http`/`path`/`url`.
6. **The render harness's own stub was wrong twice, and each time an assertion caught it rather than
   inspection.** An id-only scan of the markup broke on `$("#packtable tbody")` — the app uses ordinary
   descendant selectors, and the harness must support them rather than the app be rewritten to suit its
   test. Then a flat tag-only parse returned `""` from `.textContent`, so a passing app read as a
   failure. Both replaced with one stack-based parser used for the body *and* every dynamic `innerHTML`.
7. **`node server.mjs` died in a stack trace when the port was busy** — which it was, on the first run.
   A crash dump on the first command of a demo is a bad first experience; it now steps to the next free
   port, bounded, and says so.

8. **⚠ `evidenced` was painted in the danger role, so the app's best case looked like its worst.**
   §2.2 assigned `danger` to *evidenced*. On screen that meant lane 1 — **commodity**, a plain factual
   value — drew a full-width blood-red bar, and a line with a *complete* record would have rendered as
   the most alarming thing in the app. "An authority answered" and "an authority answered, and the
   answer is a hazard" are different facts and must not share ink. `evidenced` is now the **primary**
   role; mapped fault and liquefaction zones take a new **`evidenced-hazard`** state (danger, with a
   *dense* fill so the split survives greyscale print and colour-vision deficiency). Red is now
   rationed to the two findings that genuinely are hazards. *No assertion could have caught this — the
   ink was internally consistent and semantically inverted.*
9. **⚠ The lane state was the network's verdict, stamped onto every individual line.** Diameter and
   stationing carried a permanent `unverifiable` state taken from the network findings (28.3 % of
   mileage unparseable, 33.1 % discontinuous). But line **1341 files a plain 30**, and line **0274
   measures to within 0.18 %** of its reported length — flagging either as unverifiable is a false
   statement about a specific asset, which is precisely the failure this recipe exists to prevent.
   `perLineStates()` now decides per line and prints the reason on the row
   (*"5 parts — no single milepost axis exists"*), and the KPI is a **live per-line count** — 5 of 14 on
   0274, 4 on 1341 — shown beside the structural ceiling of 3.
10. **Ruler labels collided at every part boundary.** On the 5-part line the end-of-part number, the
    gap marker and the next part's `MP 0` overprinted into an unreadable `46.10 108mi 0`. Interior
    ticks now keep a 26 px cordon at each end, every part labels its own `MP 0`, and the gap label is
    rotated vertically into its own hatched column.
11. **The network map opened on the whole western United States.** It is constructed inside a `hidden`
    page, so its container had zero size and the boot-time `fitBounds` was computed against nothing;
    revealing the page resized it but never re-fitted. It is now fitted on first reveal.

**The lesson worth keeping** — the sibling recipe's lesson arriving from the other direction: a suite
that greps markup and asserts numbers **cannot see what a screen means**. All four of these were
semantic or spatial — invisible to 297 passing assertions, obvious in one screenshot. The static and
render harnesses catch *wiring and arithmetic*; only rendering catches *meaning*. Guards for all four
now exist (`test-strip.mjs` §6/§6b, `test-render.mjs` §3b/§8/§9) and the suite grew to **337**.

12. **The interaction surface was thinner than the design specified, and one of its gaps was a bug.**
    Three things §2 calls for were missing or dead: the **ruler brush** (`rangeSelect`, connections
    #9/#10/#11), the **deep link** (`setUrlParam`, #5/#11), and the four **operator CSV** drop zones,
    which were declared placeholders. All three now work, together with axis zoom, `←`/`→` feature
    walking, and the map driving the strip cursor back (the guideline requires master-detail to be
    *"bidirectional or not at all"*, and only one direction was wired). Building the CSV path exposed a
    real defect: **`PLINEID` is a zero-padded `String(4)`**, and a reader that coerces `"0274"` to the
    number `274` makes every row of an operator's file miss its line — silently, reporting a cheerful
    *"0 matched"*. `isNumericCell()` now refuses to coerce anything carrying a leading zero.
    *Identifiers are strings; only magnitudes are numbers.* Guards: `test-strip.mjs` §15b,
    `test-render.mjs` §§11–13. The suite grew to **422**.

13. **The map had no controls a business reader could use, and the theme was decided on taste.**
    There was no way to turn a layer off at all — on a line that is 100 % unevaluated, an amber wash
    you cannot lift. There is now **one inline-SVG control cluster** (zoom · zoom-out · zoom-to-line ·
    layers · basemap · legend) with MapLibre's own zoom box removed, a **layers drawer** naming all
    seven layers in plain English with their authority and a live feature count, and a **basemap
    drawer** naming the five keyless options by purpose. Two defects fell out of building it: hazard
    alpha tuned for pale CARTO Positron is **invisible on OpenTopoMap** — the basemap a user picks
    *precisely* to see geohazard context — so `hazardOpacity()` now follows the basemap; and the lane
    label column truncated *"Wall thickness · grade · seam"* to *"Wall thickness · ..."* for the
    compliance lead this pack is written for.
    **And the theme question got a measurement instead of an opinion** (§15c). It first said *dark*:
    the never-evaluated amber measured **3.77:1** on light against **7.12:1** on dark. But every other
    light state sat 5.57–8.34:1 — **the amber was the outlier, not the theme**, and it is the one
    state drawn as a hatch, which shows half ground and halves apparent contrast. `#b06f00` →
    `#8a5700` puts it at **5.60:1**. Both modes now clear 4.5:1 on every informational state, so
    contrast decides nothing and **light stands on fitness for purpose**: the pack prints, the artifact
    is a white-paper alignment sheet, and dark would borrow an operations-room urgency that a
    2018 centerline and multi-year hazard cycles cannot support. Suite: **468**.

**What is not built:** the `<StrataApp>` path. §2.1 specifies a full `AppLayout` (registry widgets, 19
`connections`, `sourceId`/`fromWidget` linking, one §10.3 app-local widget with a named fallback) and
serializes it — but rendering it needs `strata/packages/*` installed and built, and `strata/node_modules`
is absent from this checkout. The app here is the standalone implementation of that same design, exactly
as the sibling recipes ship theirs. The `AppLayout` is authored and unrendered; the two must be
reconciled before this recipe is called done.

---

## 0. Provenance

| | |
|---|---|
| **Source** | `https://tabaqat.net/solutions` — **Energy & Utilities** section |
| **Name on site** | Pipeline Integrity Evidence Pack |
| **Folder** | `energy_pipeline-integrity-evidence-pack` |
| **Researched & designed** | 2026-08-06 |
| **Template** | **`open-design` — "alignment-strip"**. No template was assigned to the Energy sector before this recipe; nothing is released back |
| **Demo region** | California, statewide — extent `-124.41, 32.53 → -114.13, 41.99`. Default line **PLINEID 0274** (Kinder Morgan, refined product, 91.05 mi, single-part) |
| **Regulatory frame** | 49 CFR **Part 195** (hazardous liquid) + **Gov. Code §51013.1 / AB 864**. **Not** Part 192 — see §1.8 |
| **Backend** | read-only. Strata Serve **or** ArcGIS. The only write-shaped path is a client CSV upload into a `FileDataSource`, which writes nothing to any service |

---

## 1. Study — how the market frames this

### 1.1 The question the buyer asks

> *"The State Fire Marshal's inspector is in the conference room next Tuesday with the Part 195 protocol.
> He is going to pick a line, pick a milepost, and ask me to show him the record. I know my ILI data. What
> I do not know is which lines of the pack are empty before he finds them empty — and for the ones that
> are empty because nobody in California has ever mapped that ground, I need to be able to say so and
> prove it."*

The product is not a risk score. It is **a defensible record of what is known, by whom, and when** —
assembled per line, per milepost.

### 1.2 The rules that decide it

| Rule | Authority | Requires |
|---|---|---|
| Integrity management in HCAs — identify, assess, evaluate, repair | **49 CFR §195.452** | the programme itself |
| **Data integration** — gather and integrate *all* available information about the entire pipeline | **§195.452(g)** | the pack. This is the clause the whole app serves |
| Baseline + **reassessment at ≤ 5-year intervals** | **§195.452(b), (j)(3)** | assessment dates per segment |
| Repair conditions — immediate / 60-day / 180-day | **§195.452(h)** | anomaly and dig records |
| HCA identification (populated areas, drinking-water and ecological USAs, navigable waterways) | **§195.450 / §195.452(a)** | the HCA segment table |
| Records for the **useful life of the pipeline** demonstrating compliance | **§195.452(l)** | retention, and traceability |
| External corrosion control / CP criteria | **§195.573**, AMPP-NACE **SP0169** | survey records |
| Accident reporting | **§195.50, §195.52** | incident history |
| **Risk analysis + best available technology** on intrastate lines | **Gov. Code §51013.1** (**AB 864**, 2015, post-Refugio), 19 CCR §2020 et seq. | a filed analysis, OSFM-reviewed |
| Zones of required investigation — surface fault rupture | **Alquist-Priolo Act, PRC §2621 et seq.** | a site investigation before certain projects |
| Zones of required investigation — liquefaction, seismic landslide | **Seismic Hazards Mapping Act, PRC §2690 et seq.** | *where the state has mapped* |

**The words that matter.** PHMSA's own inspection language for records is **traceable, verifiable, and
complete** — the "TVC" standard. It is a *records* standard, not a data standard: a number is not
compliant because it is right, it is compliant because you can show where it came from. That distinction
is the whole design (§2.1).

### 1.3 Three findings, measured

All three were computed on **2026-08-06** from live services, geodesically, over all 707 California
regulated intrastate hazardous-liquid pipelines (**6,093.1 measured miles**; operators report
**6,172.48**, a **+1.30 %** difference that validates both figures). Reproduce every one from §4.

**① 62.1 % of the network runs through ground nobody has evaluated.**

```
     network                             6,093.1 mi   707 lines
  ├─ in a Liquefaction Zone of
  │  Required Investigation              1,086.3 mi   17.8 %   335 lines
  ├─ evaluated, and NOT zoned            1,223.8 mi   20.1 %
  └─ NEVER EVALUATED                     3,783.0 mi   62.1 %   319 lines
                                         ─────────────────────
     zoned x unevaluated overlap               0.0 mi   (clean partition, sums to 100.0 %)

     in an Alquist-Priolo Earthquake
     Fault Zone                              61.8 mi    1.01 %   88 lines
```

The Seismic Hazards Mapping Program publishes **2,806** *Unevaluated Areas* against **738** zone
polygons — the unmapped set is 3.8× the mapped one. So on nearly two-thirds of California's regulated
liquid mileage, "not in a liquefaction zone" does **not** mean "no liquefaction hazard". It means *no
one has looked*. Four operators sit at **100 %** unevaluated (El Paso Natural Gas, Plains Marketing,
Plains Pipeline, Crestwood West Coast); Phillips 66 at 86.8 %; Kinder Morgan at 79.4 % over 1,132 miles.

And the layer cannot even tell you *which* quadrangle: **`MAPNAME_1` — the one field that looks like
it names the 7.5-minute quad — is whitespace-only on all 2,806 rows** (so are `LONGITUDE1` and
`LATITUDE_S`). Verified: `where=MAPNAME_1 IS NOT NULL AND MAPNAME_1 <> ''` returns **0 of 2,806**. An
unevaluated reach can therefore be *located* but not *named*, so the app exports its station range and
extent — *"commission a study from MP 0.00 to MP 91.05"* — rather than a quadrangle. The gap is one
layer deeper than it first appears.

**② 28.3 % of the network has no machine-readable diameter.**

`PIPE_SIZE` is a `String(254)` with **84 distinct forms**: multi-diameter lists (`'8, 10, 12'`,
`'4, 6, 8, 16'`), inch marks (`'8"'`), a mixed form (`'6", 8 "'`), an outside-diameter decimal
(`'8.625'`), comma styles with and without spaces, and **whitespace-only on 6 lines**. It is not a single
plain number on **169 lines = 1,723.1 mi = 28.3 %** of the network. Diameter is a primary material
attribute under §195.452(g), the input to every ILI tool selection and every burst-pressure calculation.
The public record cannot supply it as a number on more than a quarter of the mileage.

**③ 33.1 % of the network sits on a centerline that is not continuous.**

**145 of 707 lines (20.5 %) are multipart**, carrying **2,019.1 mi (33.1 %)**. One line has **80 parts**.
The worst inter-part gap is **108.1 miles** (PLINEID 1341, El Paso Natural Gas — 5 parts, 217.6 mi). The
parts carry no order and no measure.

This is the constraint that decides the design. A milepost axis computed as "cumulative distance along
the geometry" would silently **teleport up to 108 miles** between parts, and every station printed on the
evidence pack downstream of that jump would be wrong. The app therefore **refuses to synthesise a
continuous axis**: the ruler breaks where the record breaks, and the break is labelled with its gap
(§2.1). That refusal is the single most important honest act in the app.

### 1.4 The evidence horizon — the pack has 12 lines and the public record fills 3

| # | Evidence line | Citation | Public record |
|---|---|---|---|
| 1 | Commodity | §195.452(g) | ✅ `COMMODITY`, 7 classes |
| 2 | Nominal diameter | §195.452(g) | ⚠️ **present, not verifiable** — 84 string forms, unusable on 28.3 % |
| 3 | Length / stationing | §195.452(g) | ⚠️ **present, contradictory** — reported 6,172.48 vs measured 6,093.1 mi; discontinuous on 33.1 % |
| 4 | Wall thickness · grade · seam type · manufacturer | §195.452(g) | ⛔ **absent** — the layer has 5 descriptive fields, none of them a material property |
| 5 | Install year / construction records | §195.452(g) | ⛔ **absent** — no date field of any kind |
| 6 | MAOP + its supporting record | §195.406 | ⛔ **absent** |
| 7 | HCA segment identification | §195.450, §195.452(a) | ⛔ **absent by policy** — NPMS releases HCA only to operators and government officials; the Public Viewer serves one county per session and excludes it |
| 8 | Baseline + reassessment dates | §195.452(b), (j)(3) | ⛔ **absent** |
| 9 | Anomaly / repair / dig records | §195.452(h) | ⛔ **absent** |
| 10 | Cathodic protection surveys | §195.573, SP0169 | ⛔ **absent** |
| 11 | Incident / release history | §195.50, §195.52 | ⛔ **not serviceable** — see the probe below |
| 12 | AB 864 risk analysis + BAT | Gov. Code §51013.1 | ⛔ **absent** — filed with OSFM, not published |
| 13 | Surface fault rupture | PRC §2621 | ✅ 61.8 mi on 88 lines |
| 14 | Liquefaction | PRC §2690 | ◐ **17.8 % zoned · 20.1 % clear · 62.1 % never evaluated** |

PHMSA does publish incident data — but not as a service:

```
$ curl -s "https://data.transportation.gov/resource/qdme-9bbm.json?\$limit=1"
{"error":true,"message":"no row or column access to non-tabular tables"}
```

Every pipeline-incident dataset in that catalogue resolves to `type: href` — a download, not an
endpoint. So line 11 cannot be assembled live by any application, ours included. It enters through the
client-CSV path or not at all.

**Three of fourteen lines of the pack are answerable from the public record.** The app says so on its
face, and — this is the point — it says so *per milepost*, so the statement is a work list rather than a
disclaimer.

### 1.5 Reference solutions (benchmark + coexist, never copy)

**The incumbent is the alignment sheet**, and it is 100 years old. Every product below either produces
one or feeds one.

| Vendor / product | Owns | Blind to |
|---|---|---|
| **New Century Software (MISTRAS)** — *SheetCutter Pro* + *TemplateDesigner Pro* + *Facility Manager*; Esri Partner, Fort Collins, founded 1994; the default choice for operators standardised on ArcGIS | automated alignment-sheet generation with dynamic segmentation over LRS and non-LRS sources (geodatabase, Oracle, SQL Server, Excel); pipeline data QC | it *renders* the sheet beautifully. The sheet asserts values; it does not carry, per value, the provenance an inspector asks for |
| **DNV — Synergi Pipeline Integrity** | risk-based inspection planning, corrosion prediction, defect assessment, fitness-for-service | the engine is the product, so the arithmetic is necessarily opaque; a "we cannot know this" panel is commercially impossible inside it |
| **ROSEN — ROSYMS PIM / integrity data warehouse** | ILI-to-integrity lifecycle, API 1160 / CSA Z662 compliance, GIS integration, dashboards | strongest where the ILI run exists; the pack's *empty* lines are outside its frame |
| **Baker Hughes** · **Metegrity (Visions)** · **Dynamic Risk (IPM)** | enterprise integrity with SAP/Oracle work-order flow-through; quantitative risk | enterprise-priced, enterprise-paced, and the record-quality story is an internal report, not a screen |
| **EnerSys — POEMS Program Suite** (`ComplyMgr`, `ICAM`, `pSEc`) | documenting the *programme* and proving Plan-Do-Check-Act to a regulator | programme-level, not milepost-level; it proves the process ran, not that the ground under MP 47 was ever mapped |
| **Audubon**, **Geosyntec**, **Teren** and the API RP 1187 / INGAA geohazard consultancies | landslide and geohazard programmes; the 2023 INGAA *Framework for Geohazard Management* and API **RP 1187** (2024) both name **data management** as the critical component | consulting engagements produce a report at a date, not a standing instrument the operator drives |
| **Esri** — ArcGIS Pipeline Referencing, UPDM; **PODS** the data model | the LRS itself; the schema everyone else conforms to | a model, not an answer. Nothing in PODS records *whether an authority has ever evaluated the ground* |
| **PHMSA NPMS** · **CA OSFM Pipeline Safety Division** | the authoritative registers | NPMS is deliberately restricted (one county per public session, no HCA); OSFM's estate returns **HTTP 403** to non-browser clients |

**The gap.** Every product above is **assertive**: it shows you a value. Not one is **evidential**: none
shows you, per value, *which authority produced it, on what date, in answer to what query, and what it
means that the answer is empty*. The industry's own standard for records is traceable-verifiable-complete
— and there is no screen anywhere in this market whose unit of display is the traceability of a value
rather than the value.

### 1.6 Our edge

AI-authored, MIT/open, runs on **Strata *or* ArcGIS**, sovereign/on-prem. And the structural one: **we can
render an empty lane with its citation, because we have no engine to protect.** A vendor whose risk model
*is* the product cannot ship a strip that is two-thirds hatched — it would be marketing against itself.
For an evidence pack, the hatching is the most valuable ink on the page.

Coexistence: New Century keeps the alignment sheet; Synergi and IPM keep the risk model; PODS/UPDM keeps
the schema; ROSEN keeps the ILI. This sits **upstream of all of them** and answers a question none of
them is shaped to ask.

### 1.7 Standards, statutes & organizations to speak fluently

- **49 CFR Part 195** — hazardous liquid. §195.450 (HCA definitions), **§195.452** (IM; **(g)** data
  integration, **(h)** repair conditions, **(j)(3)** 5-year reassessment, **(l)** records), §195.406
  (MAOP), §195.573 (CP), §195.50/§195.52 (accident reporting).
- **49 CFR Part 192** — gas. Named here only for contrast: **§192.607** (material verification) and
  **§192.624** (MAOP reconfirmation — **50 % of mileage by 3 July 2028, 100 % by 2 July 2035**) are the
  loudest "records are the deliverable" clauses in US pipeline regulation, and they are the reason the
  TVC vocabulary is now industry-wide. **Out of scope for the CA demo** — see §1.8.
- **TVC** — *traceable, verifiable, complete*. PHMSA's records standard.
- **AB 864 (2015)**, Gov. Code **§51013.1**, 19 CCR **§2020** — California's post-**Refugio** (Plains
  All American Line 901, May 2015) requirement for a risk analysis and best available technology on
  intrastate lines in environmentally and ecologically sensitive areas.
- **CA OSFM Pipeline Safety Division** (CAL FIRE) — regulates intrastate hazardous liquid pipelines,
  including offshore state waters. **CPUC** regulates intrastate gas (GO 112-F). **PHMSA** regulates
  interstate.
- **Alquist-Priolo Earthquake Fault Zoning Act** PRC §2621 et seq. · **Seismic Hazards Mapping Act**
  PRC §2690 et seq. · **CGS** publishes both.
- **API RP 1187** (2024, landslide hazards) · **API RP 1160** (liquid IM) · **INGAA** *Framework for
  Geohazard Management* (2023) · **ASME B31.4/B31.8S** · **AMPP** (ex-NACE) **SP0169**.
- **PODS** (Pipeline Open Data Standard) · Esri **UPDM** / **ArcGIS Pipeline Referencing** · **NPMS**.
- **ILI** vocabulary: MFL, UT, EMAT, IMU/geometry, ERF, dig sheet, `station`/`chainage`/`engineering
  station`, alignment sheet, dynamic segmentation.

### 1.8 Honest scope — what this is **not**

- **Not a natural-gas application.** A keyword sweep of the CA catalogue's 2,569 endpoints returns **no
  gas transmission or distribution centerline** of any kind. Part 192, §192.624 and the MAOP-reconfirmation
  clock are therefore discussed but never *rendered*; the demo is Part 195 / AB 864 only.
- **Not the operator's LRS.** Stationing is measured along the **published 2018 centerline** in its
  **digitised direction**, which need not match the operator's engineering stationing or its origin. Every
  milepost on screen and in every export is labelled *"measured, published centerline"*.
- **Not a risk score.** Nothing here is ranked, weighted or coloured by consequence. See the rejected
  rival in §2.1.
- **Not an HCA tool.** HCA data is deliberately not public (§1.4 line 7). No HCA is drawn, inferred or
  approximated anywhere.
- **Not positionally precise.** A 2018 compilation last edited 2021-02-05, at statewide scale. No claim
  is made tighter than "this line intersects this zone", and never "the pipe is *n* feet from the fault".
- **Not a substitute for the site investigation.** An unevaluated band tells you a study is required. It
  does not tell you the answer the study would give.
- **Not a submission.** Nothing here is filed with OSFM or PHMSA. It is the working paper you assemble
  *before* you write what you file.

---

## 2. UI design spec (front-loaded)

### 2.1 Layout (Template: **`open-design` — "alignment-strip"**)

- **Template** `open-design` under the freestyle charter (`strata/recipes/COMPONENT-MANIFEST.md` §10):
  registry widgets and manifest config keys only, plus **one** §10.3 app-local widget with a named
  fallback (§2.6). **Never** fall back to `split-dashboard`.
- **Why this shape:** a pipeline is the one asset class whose world is **one-dimensional**. Its engineers
  do not navigate by extent, they navigate by station — and their native artifact, the alignment sheet, is
  a horizontal ruler with stacked tracks. So the navigation *is* the milepost axis, and the map is demoted
  to a locator that follows the strip cursor. Nothing else in the roster inverts map and chart this way.
- **The ink is provenance, not value.** Every band on every lane is painted in one of **four evidence
  states**, and that vocabulary is the product:

  | State | Means | Ink | Example |
  |---|---|---|---|
  | ✔ **Evidenced** | an authority answered, with a value | solid | AP zone *Vine Hill*, MP 0.00–0.33 |
  | ○ **Evidenced-absent** | an authority evaluated and found nothing | hollow outline | liquefaction: evaluated, not zoned |
  | ▨ **Never evaluated** | no authority has looked — absence is not evidence | **hatched** | 62.1 % of the network |
  | ⛔ **Operator-only** | structurally not public; only your record fills it | empty lane + citation | wall thickness, ILI, CP, HCA |

- **Signature loop:** `pick a line → each lane queries its own authority and stamps a provenance chip
  (service · query · UTC time · row count) → the coverage read-out says "3 of 14 lines evidenced · 100 %
  of this line never evaluated" → click a band → the map flies to that milepost and the dossier opens →
  export the Evidence Pack with every provenance chip on it.`
- **The signature accent is the broken ruler.** On a multipart line the axis is drawn with a visible
  break carrying its gap (`▮ 108.1 mi — parts not connected in the published centerline`). No other app
  in the library refuses to draw its own navigation, and this one does it 145 times.
- **Two pages:** `pack` (one line — the strip, the map, the pack) and `network` (all 707 — where do I
  start, and who is exposed).
- **Three drag handles** on page 1 (lines ⇄ centre, strip ⇄ map, centre ⇄ pack).
- **Wiring floor:** **19 connections** authored, ≥3 required (`docs/guide/app-design.md` §3). Eight more
  widgets link with **zero** connections through `dataSource.sourceId` / `fromWidget`.
- **Responsive:** `responsive.small` collapses both splitters to a column; the lines panel becomes
  `dock:"top"`; **the strip keeps its horizontal axis and scrolls** (never re-flows to vertical — a
  vertical milepost axis is a different instrument, and it is `transportation_airports-aviation`'s);
  map at 40 vh with a **240 px floor**.

#### The two rivals, rejected on integrity grounds

Both were designed for this business. Both die on the data, not on taste.

- **`reassessment-clock`** — a time axis to the next required assessment: §195.452(j)(3) caps
  reassessment at five years, so plot every segment by time-to-deadline and the app becomes a countdown.
  It is the most *commercially* attractive shape here and the one a buyer would ask for. **It dies because
  the public record contains no assessment date of any kind** (§1.4 line 8). Every position on the clock
  would be inferred, and an interface that tells an operator their reassessment is overdue — on a
  fabricated date — is worse than no interface. The idea survives, correctly scoped, as a lane that lights
  up the moment the client's CSV arrives.
- **`risk-surface`** — a computed likelihood × consequence score per segment, painted red/amber/green.
  This is the market default: Synergi, IPM, ROSYMS and every consultancy deliverable is some version of
  it. **It dies because both factors are missing.** Likelihood needs wall thickness, age, coating, CP and
  ILI — none public (§1.4 lines 4, 5, 9, 10). Consequence needs HCA — withheld by policy (line 7). A score
  computed from five public string fields, two of which are unreliable (§1.3 ②, ③), would carry the
  authority of a risk model with the content of a guess. This market has too much of that already; being
  the tool that declines is the position worth holding.

A third, `protocol-board` — navigate by the inspector's own Part 195 protocol questions — was rejected
on **collision** grounds rather than integrity: it is a checklist with a map beside it, it makes the
spatial data decorative, and it reads as a near neighbour of `education_campus-operations`'
`standards-rack`. Its good idea (the regulator's own document as the index) survives as the pack's row
order and citations.

#### ASCII skeleton — page 1 `pack` (desktop)

```
┌ HEADER ──────────────────────────────────────────────────────────────────────────────────────┐
│ Pipeline Integrity Evidence Pack · the alignment strip  [operator ▾][commodity ▾] ⟨1⟩⟨2⟩ ☀/☾ │
│ CA intrastate hazardous liquid · centerline 2018 (last edited 2021-02-05) · assembled 12:04Z  │
├────────────┬───────────────────────────────────────────────────────┬─────────────────────────┤
│ LINES (707)│  0274 · Kinder Morgan · Refined Product · 10" · 91.05 mi (reported 91.21)        │
│ search…    │                                                       │  EVIDENCE PACK          │
│ ▸0274 KM   │  MP 0 ──10──20──30──40──50──60──70──80──91.05         │  ●●●●●○○○○○○○○○  5 of 14 │
│  91.05 mi  │  ─────────────────────────────────────────────────    │  ┌───────────────────┐  │
│ ▸1341 EPNG │  commodity   ████████████████████████████████████ ✔   │  │ 1 commodity    ✔  │  │
│  217.6 mi ⚠│  diameter    ████████████████████████████████████ ⚠   │  │   Refined Product │  │
│ ▸0987 PPS  │  stationing  ████████████████████████████████████ ⚠   │  │   ⓘ CA_Oil_Pipe…  │  │
│  119.1 mi ⚠│  wall/grade  ································· ⛔      │  │     /0 · 12:04:07Z│  │
│ ▸0462 P66  │  install yr  ································· ⛔      │  │     1 row · 41 ms │  │
│  182.5 mi ⚠│  MAOP        ································· ⛔      │  ├───────────────────┤  │
│ …          │  HCA         ································· ⛔ §195.450                     │
│            │  reassess    ································· ⛔ §195.452(j)(3)               │
│ [filters]  │  anomaly/dig ································· ⛔ §195.452(h)                  │
│  operator  │  CP survey   ································· ⛔ §195.573                     │
│  commodity │  incidents   ································· ⛔ not serviceable              │
│  ⚠ multipart│ AB 864 RA   ································· ⛔ §51013.1                     │
│            │  fault rupt. ▌0.00–0.33 Vine Hill ▌0.33–0.39 Walnut Creek  ✔                    │
│            │  liquefaction ▨▨▨▨▨▨▨▨▨▨▨▨▨▨▨▨▨▨▨▨▨▨▨▨▨▨▨▨▨▨▨▨▨▨▨▨  ▨ 100% never evaluated   │
│            │  ─────────────────────────────────────────────────    │  PROVENANCE (12 rows)   │
│            ├───────────────────────────────────────────────────────┤  lane·authority·service │
│            │  MAP (locator, boxed) — cursor at MP 20.72            │  ·query·asOf·rows·state │
│            │  ┌ legend ────┐          ┌ ctl cluster ┐              ├─────────────────────────┤
│            │  │ ✔ evidenced│          │ + − ⌂ ≡ ▤ ☰ │              │  DOSSIER (band detail)  │
│            │  │ ▨ unevaluat│          └─────────────┘              │  Quaternary fault       │
│            │  │ ⛔ operator │   lon,lat · z · EPSG:4326             │  crossing · MP 20.72    │
│            │  └────────────┘                                       │  FLT_NAME · FLT_AGE     │
├────────────┴───────────────────────────────────────────────────────┴─────────────────────────┤
│ Source: Cal OES · CA State Geoportal (CGS) · CA Dept. of Conservation.  Stationing is measured│
│ along the published centerline in its digitised direction — not the operator's LRS.  [share][⎙]│
└──────────────────────────────────────────────────────────────────────────────────────────────┘
```

On a **multipart** line the ruler is drawn broken:

```
  MP 0 ──10──20──30──40──45.2 ▮ 108.1 mi gap ▮ 0 ──10──20──30──40──50──60──64.3      (part 2 of 5)
       └── part 1 ──────────┘   parts not connected in the published centerline   └── part 2 ──┘
```

**Phone (`responsive.small`).** Both splitters collapse to a column. Order: lines (as `panel dock:"top"`,
`flow-row` of chips) → the strip, **still horizontal, horizontally scrollable** → coverage chips → map at
40 vh (240 px floor) → the pack → provenance.

#### The `AppLayout` JSON

```jsonc
{
  "version": "1",
  "theme": { /* §2.2 */ },
  "splash": { "title": "This is a working paper, not a filing",
    "body": "It assembles what the public record can prove about a California intrastate hazardous liquid pipeline, milepost by milepost, and names what it cannot. Three of the fourteen lines of a Part 195 evidence pack are answerable from public data; the rest are yours. Stationing is measured along the published 2018 centerline in its digitised direction — it is not your LRS. Nothing here is filed with OSFM or PHMSA.",
    "dismissible": true, "once": true },
  "pages": [
    { "id": "pack", "type": "fixed",
      "header": { "kind": "row", "gap": 12, "children": [
        { "kind": "widget", "widget": { "id": "hdr", "type": "text",
            "props": { "content": "Pipeline Integrity Evidence Pack — the alignment strip", "as": "h3" } } },
        { "kind": "widget", "widget": { "id": "hdr-vintage", "type": "text",
            "props": { "as": "body2", "content": "CA intrastate hazardous liquid · centerline 2018 (service last edited 2021-02-05)" } } },
        { "kind": "widget", "widget": { "id": "flt", "type": "filter",
            "props": { "layerId": "pipes", "fields": [
              { "name": "OPERATOR",  "label": "Operator",  "type": "string" },
              { "name": "COMMODITY", "label": "Commodity", "type": "string" } ] } } },
        { "kind": "widget", "widget": { "id": "nav",      "type": "page-nav",     "props": { "variant": "tabs" } } },
        { "kind": "widget", "widget": { "id": "theme-sw", "type": "theme-switch", "props": { "initial": "light" } } }
      ]},
      "root": { "kind": "splitter", "orientation": "h", "sizes": [18, 56, 26], "minSizes": [12, 34, 18], "children": [

        { "kind": "panel", "dock": "left", "title": "Lines (707)", "width": 300,
          "responsive": { "small": { "dock": "top" } }, "children": [
          { "kind": "widget", "widget": { "id": "q-line", "type": "query",
              "props": { "fields": [
                { "name": "PLINEID",  "label": "Line ID",  "type": "string" },
                { "name": "OPERATOR", "label": "Operator", "type": "string" } ] },
              "dataSource": { "sourceId": "pipes" } } },
          { "kind": "widget", "widget": { "id": "lines", "type": "table",
              "props": { "virtualize": true, "viewportHeight": 420, "oidField": "FID",
                "columns": ["PLINEID","OPERATOR","COMMODITY","PIPE_SIZE","TOTAL_MILE"],
                "fieldAliases": { "TOTAL_MILE": "Reported mi", "PIPE_SIZE": "Diameter (as filed)" } },
              "dataSource": { "layerId": "pipes", "fromWidget": "q-line" } } } ]},

        { "kind": "splitter", "orientation": "v", "sizes": [58, 42], "minSizes": [30, 20], "children": [
          { "kind": "column", "gap": 6, "children": [
            { "kind": "widget", "widget": { "id": "strip", "type": "alignment-strip",
                "dataSource": { "fromWidget": "xsect-ap" },
                "props": {
                  "units": "mi", "showProvenance": true, "breakOnMultipart": true,
                  "asOf": "2026-08-06",
                  "axisLabel": "Measured along the published centerline (digitised direction) — not the operator's LRS",
                  "lanes": [
                    { "id": "commodity", "label": "Commodity",            "cite": "§195.452(g)", "state": "evidenced",   "field": "COMMODITY" },
                    { "id": "diameter",  "label": "Diameter",             "cite": "§195.452(g)", "state": "unverifiable", "field": "PIPE_SIZE",
                      "note": "String(254), 84 forms — unusable as a number on 28.3% of the network" },
                    { "id": "station",   "label": "Stationing",           "cite": "§195.452(g)", "state": "unverifiable",
                      "note": "reported vs measured differ; 33.1% of the network is discontinuous" },
                    { "id": "material",  "label": "Wall · grade · seam",  "cite": "§195.452(g)", "state": "operator-only", "fromWidget": "file-material" },
                    { "id": "install",   "label": "Install year",         "cite": "§195.452(g)", "state": "operator-only" },
                    { "id": "maop",      "label": "MAOP",                 "cite": "§195.406",    "state": "operator-only" },
                    { "id": "hca",       "label": "HCA segments",         "cite": "§195.450",    "state": "operator-only", "fromWidget": "file-hca",
                      "note": "withheld from the public by NPMS policy" },
                    { "id": "reassess",  "label": "Baseline / reassessment", "cite": "§195.452(j)(3)", "state": "operator-only", "fromWidget": "file-ili" },
                    { "id": "anomaly",   "label": "Anomaly / repair",     "cite": "§195.452(h)", "state": "operator-only", "fromWidget": "file-ili" },
                    { "id": "cp",        "label": "Cathodic protection",  "cite": "§195.573",    "state": "operator-only", "fromWidget": "file-cp" },
                    { "id": "incident",  "label": "Incident history",     "cite": "§195.50",     "state": "operator-only",
                      "note": "PHMSA publishes flagged files, not a service" },
                    { "id": "ab864",     "label": "AB 864 risk analysis", "cite": "Gov. Code §51013.1", "state": "operator-only" },
                    { "id": "rupture",   "label": "Surface fault rupture","cite": "PRC §2621",   "state": "evidenced",    "fromWidget": "xsect-ap" },
                    { "id": "qfault",    "label": "Quaternary fault crossings", "cite": "CGS FAM", "state": "evidenced",  "fromWidget": "xsect-qf" },
                    { "id": "liquef",    "label": "Liquefaction",         "cite": "PRC §2690",   "state": "tri",
                      "fromWidget": "xsect-liq", "absentFromWidget": "xsect-unev",
                      "note": "zoned / evaluated-and-clear / never evaluated" } ] } } },
            { "kind": "widget", "widget": { "id": "gaps", "type": "text",
                "props": { "as": "body2", "content": "Ruler breaks mark parts that are not connected in the published centerline." } } } ]},

          { "kind": "section", "mode": "fixed", "children": [
            { "kind": "widget", "widget": { "id": "map", "type": "map",
                "props": { "config": { "$ref": "layers.json" },
                  "controls": { "navigation": true, "scale": true, "legend": true,
                                "basemapSwitcher": true, "layerList": true } } } },
            { "kind": "widget", "widget": { "id": "statusbar", "type": "status-bar",
                "props": { "crs": "EPSG:4326", "precision": 5 } } } ]}
        ]},

        { "kind": "panel", "dock": "right", "title": "Evidence pack", "width": 380, "children": [
          { "kind": "flow-row", "gap": 8, "children": [
            { "kind": "widget", "widget": { "id": "kpi-lines", "type": "kpi",
                "props": { "label": "Pack lines evidenced", "value": 3, "unit": "of 14" } } },
            { "kind": "widget", "widget": { "id": "gauge-unev", "type": "gauge",
                "props": { "label": "% never evaluated", "thresholds": [20, 50], "invertColors": true,
                           "stat": { "field": "unev_mi", "op": "sum" } },
                "dataSource": { "fromWidget": "xsect-unev" } } },
            { "kind": "widget", "widget": { "id": "kpi-ap", "type": "kpi",
                "props": { "label": "Fault-zone miles", "status": "warn",
                           "stat": { "field": "band_mi", "op": "sum" } },
                "dataSource": { "fromWidget": "xsect-ap" } } } ]},
          { "kind": "widget", "widget": { "id": "pack", "type": "table",
              "props": { "oidField": "id", "viewportHeight": 300,
                "columns": ["#","evidence line","citation","state","what fills it"] } } },
          { "kind": "accordion", "titles": ["Authorities", "Provenance", "Dossier", "Bring your records"], "children": [
            { "kind": "column", "gap": 6, "children": [
              { "kind": "widget", "widget": { "id": "xsect-ap", "type": "analysis",
                  "props": { "tools": ["clip"], "title": "Alquist-Priolo Earthquake Fault Zones (PRC §2621)" },
                  "dataSource": { "sourceId": "apzones" } } },
              { "kind": "widget", "widget": { "id": "xsect-liq", "type": "analysis",
                  "props": { "tools": ["clip"], "title": "Liquefaction Zones of Required Investigation (PRC §2690)" },
                  "dataSource": { "sourceId": "liq" } } },
              { "kind": "widget", "widget": { "id": "xsect-unev", "type": "analysis",
                  "props": { "tools": ["clip"], "title": "Unevaluated Areas — nobody has looked (no nameable quad)" },
                  "dataSource": { "sourceId": "unev" } } },
              { "kind": "widget", "widget": { "id": "xsect-qf", "type": "analysis",
                  "props": { "tools": ["clip", "spatial-join"], "title": "Quaternary fault crossings (CGS FAM)" },
                  "dataSource": { "sourceId": "qfaults" } } } ]},
            { "kind": "widget", "widget": { "id": "prov", "type": "table",
                "props": { "oidField": "id", "viewportHeight": 220,
                  "columns": ["lane","authority","service","query","asOf (UTC)","rows","ms"] } } },
            { "kind": "widget", "widget": { "id": "dossier", "type": "feature-info",
                "props": { "emptyText": "Click a band on the strip, or a feature on the map." },
                "dataSource": { "fromWidget": "xsect-ap" } } },
            { "kind": "column", "gap": 6, "children": [
              { "kind": "widget", "widget": { "id": "file-material", "type": "add-data", "props": { "title": "Material properties CSV" } } },
              { "kind": "widget", "widget": { "id": "file-ili",      "type": "add-data", "props": { "title": "ILI / dig CSV" } } },
              { "kind": "widget", "widget": { "id": "file-cp",       "type": "add-data", "props": { "title": "CP / CIS CSV" } } },
              { "kind": "widget", "widget": { "id": "file-hca",      "type": "add-data", "props": { "title": "HCA segment CSV" } } } ]} ]},
          { "kind": "widget", "widget": { "id": "acts", "type": "data-actions", "props": { "hideWhenEmpty": true } } } ]}
      ]},
      "footer": { "kind": "row", "gap": 12, "children": [
        { "kind": "widget", "widget": { "id": "attrib", "type": "text", "props": { "as": "body2",
            "content": "Cal OES · CA State Geoportal (CGS) · CA Dept. of Conservation. Stationing is measured along the published centerline in its digitised direction — not the operator's LRS. Not a filing." } } },
        { "kind": "widget", "widget": { "id": "share-w", "type": "share" } },
        { "kind": "widget", "widget": { "id": "print-w", "type": "print" } } ]} },

    { "id": "network", "type": "fixed",
      "root": { "kind": "splitter", "orientation": "v", "sizes": [62, 38], "children": [
        { "kind": "splitter", "orientation": "h", "sizes": [66, 34], "children": [
          { "kind": "section", "mode": "fixed", "children": [
            { "kind": "widget", "widget": { "id": "map2", "type": "map",
                "props": { "config": { "$ref": "layers.json" },
                  "controls": { "navigation": true, "scale": true, "legend": true, "layerList": true } } } } ]},
          { "kind": "column", "gap": 8, "children": [
            { "kind": "widget", "widget": { "id": "partition", "type": "stacked-bar",
                "props": { "title": "6,093.1 measured miles by evidence state", "horizontal": true, "thickness": 34,
                  "series": [
                    { "label": "In a liquefaction zone",   "value": 1086.3, "color": "#a4262c" },
                    { "label": "Evaluated, not zoned",     "value": 1223.8, "color": "#2e6f40" },
                    { "label": "Never evaluated",          "value": 3783.0, "color": "#b06f00" } ] } } },
            { "kind": "widget", "widget": { "id": "byop", "type": "chart",
                "props": { "charts": [ { "id": "op-unev", "title": "Never-evaluated miles by operator", "kind": "bar",
                  "source": { "layer_id": "pipes", "field": "OPERATOR", "stat": "count" } } ], "limit": 12 },
                "dataSource": { "sourceId": "pipes" } } },
            { "kind": "widget", "widget": { "id": "sizes", "type": "carto",
                "props": { "title": "The record's own quality", "widgets": [
                  { "id": "w-comm", "kind": "category",  "layerId": "pipes", "field": "COMMODITY", "operation": "count" },
                  { "id": "w-size", "kind": "category",  "layerId": "pipes", "field": "PIPE_SIZE", "operation": "count", "limit": 12 },
                  { "id": "w-mi",   "kind": "histogram", "layerId": "pipes", "field": "TOTAL_MILE" } ] } } } ]}
        ]},
        { "kind": "widget", "widget": { "id": "netTable", "type": "table",
            "props": { "virtualize": true, "viewportHeight": 240, "oidField": "FID",
              "columns": ["PLINEID","OPERATOR","COMMODITY","PIPE_SIZE","TOTAL_MILE"] },
            "dataSource": { "sourceId": "pipes" } } }
      ]},
      "footer": { "kind": "row", "children": [
        { "kind": "widget", "widget": { "id": "print2", "type": "print" } } ]} }
  ],
  "connections": [ /* §2.7 */ ]
}
```

### 2.2 Theme

Structured `ThemeSpec`, **light**, with a real dark mode behind `theme-switch`.

**The reason is domain-specific, not aesthetic.** The artifact this app replaces is the **alignment
sheet** — a white-paper engineering drawing the industry has read for a century and still hands an
inspector on paper. It is the one visual convention every pipeline engineer already parses fluently, and
inverting it to a dark console would spend that fluency for nothing. Every exhibit prints light whichever
mode is on screen.

```jsonc
{ "mode": "light",
  "colors": { "primary": "#1f4e5f", "secondary": "#6b7c85", "success": "#2e6f40", "info": "#1f4e5f",
              "warning": "#b06f00", "danger": "#a4262c", "light": "#f7f5f1", "dark": "#16212a" },
  "fonts": { "scale": "compact", "mono": "ui-monospace, Menlo, monospace" },
  "variables": { "--strata-radius-md": "3px", "--strata-motion-base": "160ms",
                 "--strata-elevation-1": "0 1px 2px rgba(22,33,42,.10)" },
  "overrides": { "kpi": { "--strata-mono": "ui-monospace, Menlo, monospace" } } }
```

Semantic roles are load-bearing: `success` = an authority looked and found nothing · `danger` = an
authority looked and found a hazard · `warning` = **nobody looked**. Note the deliberate inversion —
**amber is the honest colour of ignorance, and it dominates the page** (62.1 % of the network). A viewer
who reads amber as "medium risk" has misread the app, which is why the legend spells every state out
in words and why the hatch exists.

**Six evidence states, not four.** The design shipped four; rendering it added two. `evidenced` had
been assigned the **danger** role, so a commodity — a plain factual value — drew a full-width blood-red
bar and a line with a *complete* record looked like the most alarming thing in the app. A mapped fault
or liquefaction zone now takes its own **`evidenced-hazard`** (danger, *dense* fill); `evidenced` takes
the primary role. `unverifiable` was always in the code and is now stated here too.

| | | ink |
|---|---|---|
| ✔ **Evidenced** | an authority answered, with a value | solid, primary |
| ▲ **Evidenced — hazard present** | an authority answered, and the answer is a mapped hazard | dense 45°, danger |
| ○ **Evidenced-absent** | an authority evaluated this ground and found nothing | hollow outline, success |
| ▨ **Never evaluated** | no authority has looked here | 45° hatch, warning |
| ⚠ **Present, not verifiable** | the record carries a value that cannot be relied on as filed | cross-hatch, warning |
| ⛔ **Operator-only** | structurally not public; only the operator's files can fill it | dotted empty |

**Never encode the states by hue alone.** They differ by **fill pattern first**: solid (evidenced) ·
hollow outline (evidenced-absent) · **45° hatch** (never evaluated) · empty with a dotted baseline
(operator-only). The pack prints, is read in greyscale, and red/green sits at the classic deuteranopia
confusion pair, so the non-colour channel is the primary encoding and hue is the reinforcement. The
legend swatches carry the same patterns so they cannot drift from the strip.

Map ink is theme-aware and lives in one table read by both the map and the legend. Basemap keyless and
mode-paired (CARTO Positron / Dark Matter); `esriSMSCircle`/`esriSMSSquare` only; hazard polygons at
~40/255 alpha so overlapping zones stay readable. Tabular monospaced numerals wherever a station meets a
citation.

**Motion is informative, and only here.** Lanes **fade in as their authority answers** (`animate:"fade"`,
`stagger` off — each lane resolves on its own clock). That is not decoration: it shows the user which
service is slow and makes the assembly legible as an act of evidence-gathering. Nothing else in the app
animates, and the map's container never does.

### 2.3 KPI cards

Three, plus the network partition on page 2. Deliberately few — the strip is the instrument.

| Card | Binding | On line 0274 |
|---|---|---|
| **Evidenced for this line** | live count of lanes resolving to `evidenced`/`tri` | **5 of 14** on 0274 · **4** on 1341 *(structural ceiling: 3)* |
| **% never evaluated** (`gauge`) | `stat {field:"unev_mi", op:"sum"}` ÷ line length, `thresholds:[20,50]`, `invertColors` | **100 %** — all 91.05 mi *(network: 62.1 %)* |
| **Fault-zone miles** | `stat {field:"band_mi", op:"sum"}` over `xsect-ap` | **0.39 mi** *(network: 61.8 mi on 88 lines)* |

The gauge is inverted on purpose: a high number is not a hazard finding, it is an **evidence** finding.
Its label says so.

### 2.4 Charts & tables

- **`stacked-bar` — the network partition** (page 2): horizontal, three series, 1,086.3 / 1,223.8 /
  3,783.0 miles. It sums to 6,093.1 and the overlap is exactly 0.0 mi — the one chart in the app that is
  a proof rather than a summary.
- **`chart` — never-evaluated miles by operator** (page 2): bar, `OPERATOR`, `limit: 12`. Emits
  `categorySelect` → `filter` + `flash`. Verified top rows: Kinder Morgan **898.6 mi (79.4 %)** ·
  Phillips 66 **648.1 (86.8 %)** · Shell **537.2 (79.8 %)** · Crimson **362.1 (38.9 %)** · Pacific
  Pipeline System **291.0 (56.7 %)**; El Paso Natural Gas, Plains Marketing, Plains Pipeline and
  Crestwood West Coast at **100 %**.
- **`carto` — the record's own quality** (page 2): a category widget on `PIPE_SIZE` is the fastest way to
  *see* finding ② — the top twelve values include `'8, 10'`, `'6, 8'`, `'8.625'` and `' '`. Putting the
  register's own dirt on screen as a brushable widget is more persuasive than any sentence about it.
- **`table` — the lines list** (page 1 left): `virtualize`, `oidField: "FID"` **(not `OBJECTID` — trap
  P1)**, five columns, a ⚠ badge on the 145 multipart lines. A row click selects the line.
- **`table` — the pack** (page 1 right): 14 static rows, columns `# · evidence line · citation · state ·
  what fills it`. This is the export's table of contents.
- **`table` — provenance**: one row per lane that queried an authority — `lane · authority · service ·
  query · asOf (UTC) · rows · ms`. **This table is the traceable-and-verifiable half of TVC**, and it is
  the reason the export is worth handing to an inspector.
- **`feature-info` — the dossier**: the clicked band or map feature (`Zone_Name`, `FLT_NAME`, `FLT_AGE`,
  `MAPNAME_1`).

### 2.5 Capabilities to use

Layout `splitter` ×3 · `panel` ×2 · `accordion` · `flow-row` · `section mode:"fixed"` · widgets
`map legend layer-panel basemap table chart carto stacked-bar filter query kpi gauge feature-info
data-actions analysis add-data text status-bar share print theme-switch page-nav` + the one app-local
`alignment-strip` · triggers `filterChange categorySelect rowSelect featureSelect rangeSelect
extentChange recordsChange hover flash` · actions `filter zoomTo panTo flash viewInTable showStatistics
setUrlParam message` · `dataSource.sourceId` and `fromWidget` linking · `@strata/processing`
**clip** + **spatial-join** (the crossings) · `FileDataSource` (the four operator CSVs) · `imageserver`
source kind (PGA) · structured theme · app-shell (`header`/`footer`/`splash`) · `animate:"fade"` on lane
resolution.

**Deliberately not used** — reasons, not omissions. **Time slider / `date-filter`**: the centerline has
no date field at all, and the one date in the system (the service's `lastEditDate`) is a publication
event, not a pipeline event. **Routing / isochrones**: a pipeline is not a road; a drive time along it
is meaningless. **`weighted-overlay`**: weighting these five public fields into a score is precisely the
`risk-surface` rival rejected in §2.1 — shipping the widget would smuggle it back in. **hexbin /
hotspot**: density over 707 statewide polylines is noise, and a hotspot of *centerlines* would be read
as a hotspot of *risk*. **`near-me` / `search` / geocoding**: you do not geocode a pipeline; you station
it. **`swipe` / `compare`**: there is no before/after — one vintage exists. **`bookmarks` /
`exhibit`**: the strip is the navigation. **`views` + `mapState`**: the line selection is the state
machine. **`RestDataSource` / `StreamDataSource`**: the centerline was last edited in 2021 and the
seismic zones move on a multi-year cycle; a live-looking widget would be theatre. **`elevation`**: a
profile would be genuinely apt for a pipeline, but it needs a DEM sampling service we have not verified,
and a fabricated profile on an engineering instrument is unacceptable — see §7.

**Not wired, and why** — `buttonClick`, `mapClick`, `sketchComplete`, `viewChange`, `pageChange`,
`timer`: the trigger types and dispatchers ship but the **widget emitters are pending** (Manifest §4.1).
Consequently this design contains **no `window`** — a modal whose only opener is a `buttonClick` could
never be opened, so the statute text and the "bring your records" uploads live in an always-reachable
`accordion`. Likewise no `navigate` / `refresh` / `selectByGeometry` / `updateRecord`: the dispatchers
ship but `<StrataApp>`'s callbacks are pending, so the crossings are recomputed by the user running the
`analysis` widget rather than by a `refresh` wire. A connection that never fires is worse than none.

**No write path exists**, by design. The four CSV uploads populate client-side `FileDataSource`s; nothing
is written to any service, so there is no `assertEsriBackend` guard and no read-only degradation
question. It runs identically on Strata Serve and ArcGIS.

### 2.6 The one new widget (§10.2/§10.3) and its day-1 fallback

| Key | Purpose | Emits | Fallback that ships day 1 |
|---|---|---|---|
| **`alignment-strip`** | a linear-referenced strip: a milepost ruler (**broken at multipart discontinuities**) plus N evidence lanes, each band painted in one of the six evidence states and carrying its provenance | `rangeSelect` `{layerId, field:"station_mi", min, max}` on a ruler brush · `featureSelect` `{layerId, oids}` on a band click · `hover` | a **`stacked-bar`** per lane (`horizontal: true`, segment widths = band mileages, colours = the six states) + a **`table`** of bands (`lane · from_mi · to_mi · state · authority`) + the header **`filter`**. Same numbers, same wiring shape, no ruler and no visible break |

Everything else — the pack, the provenance table, the dossier, the four uploads — uses **shipped widgets
only** (`table` with static rows, `feature-info`, `add-data`, `accordion`, `text`). Most "missing"
widgets are an existing widget wired differently, and those were.

**Full contract (§10.3):**

| | |
|---|---|
| **Registry key** | `alignment-strip` |
| **Purpose** | Render evidence coverage along a pipeline's measured length. The navigation is the axis; the ink is the evidence state, not the value. |
| **Delivery** | App-local first — `<StrataApp registry={{ "alignment-strip": AlignmentStrip }}>` via `mergeRegistry`. No core change. |
| **Props** | `lanes: { id, label, cite, state, field?, fromWidget?, absentFromWidget?, note? }[]` (**required**) · `units?` `"mi"\|"km"` (default `"mi"`) · `breakOnMultipart?` (default `true`) · `showProvenance?` (default `true`) · `asOf?` · `axisLabel?` · `cursorMi?` |
| **Lane `state`** | `"evidenced"` · `"evidenced-absent"` · `"never-evaluated"` · `"operator-only"` · `"unverifiable"` · `"tri"` (paints all three seismic states from `fromWidget` + `absentFromWidget`) |
| **Emits** | **`rangeSelect`** `{layerId, field:"station_mi", min, max}` (ruler brush) · **`featureSelect`** `{layerId, oids}` (band click) · **`hover`** `{layerId, oids}`. All shipped trigger types; `source` is its own `id` |
| **Honors** | `filter` (recomputes lane coverage within the incoming `where`) · `flash` (pulses the matching band) · `clear` (drops the cursor and the selection) |
| **dataSource** | `{ fromWidget }` or `{ sourceId }` for the primary lane; each lane resolves its own records through the **injected `outputs` registry** by `lanes[].fromWidget` — the same mechanism `dataSource.fromWidget` uses, never a fetch around a source |
| **Visual** | `--strata-*` tokens only. Solid = `--strata-success`/`--strata-danger`, hatch = `--strata-warning`, hollow = `--strata-secondary`, empty = dotted `--strata-border`. Pattern is the primary channel, hue the reinforcement |
| **RTL** | the axis mirrors under `direction: rtl` and the ruler numbers run right-to-left — a real requirement for a Gulf deployment (§2.5 i18n note) |
| **Interaction** | a band is a **toggle**: clicking the selected band clears the cursor, the dossier and the flash |
| **Refuses** | to draw a continuous axis across a multipart gap. With `breakOnMultipart: true` it renders one sub-axis per part with an explicit labelled break; with `false` it renders nothing and shows why |

### 2.7 `connections` (19 — every wire uses a shipped emitter)

| # | from | trigger | to | action | options | User-visible behaviour |
|---|---|---|---|---|---|---|
| 1 | `lines` | `rowSelect` | `map` | `zoomTo` | `{layerId:"pipes"}` | pick a line — the map frames it |
| 2 | `lines` | `rowSelect` | `map` | `flash` | `{layerId:"pipes"}` | and it pulses |
| 3 | `lines` | `rowSelect` | `strip` | `filter` | | the strip rebuilds for that line |
| 4 | `lines` | `rowSelect` | `dossier` | `viewInTable` | | the line's own record fills the dossier |
| 5 | `lines` | `rowSelect` | — | `setUrlParam` | `{param:"line"}` | the line is deep-linkable |
| 6 | `strip` | `featureSelect` | `map` | `zoomTo` | `{layerId:"bands"}` | click a band — the map flies to that milepost |
| 7 | `strip` | `featureSelect` | `map` | `flash` | `{layerId:"bands"}` | the band pulses on the centerline |
| 8 | `strip` | `featureSelect` | `dossier` | `viewInTable` | | the crossing's dossier opens |
| 9 | `strip` | `rangeSelect` | `pack` | `filter` | | brush the ruler — the pack narrows to that reach |
| 10 | `strip` | `rangeSelect` | `prov` | `filter` | | so does the provenance table |
| 11 | `strip` | `rangeSelect` | — | `setUrlParam` | `{param:"mp"}` | a milepost range is shareable |
| 12 | `strip` | `hover` | `map` | `flash` | `{layerId:"bands"}` | hovering a band lights the map |
| 13 | `flt` | `filterChange` | `map` | `filter` | `{layerId:"pipes"}` | operator/commodity filter drives the map |
| 14 | `flt` | `filterChange` | `lines` | `filter` | | and the lines list |
| 15 | `lines` | `rowSelect` | `prov` | `filter` | | the provenance table rebuilds for the selected line |
| 16 | `map` | `featureSelect` | `dossier` | `viewInTable` | | click the map — same dossier |
| 17 | `map` | `extentChange` | `kpi-ap` | `showStatistics` | | pan/zoom on page 2 — fault-zone miles recompute for the view |
| 18 | `byop` | `categorySelect` | `map2` | `filter` | `{layerId:"pipes"}` | click *Phillips 66* — the network map shows only its lines |
| 19 | `byop` | `categorySelect` | `netTable` | `filter` | | the table follows |

**Plus eight widgets that link with *zero* connections** — `lines`, `netTable`, `sizes` and `byop` via
`dataSource.layerId`/`sourceId: "pipes"`; `gauge-unev`, `kpi-ap`, `dossier` and the `strip` itself via
`dataSource.fromWidget`. The line search needs no wire either: `lines` binds
`{ layerId: "pipes", fromWidget: "q-line" }`, so typing an ID or an operator narrows the list directly.
Filter changes reach all of them through the store, which is why the KPI strip has no wires of its own.

The spine:

```jsonc
{ "from": "lines", "trigger": "rowSelect",      "to": "map",     "action": "zoomTo", "options": { "layerId": "pipes" } },
{ "from": "lines", "trigger": "rowSelect",      "to": "strip",   "action": "filter" },
{ "from": "strip", "trigger": "featureSelect",  "to": "map",     "action": "zoomTo", "options": { "layerId": "bands" } },
{ "from": "strip", "trigger": "featureSelect",  "to": "dossier", "action": "viewInTable" },
{ "from": "strip", "trigger": "rangeSelect",    "to": "pack",    "action": "filter" },
{ "from": "strip", "trigger": "rangeSelect",    "to": "prov",    "action": "filter" },
{ "from": "flt",   "trigger": "filterChange",   "to": "map",     "action": "filter", "options": { "layerId": "pipes" } },
{ "from": "byop",  "trigger": "categorySelect", "to": "map2",    "action": "filter", "options": { "layerId": "pipes" } }
```

---

## 3. Data sources

**Full inventory, field lists, counts, distributions and eleven traps:
`pipeline-catalog-ca.json` beside this file**, and
`../../data_sources/data_sources_ca.md` § *Energy — CA hazardous liquid pipelines and the seismic
evidence gap*. Everything below was fetched live **2026-08-06**. All EPSG:4326 on ingest. **CORS is open
on all three servers** — browser-direct, no proxy.

### 3.1 The headline — the catalogue named the pipeline layer but never characterised it

`data_sources_ca.md` lists `CA_Oil_Pipeline_2018/FeatureServer` as one bare URL in the Cal OES hosted-org
dump, with no role, no schema and no [ENERGY] section anywhere in the solution data map. It is in fact
**the** primary asset layer for this entire vertical: 707 features, 6,172 reported miles, the operator,
the commodity and the diameter of every regulated intrastate hazardous-liquid pipeline in California.

| Catalogue said | Verified to exist |
|---|---|
| one unannotated URL | **707** polylines · 7 fields · `objectIdField` **`FID`** · 3857 → request `outSR=4326` |
| no [ENERGY] role table | commodity, operator and mileage distributions (§1.3, catalog JSON) |
| seismic layers listed only for *banking collateral* and *emergency response* | the **Unevaluated Areas** sublayer — **2,806** polygons, the single most important layer in this recipe, not mentioned anywhere |
| (not known) | **no natural-gas centerline exists** in the catalogue at all — Part 192 is undemonstrable on CA public data |

`data_sources_ca.md` has been amended accordingly (§7.7).

### 3.2 Role × region

*(Only California was swept. The AB 864 requirement is distinctively Californian and the Seismic Hazards
Mapping Act zones are a California instrument; a second region needs its own statute and its own hazard
programme, not just its own data.)*

| Role | California (verified 2026-08-06) | National | Client-supplied |
|---|---|---|---|
| **Centerline + operator + commodity** ★★ | `services.arcgis.com/BLN4oKB0N1YSgvY8/…/CA_Oil_Pipeline_2018/FeatureServer/0` (**707**) | NPMS — restricted | operator LRS / PODS extract |
| **Surface fault rupture (regulatory)** ★★ | `services.gis.ca.gov/…/GeoscientificInformation/Fault_Lines/MapServer/`**`4`** (**547** AP zones) | — | — |
| **Liquefaction (regulatory)** ★★ | `…/Liquefaction/MapServer/`**`0`** (**738**) | — | — |
| **Never evaluated** ★★★ | `…/Liquefaction/MapServer/`**`4`** (**2,806**; ⚠️ `MAPNAME_1` is whitespace on every row — locatable, not nameable) | — | — |
| Quaternary fault crossings | `gis.conservation.ca.gov/…/CGS/FAM_QFaults/FeatureServer/0` (**38,646**) | USGS QFaults | — |
| Ground motion at a station | `…/CGS/MS48_GroundMotion_PGA_10pc50/ImageServer` (identify → g) | USGS NSHM | — |
| Landslide susceptibility | `…/CGS/MS58_LandslideSusceptibility_Classes/MapServer` — **raster, identify only** (trap P6) | — | — |
| Line endpoints / terminals | `…/CA_OilRefineriesandTerminals/FeatureServer/0` (**80**) | — | — |
| **HCA segments** | ⛔ withheld — NPMS operator/government only | ⛔ | required CSV |
| **ILI · digs · CP · MAOP · material · AB 864** | ⛔ not public | ⛔ | required CSV |
| **Incident history** | ⛔ | ⛔ **non-tabular** on data.transportation.gov (§1.4) | optional CSV |
| **Gas transmission centerline** | ⛔ **does not exist** in the CA catalogue | restricted | — |
| Basemap | keyless OSM · CARTO Positron/Voyager/Dark · OpenTopoMap | | |

### 3.3 The demo region and the demo line

**California, statewide** for page 2. Page 1 opens on **PLINEID 0274** — Kinder Morgan, Refined Product,
`PIPE_SIZE` `'10'`, **one part**, 1,352 vertices, **91.05 measured miles against 91.21 reported (0.18 %)**.
It was chosen because it is *honest*: single-part, so the milepost axis is real; and it still exhibits
four of the six evidence states — two AP fault zones (Vine Hill MP 0.00–0.33, Walnut Creek MP
0.33–0.39), three Quaternary fault crossings (MP 6.88, 20.72, 28.72), no mapped liquefaction zone, and
**every one of its 91.05 miles inside an area the state has never evaluated** (13 abutting polygons,
none of which the layer can name).

The lines list opens with **PLINEID 1341** (El Paso Natural Gas, 5 parts, 217.6 mi, **108.1-mile
inter-part gap**, 100 % unevaluated) pinned second and badged ⚠ — it is the discontinuity exemplar, and
the app must be seen refusing to station it.

---

## 4. Verify each URL first (terminal)

```bash
# ── 1. THE CENTERLINE. Cal OES hosted org. CORS "*" — browser-direct, no proxy.
P=https://services.arcgis.com/BLN4oKB0N1YSgvY8/arcgis/rest/services/CA_Oil_Pipeline_2018/FeatureServer/0
curl -s "$P?f=json" | head -c 400        # CA_Pipelines_2018 · polyline · SR 3857 · maxRecordCount 2000
curl -s "$P/query?where=1=1&returnCountOnly=true&f=json"                       # -> {"count":707}

# ── 2. TRAP P1 — THE OID IS `FID`, AND A DIFFERENT COLUMN IS CALLED `OBJECTID`.
curl -s "$P/query?where=1=1&outFields=FID,OBJECTID,PLINEID&returnGeometry=false&resultRecordCount=5&f=json"
#  -> objectIdFieldName: "FID";  FID 4 -> OBJECTID 7.   They diverge.
#     The house rule "OBJECTID is always the OID" is FALSE here. Highlight/select on FID or you will
#     light up the wrong pipeline.

# ── 3. TRAP P2 — Shape__Length IS WEB-MERCATOR METRES, NOT GROUND METRES.
curl -s -G "$P/query" --data-urlencode "where=1=1" --data-urlencode "f=json" \
  --data-urlencode 'outStatistics=[{"statisticType":"sum","onStatisticField":"Shape__Length","outStatisticFieldName":"m"},{"statisticType":"sum","onStatisticField":"TOTAL_MILE","outStatisticFieldName":"mi"}]'
#  -> m = 12,042,671  (= 7,483 mi)   mi = 6,172.48
#     A 21% "discrepancy" that is really 1/cos(latitude). True geodesic length is 6,093.1 mi.
#     NEVER report mileage from Shape__Length on a 102100 layer.

# ── 4. TRAP P4 — THE DIAMETER FIELD IS A STRING WITH 84 FORMS.
curl -s -G "$P/query" --data-urlencode "where=1=1" --data-urlencode "f=json" \
  --data-urlencode 'outStatistics=[{"statisticType":"count","onStatisticField":"FID","outStatisticFieldName":"n"}]' \
  --data-urlencode "groupByFieldsForStatistics=PIPE_SIZE" --data-urlencode "orderByFields=n DESC"
#  -> '8'(132) '6'(100) '12'(84) '10'(68) '16'(45) '4'(39) '8, 10'(24) '6, 8'(16) … ' '(6) '8.625'(5) '8"'(4)
#     84 distinct. Not a single plain number on 169 lines = 1,723.1 mi = 28.3% of the network.

# ── 5. THE REGULATORY HAZARD LAYERS. State Geoportal, CGS source. CORS reflects Origin.
G=https://services.gis.ca.gov/arcgis/rest/services/GeoscientificInformation
curl -s "$G/Fault_Lines/MapServer/4/query?where=1=1&returnCountOnly=true&f=json"    # ->   547  AP zones
curl -s "$G/Liquefaction/MapServer/0/query?where=1=1&returnCountOnly=true&f=json"   # ->   738  zoned
curl -s "$G/Liquefaction/MapServer/4/query?where=1=1&returnCountOnly=true&f=json"   # -> 2,806  UNEVALUATED
curl -s -G "$G/Liquefaction/MapServer/4/query" --data-urlencode "returnCountOnly=true" \
     --data-urlencode "where=MAPNAME_1 IS NOT NULL AND MAPNAME_1 <> ''" --data-urlencode "f=json"
#  -> {"count":0}   of 2,806  (trap P10)
#  ⚠ MAPNAME_1 LOOKS like the 7.5' quadrangle name and is whitespace on every row. So is LONGITUDE1
#    and LATITUDE_S. An unevaluated reach is locatable, never nameable.
#  ⚠ All three report objectIdField NULL though an OID field exists. Bind read-only (trap P5).

# ── 6. THE SPATIAL QUESTION — the mechanism the whole app rests on.
#     Fetch ONE line at FULL resolution, then POST its geometry to each authority.
curl -s -G "$P/query" --data-urlencode "where=PLINEID='0274'" --data-urlencode "outSR=4326" \
     --data-urlencode "f=json" -o /tmp/l.json
python - <<'PY'
import json,urllib.parse,urllib.request
g=json.load(open('/tmp/l.json'))['features'][0]['geometry']
geom=json.dumps({"paths":g["paths"],"spatialReference":{"wkid":4326}})
for name,url in [
 ("AP Earthquake Fault Zone","https://services.gis.ca.gov/arcgis/rest/services/GeoscientificInformation/Fault_Lines/MapServer/4/query"),
 ("Liquefaction Zone",       "https://services.gis.ca.gov/arcgis/rest/services/GeoscientificInformation/Liquefaction/MapServer/0/query"),
 ("Unevaluated Areas",       "https://services.gis.ca.gov/arcgis/rest/services/GeoscientificInformation/Liquefaction/MapServer/4/query"),
 ("Quaternary fault traces", "https://gis.conservation.ca.gov/server/rest/services/CGS/FAM_QFaults/FeatureServer/0/query")]:
    d={"geometry":geom,"geometryType":"esriGeometryPolyline","spatialRel":"esriSpatialRelIntersects",
       "inSR":"4326","where":"1=1","returnCountOnly":"true","f":"json"}
    r=urllib.request.Request(url,data=urllib.parse.urlencode(d).encode())
    print(f"{name:26s}", json.load(urllib.request.urlopen(r,timeout=120)))
PY
#  -> AP Earthquake Fault Zone   {'count': 2}    Vine Hill, Walnut Creek
#     Liquefaction Zone          {'count': 0}    <-- NOT an all-clear. See the next line.
#     Unevaluated Areas          {'count': 13}   <-- 13 abutting polygons covering the WHOLE line
#     Quaternary fault traces    {'count': 3}
#     Clipped geodesically (shapely, make_valid): AP 0.32 mi · liquefaction 0.00 mi ·
#     unevaluated 91.05 mi = 100.0% of the line. The app reproduces all three exactly.
#  ⚠ A 1,352-vertex polyline exceeds the shell's ARG_MAX. POST as a form body, never in a query string.

# ── 7. THE THREE-WAY PARTITION — the headline. Geodesic, all 707 lines, make_valid before intersect.
#     (full script in pipeline-catalog-ca.json; result reproduced here)
#     network                 6,093.1 mi   (operators report 6,172.48 — +1.30%)
#       in a liquefaction zone   1,086.3 mi  17.8%   335 lines
#       evaluated, not zoned     1,223.8 mi  20.1%
#       NEVER EVALUATED          3,783.0 mi  62.1%   319 lines
#       zoned x unevaluated overlap  0.0 mi          <-- clean partition, sums to 100.0%
#       in an AP fault zone         61.8 mi   1.01%   88 lines

# ── 8. TRAP P3 — A THIRD OF THE NETWORK HAS NO CONTINUOUS AXIS.
#     145 of 707 lines are multipart (2,019.1 mi = 33.1%); max 80 parts; worst inter-part gap 108.1 mi.
curl -s -G "$P/query" --data-urlencode "where=PLINEID='1341'" --data-urlencode "outSR=4326" \
     --data-urlencode "f=geojson" | python -c "
import sys,json; g=json.load(sys.stdin)['features'][0]['geometry']
print('type',g['type'],'| parts',len(g['coordinates']))"
#  -> type MultiLineString | parts 5      El Paso Natural Gas, 217.6 mi, 108.1 mi gap between parts.
#     DO NOT concatenate parts into one milepost axis. Break the ruler and label the gap.

# ── 9. TRAP P7 — GENERALIZE FOR DRAWING, NEVER FOR MEASURING.
curl -s "$P/query?where=1=1&outFields=PLINEID&outSR=4326&f=geojson" | wc -c                       # 9.04 MB
curl -s "$P/query?where=1=1&outFields=PLINEID&outSR=4326&f=geojson&maxAllowableOffset=0.0005" | wc -c  # 0.38 MB
#     24x smaller and correct for the map. But 0.0005 deg is ~55 m — enough to move a crossing in or
#     out of a zone. The selected line is ALWAYS re-fetched at full resolution before stationing.

# ── 10. TRAP P6 — LANDSLIDE SUSCEPTIBILITY IS A RASTER.
C=https://gis.conservation.ca.gov/server/rest/services/CGS
curl -s "$C/MS58_LandslideSusceptibility_Classes/MapServer/0/query?where=1=1&returnCountOnly=true&f=json"
#  -> {"error":{"code":400,"message":"Invalid or missing input parameters."}}   no fields, no OID.
#     Only /identify works, returning a pixel class. You cannot compute landslide MILES — only a
#     class at a station. API RP 1187 Level-1 screening is a per-station read here.

# ── 11. THE ONE CONTINUOUS LANE — ground motion at a station.
curl -s -G "$C/MS48_GroundMotion_PGA_10pc50/ImageServer/identify" \
  --data-urlencode 'geometry={"x":-122.02,"y":37.95,"spatialReference":{"wkid":4326}}' \
  --data-urlencode "geometryType=esriGeometryPoint" --data-urlencode "f=json"
#  -> {"value":"0.556931", …}     0.56 g PGA, 10% in 50 yr, on line 0274 near Walnut Creek.

# ── 12. THE EVIDENCE HORIZON. Confirm the gaps BEFORE designing around them.
curl -s "https://data.transportation.gov/resource/qdme-9bbm.json?\$limit=1"
#  -> {"error":true,"message":"no row or column access to non-tabular tables"}
#     PHMSA incident data is a DOWNLOAD, not a service. Every pipeline-incident dataset is type "href".
curl -s -o /dev/null -w "%{http_code}\n" "https://osfm.fire.ca.gov/what-we-do/pipeline-safety-and-cupa"
#  -> 403     OSFM's estate refuses non-browser clients. Do not design against a scrape of it.

# ── 13. CORS on all three servers — no proxy anywhere in this app.
for U in "https://services.arcgis.com/BLN4oKB0N1YSgvY8/arcgis/rest/services/CA_Oil_Pipeline_2018/FeatureServer/0" \
         "https://services.gis.ca.gov/arcgis/rest/services/GeoscientificInformation/Fault_Lines/MapServer/4" \
         "https://gis.conservation.ca.gov/server/rest/services/CGS/FAM_QFaults/FeatureServer/0"; do
  curl -sD- -o /dev/null -H "Origin: https://example.com" "$U?f=json" | grep -i access-control-allow-origin
done
#  -> *  ·  https://example.com  ·  https://example.com

# ── 14. THE EXTENT for initialState.viewpoint.
curl -s -G "$P/query" --data-urlencode "where=1=1" --data-urlencode "returnExtentOnly=true" \
     --data-urlencode "outSR=4326" --data-urlencode "f=json"
```

---

## Guided wizard — **the prompts that assign the app's defaults**

Launch with `/recipe energy_pipeline-integrity-evidence-pack`. Ask each group, **apply the default so
"accept all" builds a complete app**, confirm a one-line summary, then run §5.

| # | Wizard question | Options → **default** | The default it assigns |
|---|---|---|---|
| 1 · App | Title & extract date? | free text → **"Pipeline Integrity Evidence Pack — the alignment strip"**, today | header title + the vintage line stamped on every lane and export |
| 2 · Regime | Which regulation frames the pack? | **49 CFR Part 195 (hazardous liquid) + AB 864** · Part 192 (gas — *no CA public centerline exists*, §1.8) | the 14 pack rows and their citations |
| 3 · Line | Which line opens? | **PLINEID 0274** (single-part, 91.05 mi) · pick from 707 · none | page-1 selection + `initialState.viewpoint` |
| 4 · Axis | What do we do with the 145 multipart lines? | **break the ruler and label the gap** · refuse to station them at all | `alignment-strip.breakOnMultipart` |
| 5 · Units | Miles or kilometres? | **miles** · km | the ruler, every station, every export |
| 6 · Seismic lanes | Which authorities does the strip query? | **AP fault zones + liquefaction + unevaluated + Quaternary traces** · add PGA profile · minimum (AP only) | the `lanes` array + the `analysis` runs |
| 7 · Ignorance | Show the "never evaluated" state? | **yes — it is 62.1 % of the network** · no | the hatched lane and the inverted gauge |
| 8 · Your records | Upload operator CSVs now? | **skip — show the empty lanes with citations** · material · ILI/digs · CP · HCA | `FileDataSource`s behind the accordion |
| 9 · Network page | Include the statewide roll-up? | **yes** · no | page `network` + the partition bar |
| 10 · Look | Light or dark? | **light** (it is an alignment sheet; prints anyway) · dark | `ThemeSpec.mode` + `basemapForTheme` |
| 11 · Export | What does the print button produce? | **the Evidence Pack** — line, 14 rows, every band, every provenance chip, the axis caveat · summary only | `print` exhibit + `table` CSV |
| 12 · Locale | English only, or EN + AR/RTL? | **EN only** (CA statutes are English) · EN + AR | `i18n`; the strip mirrors under RTL |

---

## 5. Prompt-script (run in order)

```
A. /new-app — "Pipeline Integrity Evidence Pack — the alignment strip", Template: open-design.
   Structured ThemeSpec (§2.2), LIGHT, primary #1f4e5f. App-shell: header (title · vintage line ·
   operator/commodity filter · page-nav · theme-switch), footer (attribution · the stationing caveat ·
   share · print), splash titled "This is a working paper, not a filing".
   Two pages: `pack`, `network`. Register the one app-local widget (§2.6) with its fallback behind a flag.

B. /add-data — the verified spine, all outSR=4326:
     pipes     CA_Oil_Pipeline_2018/FeatureServer/0                    (707)  oidField FID, NOT OBJECTID
     apzones   GeoscientificInformation/Fault_Lines/MapServer/4        (547)
     liq       GeoscientificInformation/Liquefaction/MapServer/0       (738)
     unev      GeoscientificInformation/Liquefaction/MapServer/4     (2,806)  <-- the headline layer
     qfaults   CGS/FAM_QFaults/FeatureServer/0                     (38,646)
     terminals CA_OilRefineriesandTerminals/FeatureServer/0             (80)
   Draw `pipes` with maxAllowableOffset=0.0005 (9.04 MB -> 0.38 MB). Re-fetch the SELECTED line at
   full resolution before any measurement (trap P7).

C. /symbology + /popup — genuine ESRI drawingInfo/popupInfo on verified fields only.
     pipes     uniqueValue on COMMODITY; 2 px; selected line 4 px in --strata-primary.
     apzones   simple fill, danger @ 40/255, outline two tones darker.
     liq       simple fill, danger @ 30/255.   unev  HATCHED fill, warning @ 30/255.
     qfaults   simple line, 1 px, secondary.
     popupInfo pipes: PLINEID, OPERATOR, COMMODITY, PIPE_SIZE (labelled "as filed"), TOTAL_MILE
               (labelled "reported"), plus the measured length. NEVER show FID or OBJECTID.
     esriSMSCircle/esriSMSSquare only — never esriSMSPath.

D. /analyze — the crossings. One `analysis` (clip) run per authority against the SELECTED line:
     xsect-ap · xsect-liq · xsect-unev · xsect-qf.  Each publishes bands as an output carrying
     {from_mi, to_mi, band_mi, label, authority, service, query, asOf, rows, ms}.
   make_valid every hazard polygon before intersecting — server-generalized polygons self-intersect and
   GEOS/Turf will throw "side location conflict" (trap P8).
   Add the union of bands to the map as the `bands` GeoJSON layer.

E. /panel — the alignment strip (§2.6) with the 14 lanes of §1.4 as its `lanes` prop, each carrying its
   citation and evidence state. breakOnMultipart: true. Bind dataSource.fromWidget = "xsect-ap".
   Stamp the extract date and the axis caveat on the ruler.

F. /panel table — the pack (14 static rows) and the provenance table (lane · authority · service ·
   query · asOf · rows · ms). Then the "bring your records" accordion with four add-data uploads.

G. /panel statistics + /panel chart + /panel carto — page 2:
     stacked-bar: 1,086.3 / 1,223.8 / 3,783.0 miles (it must sum to 6,093.1).
     chart: never-evaluated miles by operator, limit 12.
     carto: category on PIPE_SIZE — show the register's own dirt.

H. WIF: author AppLayout.connections — the 19 rows of §2.7. Shipped emitters ONLY;
   no window, no buttonClick, no navigate. Verify the signature loop end to end.

I. Controls + composed export: scale, legend, basemapSwitcher, layerList; status-bar in EPSG:4326;
   print = the Evidence Pack (line · 14 rows · every band · every provenance chip · the axis caveat);
   share deep-link carrying ?line= and ?mp=.
```

---

## 6. Verify

| Check | Pass |
|---|---|
| Silhouette is `alignment-strip`; distinct from every sibling and every researched open-design | ✅ §2.1 — no other design navigates by a linear-referenced distance axis; aviation's is a *vertical* feet-MSL ladder |
| ≥3 `connections` fire; the signature loop works end to end | ✅ 19 authored, all on shipped emitters; +8 widgets link via `sourceId`/`fromWidget` |
| No wire uses a pending emitter | ✅ `buttonClick`/`mapClick`/`sketchComplete`/`viewChange`/`pageChange`/`timer` unwired; **no `window`** as a result |
| Every `layerId` + field verified against the live service (§4) | ✅ 707 · 547 · 738 · 2,806 · 38,646 · 80; field lists in `pipeline-catalog-ca.json` |
| Selection uses `FID`, never `OBJECTID` | ✅ trap P1 — the layer carries both and they diverge |
| No mileage is ever computed from `Shape__Length` | ✅ trap P2 — geodesic only; the Mercator figure (7,483 mi) appears once, labelled as the rejected method |
| Diameter is never parsed to a number for analysis | ✅ trap P4 — shown verbatim, labelled *"as filed"*; the 28.3 % figure is stated |
| **No continuous axis is drawn across a multipart gap** | ✅ trap P3 — the ruler breaks, the gap is labelled in miles, and line 1341 (108.1 mi) is pinned as the exemplar |
| Measurement never runs on a generalized geometry | ✅ trap P7 — the selected line is re-fetched at full resolution |
| Hazard polygons are `make_valid`-ed before intersection | ✅ trap P8 — verified to throw without it |
| The three-way partition is a true partition | ✅ 17.8 + 20.1 + 62.1 = 100.0 %, overlap exactly 0.0 mi (§4 step 7) |
| "Not in a liquefaction zone" is never rendered as an all-clear | ✅ the hatched state, the inverted gauge, the legend wording, and the pack row |
| The six evidence states are not encoded by hue alone | ✅ §2.2 — solid / hollow / 45° hatch / dotted-empty in both the strip and the legend |
| No risk score, rank or weighting appears anywhere | ✅ §2.1 rejected rival; `weighted-overlay` deliberately unused |
| No HCA is drawn, inferred or approximated | ✅ §1.4 line 7 — pack row states the policy |
| Every evidenced band carries a provenance chip | ✅ service · query · UTC · rows · ms, in the strip and in the export |
| The app never claims to be a filing or the operator's LRS | ✅ splash + header vintage + footer + ruler label + every export |
| `responsive.small` collapses every side-by-side row | ✅ both splitters → column; lines → `dock:"top"`; **the strip stays horizontal and scrolls**; 240 px map floor |
| Basemap keyless; everything EPSG:4326 | ✅ CARTO/OSM only; `outSR=4326` on every request |
| No proxy is required | ✅ CORS verified open on all three servers (§4 step 13) |
| Runs on Strata **and** ArcGIS | ✅ read-only throughout; the CSV path writes to a client-side `FileDataSource` only |
| Gas (Part 192) is discussed but never rendered | ✅ §1.8 — no CA public gas centerline exists |

---

## 7. Harvest (gaps → strata-core)

1. **No shipped widget renders a linear-referenced strip.** Built here as the one §10.3 widget. The
   pattern generalises to every linear asset — transmission corridors, rail, highway pavement, fibre
   routes, canals — which is five sectors' worth of reuse. *Harvest candidate.*
2. **`chart` has no `referenceLine` / threshold prop.** *(Fourth request across researched recipes —
   also `industry_mining-and-concession-compliance` §7.1, `education_campus-operations` §7.4. Overdue.)*
3. **`table` has no `orderBy` / `sort` prop**, so "longest unevaluated stretch first" needs a filter or a
   chart. *(Third request.)*
4. **`button` does not emit `buttonClick`**, so a `window` cannot be opened declaratively. This design
   dropped its statute modal and used an `accordion`. *(Fourth request across researched recipes.)*
5. **`@strata/processing` has no linear-referencing helper.** `lineIntersect` + cumulative geodesic
   length + `along`/`nearestPointOnLine` were written by hand here. A `station(line, point)` /
   `bands(line, polygons)` pair would be a small, high-value addition — and would carry the multipart
   refusal as a first-class error rather than leaving each app to rediscover it.
6. **`MapServer` layers with `objectIdField: null`** (all three State Geoportal hazard layers, and
   `MOLLeadAgency` before them) still have no documented handling in the manifest. **Second recipe to hit
   it** — §5 *Rules that bite* should say: read the layer JSON, and if `objectIdField` is absent, bind
   read-only and do not attempt OID-keyed selection.
7. **The `OBJECTID`-is-always-the-OID rule is wrong often enough to need a caveat.** This layer's OID is
   `FID` *and* it carries a decoy `OBJECTID`. `CLAUDE.md` and the manifest both state the rule
   unconditionally; they should say "read `objectIdFieldName` from the service, and never assume".
8. **Catalogue corrections** — proposed text applied to `../../data_sources/data_sources_ca.md`: a new
   **§ Energy — [ENR]** role table, the `CA_Oil_Pipeline_2018` schema and its four traps, and the
   **Unevaluated Areas** sublayer, which no recipe had noticed and which is the largest single
   evidence-gap layer in the California catalogue.
10. **The guide does not specify the house *feel* for standalone builds, and four apps prove it.**
   `app-design.md` fixes principles and `COMPONENT-MANIFEST.md` §6 fixes the `--strata-*` token
   vocabulary — both enforceable, and this app was violating the second until it was diffed against
   its siblings. But neither fixes **base font size, the component type scale, chrome dimensions or
   corner radii** for a standalone HTML build, and the manifest's type scale
   (`--strata-h1…-body2`, `fonts.scale`) lives only on the `<StrataApp>` path that every solution app
   bypasses. Result: mining 15 distinct font sizes, catchment 13, campus-operations 8, this app 11 —
   two different font stacks, two base sizes (13 px vs 14 px), four sets of corner radii, all
   "compliant". **Proposal:** a *Standalone app chrome* section in `app-design.md` fixing base
   14 px/1.45, the 10.5–12.5 working band, the 8 px control radius and the 270 px/10 px drawer, plus a
   shared `strata-app.css` the standalone builds link — so the house feel stops depending on which app
   was written last.

9. **Harvest candidates:** **`alignment-strip`** (navigation by a linear-referenced axis) and the
   **six-state evidence vocabulary** (*evidenced · evidenced-hazard · evidenced-absent · never-evaluated ·
   unverifiable · operator-only*),
   which is reusable by any recipe whose product is a defensible record rather than a value. Promote per
   the `APP-TEMPLATE-LIBRARY.md` harvest rule if reused twice.

---

## 7a. Presentation & article

| File | |
|---|---|
| `presentation/index.html` | 10-slide tabaqat deck, 16:9, keyboard + click nav, prints one slide per page. Opens on *"Two-thirds of California's regulated liquid pipelines run through ground nobody has mapped."* The arc: the partition → **the framing** → the one-dimensional instrument → the evidence-state vocabulary → three of fourteen → four traps → the refusal → the bug → honest scope |
| `presentation/linkedin-article.md` | ~1,250-word article, a ~180-word teaser, and a **30-row claims note** tracing every figure to the query or document it came from — plus an explicit **"Do not claim"** list and six gaps to volunteer if asked |
| `presentation/verify-figures.mjs` | **76 checks, green.** Re-queries every published figure against the live services *and* enforces the framing rules below |

Both were verified against the **live services**, not against this file: every figure re-queried and
matched, and the framing checked mechanically rather than by eye.

**The framing is load-bearing, and it is the *inverse* of the mining recipe's.** There, a company was
absent from a published list for reasons only it knew, so the rule was *never name an operator*. Here
the gap belongs to **the State of California's own mapping programme** — no operator has any say in
whether CGS has evaluated a quadrangle. So *"86.8 % of ⟨operator⟩'s mileage has never been evaluated"*
would read as negligence and would misattribute a public-sector data gap to a private company.

Three rules, all machine-enforced by `verify-figures.mjs` §4:

1. **No operator is named** anywhere in the deck or the article (16 names checked). The per-operator
   view stays inside the app, where the reader can see it is a map of the *state's* coverage.
2. **An absence of evaluation is never rendered as a hazard finding** — no *unsafe*, *at risk*,
   *in violation*, *non-compliant*, *negligent*, *likely to fail*.
3. **Both artifacts carry the caveats**: the vintage, the stationing caveat, the naming of the
   programme that did not evaluate, the governing clause, and an explicit disclaimer that this is
   **not a risk score** — with the reason it cannot be one.

⚠ Writing that guard reproduced this recipe's recurring trap for the **third** time: the
forbidden-word scan matched the *Do-not-claim list itself*, and the "risk score" ban would have banned
the disclaimer. Both were fixed by narrowing the **target** (scan published prose; strip the policy
sections) and by inverting the weak one (require the disclaimer rather than ban the phrase) — never by
weakening the guard. The earlier two were a grep for `proxy` hitting the server's own comment, and a
search for the danger hex hitting the hazard pattern's own swatch.

---

## 7a-i. Open questions & risks

| # | Risk | Mitigation shipped |
|---|---|---|
| **R1** | **The centerline is 2018 and was last edited 2021-02-05.** Lines have been sold, abandoned and re-routed since | the vintage is in the header, the footer, the splash and every export; the pack's own row 3 states it; nothing time-sensitive is computed from it |
| **R2** | **A user reads the hatched lane as "low risk".** Amber is conventionally "medium", and here it means *unknown* | the state is spelled out in words in the legend, the gauge is inverted and labelled *"% never evaluated"*, and the pack row reads *"no authority has evaluated this ground"* |
| **R3** | **Derived stationing is mistaken for the operator's LRS.** Direction, origin and part order are all the publisher's, not the operator's | the caveat is printed on the ruler itself, in the footer and on every export; the wizard cannot turn it off |
| **R4** | **A multipart line is silently concatenated** by a future contributor, teleporting the axis up to 108 mi | `breakOnMultipart` defaults true; the widget contract says it *refuses*; line 1341 is pinned in the demo list so the refusal is always on screen |
| **R5** | **`PIPE_SIZE` gets parsed** because `'8'` parses on 132 lines and the failure is silent on 169 | never parsed for analysis; rendered verbatim and labelled *"as filed"*; the 28.3 % figure is a pack row, not a footnote |
| **R6** | **Someone adds a risk score** because every competitor has one | §2.1 records the rejection with its evidence; `weighted-overlay` is listed as deliberately unused for exactly this reason |
| **R7** | **The hazard services change sublayer numbering.** `Liquefaction/4` being *Unevaluated Areas* is not self-documenting | sublayer ids and names are config, asserted against the live `?f=json` on build; a mismatch fails loudly rather than drawing the wrong lane |
| **R8** | **CGS republishes the Seismic Hazard Zones** and the 62.1 % moves | that is the intended growth path, not a rewrite: the partition is recomputed from live services, and a *falling* number is the app reporting good news |
| **Q1** | Should the app ingest a **PODS or UPDM extract** directly rather than four CSVs? It is what an operator already has, and it would fill nine lanes at once. Needs a client to define which PODS version and which tables — deferred, with the CSV shape published as the interim contract. |
| **Q2** | Is an **elevation profile** worth adding as a lane? It is the alignment sheet's other classic track and `elevation` ships. It needs a verified DEM sampling endpoint we do not yet have — and a fabricated profile on an engineering instrument is worse than no profile. Held. |
| **Q3** | **Sibling states?** The strip and the evidence-state vocabulary are jurisdiction-neutral, but the *content* is not: the Seismic Hazards Mapping Act's unevaluated-areas layer is a California artifact, and the AB 864 lane is Californian statute. Another state needs its own hazard programme, not just its own pipelines. |
| **Q4** | **Gas.** Part 192, §192.607 and the §192.624 MAOP clock (**50 % by 3 July 2028**) are the larger commercial opportunity, and the pack structure transfers unchanged. But no CA public gas centerline exists (§1.8), so a gas demo needs operator data. Flagged as the first paid-engagement extension. |

---

## 8. Sources

**Internal** — `../APP-TEMPLATE-LIBRARY.md` · `../DESIGN-CONTEXT.md` · `../DESIGN-REQUEST-PROMPT.md` ·
`strata/recipes/COMPONENT-MANIFEST.md` (§3 registry, §4 triggers/actions, §10 freestyle charter) ·
`strata/docs/guide/app-design.md` · **`pipeline-catalog-ca.json`** (this folder — the data evidence) ·
**`DESIGN-PROPOSAL.md`** (this folder — the design argument) ·
**`../../data_sources/data_sources_ca.md`**.

**Regulation & statute** (retrieved 2026-08-06)
- **49 CFR §195.452** — pipeline integrity management in HCAs, incl. **(g)** data integration,
  **(h)** repair conditions, **(j)(3)** the 5-year reassessment interval, **(l)** records for the useful
  life of the pipeline. Via `law.cornell.edu/cfr/text/49/195.452` and `phmsa.dot.gov/regulations/title49/section/195.452`.
- **49 CFR §195.450** (HCA definitions), **§195.406** (MAOP), **§195.573** (CP), **§195.50/§195.52**.
- **49 CFR §192.607** (material verification) and **§192.624** (MAOP reconfirmation — **50 % by
  3 July 2028, 100 % by 2 July 2035**), via `ecfr.gov` and the 2019 Gas Transmission "Mega Rule"
  (84 FR, 1 Oct 2019). Cited for the TVC vocabulary; **out of scope** for the CA demo (§1.8).
- **PHMSA** *Stakeholder Communications: Integrity Management* (`primis.phmsa.dot.gov/Comm/IM.htm`);
  the **traceable, verifiable, complete** records standard.
- **AB 864 (2015)**, Gov. Code **§51013.1**, 19 CCR **§2020 et seq.**; **SB 295 (2015)**; the May 2015
  **Refugio / Plains Line 901** release that produced both.
- **Alquist-Priolo Earthquake Fault Zoning Act** PRC §2621 et seq.; **Seismic Hazards Mapping Act**
  PRC §2690 et seq.
- **API RP 1187** (2024, landslide hazards) · **API RP 1160** · **INGAA** *Framework for Geohazard
  Management* (2023) · **ASME B31.4 / B31.8S** · **AMPP-NACE SP0169**.

**Competitive** (page-verified 2026-08-06)
- **New Century Software (a MISTRAS company)** — `newcenturysoftware.com`; *SheetCutter Pro*,
  *TemplateDesigner Pro*, *Facility Manager*; Esri Partner listings at
  `esri.com/partners/new-century-software-…`.
- **DNV** *Synergi Pipeline Integrity* · **ROSEN** *ROSYMS PIM* · **Baker Hughes** · **Metegrity**
  *Visions* · **Dynamic Risk** *Integrity Program Manager* · **EnerSys** *POEMS Program Suite*
  (`enersyscorp.com/software/program-suite/`).
- **Audubon** *Pipeline Integrity Data Integration & Analytics* · **Geosyntec** *Best Practices in
  Geohazard Data Management* · **Teren** on API RP 1187.
- **Esri** ArcGIS Pipeline Referencing / UPDM · **PODS Association**.
- **PHMSA NPMS** — `npms.phmsa.dot.gov` (Public Viewer: one county per session; GIS distribution
  restricted to operators and government officials) · **CA OSFM Pipeline Safety Division** —
  `osfm.fire.ca.gov` (**403** to non-browser clients).

**Data** — Cal OES hosted org (`CA_Oil_Pipeline_2018`, `CA_OilRefineriesandTerminals`); California State
Geoportal `GeoscientificInformation` (`Fault_Lines`, `Liquefaction`); CA Dept. of Conservation / CGS
(`FAM_QFaults`, `MS48_GroundMotion_*`, `MS58_LandslideSusceptibility_Classes`). All endpoints, counts,
field lists, measured mileages and eleven traps: **`pipeline-catalog-ca.json`**.

---

## Modernization (parity release)

> Applies the Experience-Builder-parity features (Phases 1–7). See `strata/recipes/MODERNIZATION.md` +
> `COMPONENT-MANIFEST.md` §8.

- **Structured theme** — the whole look from one `primary` hex, with `success`/`warning`/`danger`
  carrying real semantics (*looked and found nothing* / *nobody looked* / *looked and found a hazard*)
  and a `kpi` override for tabular monospace.
- **App-shell** — `header` (title · vintage · operator/commodity filter · nav · theme), `footer`
  (attribution · the stationing caveat · share · print), and a `splash` titled *"This is a working paper,
  not a filing"*.
- **DataSource linking** — `sourceId: "pipes"` links the two tables, the carto panel and the operator
  chart with **zero** `connections`; `fromWidget` on the four `analysis` outputs feeds the gauge, the
  fault-zone KPI, the dossier and the strip itself. Live `kpi.stat` / `gauge.stat` throughout.
- **Layout nodes** — `splitter` ×3 on page 1 and ×2 on page 2, `panel` (lines and pack, `dock:"top"` on
  phones), `accordion` (provenance · dossier · bring-your-records), `flow-row` (the KPI strip),
  `section mode:"fixed"` (the map box). **No `window`** — deliberately, its opener is a pending emitter.
- **`analysis` widget** — four `clip` runs, one per authority, each publishing its bands as an output
  that two or more widgets consume.
- **`query` widget** — the line-ID/operator search publishing an output rather than a bare filter.
- **`FileDataSource`** — four CSV uploads that turn nine empty lanes into evidenced ones, without a
  single write to any service.
- **Motion, used once and for information** — lanes fade in as their authority answers, so the user can
  see which service is slow. Nothing else animates.
- **i18n** — **EN only** ships: the binding statutes are Californian and English-only. The string table
  is kept so every phrase lives in one place, and the `alignment-strip` contract specifies RTL mirroring
  of the axis, because the first non-US deployment of this pack will be a Gulf operator and a milepost
  ruler that does not mirror is unusable in Arabic.
