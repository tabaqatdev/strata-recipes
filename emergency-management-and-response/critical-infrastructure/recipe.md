# Recipe — Lifelines (Critical Infrastructure / Community Lifelines, reinvented)

A reproducible path to a **live Community-Lifelines board** on strata-core, for California: the **8
FEMA lifelines as a status ledger** whose colors are **suggested by live feeds and confirmed by
humans**, a full-bleed map fusing the **Cal OES 15-minute outage feed with the asset registry**
("which hospitals are inside the outage *right now*"), a **cascade lens** that renders downstream
lifeline impacts when you click an outage or sketch a hazard (power → water → hospitals → shelters,
confidence-tagged), and a **Senior Leadership Brief generated from live state** — the FEMA slide,
without the PowerPoint ritual. Designed **open-design** from 2026 research beyond ESRI (§1), with a
second research pass that **internet-verified additional CA data endpoints before any synthesis**.
Full rationale: `DESIGN-PROPOSAL.md`; the v1 criticality ranked-list is preserved as
`RECIPE-v1-ranked-list.md`.

> **Scope (honest).** Assets and feeds are cataloged in `ci-catalog-ca.json`: the 2,339-service CA
> library with **186 services tagged to the 8 lifelines**, plus **10 external feeds — 7 verified live**
> (CEC plants; CEC transmission — **layer /2**, the /0 404s; SWRCB SABL water areas — `objectIdField:
> OBJECTID_1`; HCAI facilities — seismic ratings are CSV-joined, not spatial; NASA-NCCS HIFLD mirror —
> **~2019 archive vintage, labeled**; Cal OES power-outage areas — 15-min, the fusion feed), and 3
> flagged: CEC Substation is **token-gated (499)**, NREL AFDC env-blocked, USACE NID payload-unverified
> — re-test from the browser, never silently synthesize. **Water-system and fuel status have no
> machine-readable feed** — those lifelines render honest gray with a "needs report" affordance.
> Cascade edges are **geographic/heuristic (Rinaldi-typed) until utility topology exists** — every
> item carries a confidence tag. Color confirm/override writes need an ESRI backend 🔶 (local+toast
> otherwise). HIFLD Open retired Aug 2025 — registry staleness is a displayed property, not a secret.

---

## 1. Study — the market beyond ESRI (research 2026-07-20)

**Doctrine:** FEMA Community Lifelines are **8** (Water Systems added Aug 2023): Safety & Security ·
Food, Hydration, Shelter · Health & Medical · Energy · Communications · Transportation · Hazardous
Materials · Water Systems; colors green/yellow/red/gray with component rollups (Toolkit v2.1). FEMA's
**2024 doctrine review explicitly asks what standard data should feed each component** — the practice
today is hand-assembled SLB slides and spreadsheets. **DHS CLSS** (G&H Intl., free to SLTTs since
~Apr 2025, 152 agencies piloted) structures the *reporting*; it does not auto-fuse live feeds into
suggested colors. **One Concern Domino** proved dependency-aware *downtime* modeling (first-to-fail
ranking) then pivoted to insurers/Japan — the US public-sector cascade view is unserved. **HIFLD Open
is dead** (Aug 2025): layers scattered to HSDL archives, EIA/HHS/FCC originators, and state mirrors.
**Live-status world:** Cal OES Power Outage Areas (**public ArcGIS FeatureServer, 15-min**,
PG&E+SCE+SDG&E), EAGLE-I/ODIN (county outage, gov-gated real-time), PowerOutage.us (EM API,
invite-gated), CAISO grid condition (open OASIS), FCC DIRS (daily % cell sites down per county;
granular agency-gated), SDWIS (quarterly, not status), GasBuddy fuel tracker (no API). **Cascade
analysis** lives in Argonne consulting PDFs (EPfast/Restore, RRAP) and Rinaldi's taxonomy — **no
shipping public-sector web app shows click-hazard → downstream lifeline impacts.** The three empty
intersections this product occupies: feed-suggested colors · outage×asset fusion · the cascade lens —
with the generated SLB as the adoption wedge.

## 2. UI design spec (front-loaded)

### 2.1 Layout (Template: `open-design` — "lifeline-board")
- Charter/manifest §10; three §10.3 candidates (`lifeline-chip`, `cascade-lens`, `feed-bar`) with
  day-1 fallbacks (§2.6). Anti-collision: v1 `ranked-list` retired; distinct from `incident-anchor`
  (cop — incident rail/editorial feed vs **status ledger/fusion+cascade**) and every other emergency
  sibling. Harvest candidate: **`lifeline-board`**.

```
┌ HEADER: ● LIVE · jurisdiction ▾ · op-period ▾ · wall · [Generate SLB] ───────────────┐
│ LIFELINE LEDGER (330px)      │            FULL-BLEED MAP                             │
│ ⚡ Energy       ●YELLOW ↘     │  Cal OES outage polygons (15-min, age-badged) ·       │
│   suggested: 61k custs out   │  lifeline assets (plants/hospitals/water/shelters/    │
│   ✎ confirm / override       │  towers) · transmission · FUSION STRIP:               │
│ ✚ Health & Med ●GREEN        │  "11 assets in outage areas now" ·                    │
│ ▢ Water Systems ●GRAY        │  CASCADE LENS (click outage / sketch hazard):         │
│   no feed — needs report     │   ⚡→▢ 3 water systems de-energized (geo, high conf)   │
│ … all 8, FEMA colors;        │   ▢→✚ 1 hospital on system #0710010 (heuristic)      │
│ suggested=hollow,            │   downstream list → zoom/flash                        │
│ confirmed=solid              │                                                       │
├──────────────────────────────┴───────────────────────────────────────────────────────┤
│ FEED BAR: CalOES outages ⬤4m · CAISO Normal ⬤9m · DIRS 1.2% ⬤daily · SABL ⬤static    │
└──────────────────────────────────────────────────────────────────────────────────────┘
```
- Pages: `board` (fixed) + `slb` (print/export view). Catalog drawer (2,339 library) inherited from
  The Picture. Phone: ledger is the page; map per lifeline.

### 2.2 Theme
FEMA-literal status colors (green `#2b8a3e` / yellow `#f5c542` / red `#e03131` / gray `#8a93a6`) on
dark ops base; **suggested = hollow chip, confirmed = solid** (the epistemic state is a pixel);
freshness dots + cadence labels on every feed (15-min / daily / static / archive-2019); cascade edges
as dashed arcs, opacity = confidence; wall mode; EN+AR/RTL; the SLB export mirrors FEMA's own grid so
leadership sees a familiar slide.

### 2.3 The differentiators
1. **Feed-suggested colors** (FEMA's 2024 ask, answered): Energy from Cal OES outage totals + CAISO
   condition; Communications from DIRS daily %; Health from HCAI-in-outage count; thresholds
   documented per lifeline; human confirm/override (attributed) is the CLSS-compatible workflow.
2. **Fusion strip** (`pointsWithin`): assets × live outage polygons, per lifeline, recomputed each
   15-min pull — "which hospitals/water systems/shelters are dark *now*."
3. **Cascade lens**: outage click or hazard sketch → Rinaldi-typed heuristic fan-out (energy →
   water-systems via SABL area containment → health via facility-in-area → shelter) rendered as arcs +
   a confidence-tagged downstream list. Labeled heuristic until utility topology exists.
4. **Generated SLB**: the Senior Leadership Brief from live state — colors, limiting factors, trend,
   fusion counts, cascade highlights. Zero re-typing.
5. **Honest gray**: water/fuel status has no feed — the ledger says so and asks for a report.

### 2.4 Wiring (→ `AppLayout.connections`, implemented in §5)
ledger `rowSelect` → map `filter`/`showHide` + fusion `showStatistics`; outage `featureSelect` →
cascade `selectByGeometry`+`viewInTable`; cascade item `rowSelect` → `zoomTo`+`flash`; confirm
`buttonClick` → `updateRecord` 🔶/`message`; `timer` → `refresh` (15-min) → suggested colors recompute;
[Generate SLB] → `export`; jurisdiction ▾ → `filter` all; drawer `rowSelect` → `showHide`; draw
`sketchComplete` → cascade what-if.

### 2.5 Capabilities (sweep in DESIGN-PROPOSAL §8)
processing `pointsWithin` (the fusion join) + `withinDistance`/`buffer`/`aggregate` · routing (nearest
operable facility on failure) · timeslider (outage replay) · export (SLB + map pack) · arcade
(status + suggested/confirmed classing) · RestDataSource · timer→refresh · draw · theme/i18n ·
feature-arcgis/auth 🔶. Deliberately unused: swipe/story/media-pager/reporter/weighted-overlay.

### 2.6 §10.3 New-widget blocks (fallbacks ship day 1)
1. **`lifeline-chip`** → *fallback* `kpi` + arcade tint + confirm `button`.
2. **`cascade-lens`** → *fallback* `pointsWithin` fan-out → grouped `table` + GeoJSON arc layer.
3. **`feed-bar`** → *fallback* `text` chips + `timer → refresh` (freshness pattern from The Picture).

## 3. The data catalog (source of truth)
**`ci-catalog-ca.json`** via **`build_ci_catalog.py`** (reuses the Geo Atlas CA parser): library
2,339; **186 lifeline-tagged** — transportation 75 · water-systems 36 · energy 20 · safety-security 17
· health-medical 13 · communications 10 · food-hydration-shelter 10 · hazmat 5; **external** block =
the 10 internet-verified feeds with status + gotcha strings (verified-live / TOKEN-GATED /
env-blocked / payload-unverified). Extend `LIFELINES` regexes and `EXTERNAL` as feeds evolve; rerun
after re-crawls.

## 4. Verify each URL first (terminal)
```bash
# CEC plants (keyless; wkid 4269 → outSR=4326):
curl -s "https://services3.arcgis.com/bWPjFyq029ChCGur/arcgis/rest/services/Power_Plant/FeatureServer/0/query?where=1%3D1&returnCountOnly=true&f=json"
# CEC transmission — LAYER /2 (the /0 404s):
curl -s "https://services3.arcgis.com/bWPjFyq029ChCGur/arcgis/rest/services/Transmission_Line/FeatureServer/2?f=json" | python3 -c "import sys,json;d=json.load(sys.stdin);print(d['name'],d['geometryType'])"
# SABL water areas (objectIdField OBJECTID_1):
curl -s "https://gispublic.waterboards.ca.gov/portalserver/rest/services/Drinking_Water/California_Drinking_Water_System_Area_Boundaries/FeatureServer/0?f=json" | python3 -c "import sys,json;d=json.load(sys.stdin);print(d['objectIdField'],[f['name'] for f in d['fields']][:8])"
# HCAI facilities:
curl -s "https://services5.arcgis.com/fMBfBrOnc6OOzh7V/arcgis/rest/services/facilitylist/FeatureServer/0/query?where=1%3D1&returnCountOnly=true&f=json"
# Cal OES outage areas — resolve the concrete layer from the CalOES org / gis.data.ca.gov item, then:
#   .../FeatureServer/0/query?where=1%3D1&returnCountOnly=true&f=json   (expect 15-min freshness)
```
Gotchas: CEC Substation returns **499 Token Required** — use the HIFLD mirror; NASA mirror is a 2019
archive (label it); hosted `f=geojson` casing varies; everything `outSR=4326`.

## Guided wizard

| # | Question | Options → **default** | Assigns |
|---|---|---|---|
| 1 | Title? | → **"Lifelines — California"** | header |
| 2 | Jurisdiction scope? | **statewide + county ▾** · one OA | filter + SLB header |
| 3 | Lifeline set? | **all 8 (2023 doctrine)** · legacy 7 | ledger rows |
| 4 | Fusion feed? | **Cal OES outages (15-min)** · my utility feed | the fusion join source |
| 5 | Suggested-color thresholds? | **defaults (documented per lifeline)** · custom | arcade rules |
| 6 | Confirm/override store? | **local (template)** · ESRI layer 🔶 | writes path |
| 7 | Cascade mode? | **heuristic (labeled)** · off until utility data | lens behavior |
| 8 | Outputs? `[multi]` | **SLB PDF + map pack + CSV** | export wiring |
| 9 | Theme? | **FEMA-status dark / EN+AR / wall** | ThemeSpec |

## 5. Prompt-script (run in order)
```
A. python3 build_ci_catalog.py — confirm 2339/186/8 + external 10(7). Register as RestDataSource.
B. /new-app — "Lifelines — California", open-design per §2.1: pages board + slb; FEMA-status ThemeSpec
   (§2.2) with suggested/confirmed + freshness arcade tokens; header (LIVE, jurisdiction ▾, op-period,
   wall, Generate SLB).
C. Map spine: Cal OES outage polygons (15-min timer→refresh, age badge) + the 8 lifelines' asset
   layers from the tagged core + external verified (CEC plants, transmission /2, SABL, HCAI, HIFLD
   mirror labeled archive) + transmission lines context.
D. LEDGER: 8 lifeline-chip rows (fallback kpi+arcade+button): color = suggested-by-feeds (hollow;
   thresholds per §2.3.1) until confirmed (solid, attributed via updateRecord 🔶 else local+toast);
   trend arrow vs last period; limiting-factor line; water/fuel render honest gray + "needs report".
E. FUSION: pointsWithin(assets × outage polygons) per lifeline → fusion strip counts + tinted assets;
   recompute on every refresh; rowSelect a lifeline → its assets + overlays + fusion detail.
F. CASCADE LENS: outage featureSelect or draw sketchComplete → heuristic fan-out: energy→water
   (SABL areas intersecting outage), water→health (HCAI facilities within affected SABL areas),
   →shelters; render dashed arcs (GeoJSON) + downstream table with confidence tags + zoom/flash;
   "heuristic — validate with utility data" label pinned to the lens.
G. FEED BAR: freshness chips w/ cadence labels (15-min / daily / static / archive-2019); CAISO
   condition + DIRS daily % as manual-entry chips until parsed feeds exist (labeled).
H. [Generate SLB] → export: the FEMA grid from live state (colors, limiting factors, fusion counts,
   cascade highlights, feed vintages) + per-lifeline map pack. Catalog drawer (2,339) inherited from
   The Picture. Wall mode; EN/AR.
I. Verify §6; log gaps to §7.
```

## 6. Verify
| Check | Pass |
|---|---|
| Ledger shows 8 lifelines; suggested colors hollow, confirmed solid; overrides attributed | ☐ |
| Fusion strip: real assets inside real outage polygons, per lifeline, refreshed ≤15-min | ☐ |
| Cascade lens: outage click → typed, confidence-tagged downstream list + arcs; heuristic label visible | ☐ |
| Water & fuel lifelines render honest gray with "needs report" — never fake green | ☐ |
| SLB PDF generated from state matches FEMA's grid; zero re-typing; feed vintages printed | ☐ |
| CEC /2 gotcha, SABL OBJECTID_1, HIFLD archive labeling all honored in the running app | ☐ |
| Token-gated/unverified feeds appear with status flags, not as silent absences | ☐ |
| Sketch a hazard → what-if cascade renders without any live outage | ☐ |
| Beats the bar: feed-suggested colors + outage×asset fusion + cascade + generated SLB exist together in NO incumbent (CLSS, RAPT, EAGLE-I, Domino) | ☐ (judge) |

## 7. Harvest
- **`lifeline-chip`**, **`cascade-lens`**, **`feed-bar`** widgets (fallbacks shipped) — §10 candidates;
  silhouette **`lifeline-board`** (grid ops, ports, campuses).
- The **status:suggested/confirmed epistemic pattern** belongs in the theme/arcade layer.
- A **dependency-edge store** (typed asset→asset edges) — future `@strata/data-source` concern.
- Parsers worth building: FCC DIRS daily report → county %, CAISO OASIS condition.
- Catalog spine now serves three products (Atlas, COP, Lifelines) — promote the role/lifeline tagging
  convention into the shared builder.

## 8. Sources
- FEMA: fema.gov/lifelines (8 lifelines, 2023 Water Systems addition; 2024 doctrine review) ·
  Lifelines Toolkit v2.1 PDF · RAPT user guide 2025 · Ohio EMA SLB deck.
- DHS/CISA: CLSS fact sheet + G&H/CUSEC pages · CISA Gateway · Infrastructure Dependency Primer ·
  HIFLD transition (NAPSG webinar, hsdl.org/hifld, geoMusings).
- Market: One Concern Domino (launch release, TechCrunch/SOMPO, Crunchbase) · UrbanFootprint grid
  resilience · Argonne EPfast/Restore + RRAP · Rinaldi et al. 2001.
- Feeds: data.ca.gov Power Outage Areas (Cal OES, 15-min) · PowerOutage.us EM API · DOE EAGLE-I/ODIN ·
  CAISO OASIS · FCC DIRS (Feb-2025 mandatory rule; CRS R48776) · EPA SDWIS / CA Drinking Water Watch ·
  GasBuddy tracker · PG&E partner data portals · CPUC outage-map index.
- Verified endpoints (2026-07-20 agent): CEC Power_Plant/0 · Transmission_Line/**2** · Substation
  (token-gated) · SWRCB SABL · HCAI facilitylist · NASA NCCS hifld_open energy + public_health ·
  NID/NREL/NLD flagged unverified.
- Data: `data_sources_ca.md` (2026-07-16) → `ci-catalog-ca.json` (`build_ci_catalog.py`).
- Internal: `DESIGN-PROPOSAL.md` · `RECIPE-v1-ranked-list.md` · manifest §10 · app-design guide.

---

## Modernization (parity release)
> Native: RestDataSource catalog spine (3rd product on it) · timer→refresh 15-min loop · arcade
> epistemic classing (suggested/confirmed) · pointsWithin fusion · export-from-state SLB ·
> panels/windows shells · honest-gray as a first-class state.
