# Recipe — The Compact (Disaster Data Exchange, reinvented)

A reproducible path to a **data mutual-aid exchange** on strata-core: agencies sign the compact **once,
in blue skies** — registering **Data Mission Ready Packages** (named, typed layer offerings with schema,
refresh SLA, sensitivity tier, and auto-share conditions) and pre-subscribing as **dormant grants** —
so that when a **declaration** hits, the exchange **flips matching grants live in seconds**: shared
layers appear on the incident map wearing **provenance passports**, a per-AOI **Data Grid** shows which
core layers exist and which are missing, everything not pre-negotiated flows through a **213RR-style
tracked data request**, every access is logged, and **stand-down auto-revokes access and renders the
audit trail as the after-action data annex**. Designed **open-design** from 2026 research beyond ESRI
(§1) — the thesis: *pre-position everything expensive (law, trust, packages, protocol) so incident time
is selection and activation, never negotiation.* Full rationale: `DESIGN-PROPOSAL.md`; the v1
clearinghouse is preserved as `RECIPE-v1-tabbed-workbench.md`.

> **Scope (honest).** The **layer substrate is real and live** — MRP offerings bind to the shared CA
> catalog spine (`../emergency-management_common-operating-picture/cop-catalog-ca.json` roles: DWR flood, CAL FIRE perimeters, Cal OES
> shelters, Caltrans roads, HCAI facilities — keyless public services). The **exchange objects** (orgs,
> MRPs, grants, requests, incidents, audit) are app-side stores in the template (seeded compact of 6
> orgs / 14 MRPs / drill history); **production enforcement — token minting, grant revocation, steward
> approvals — requires the Strata-Serve/ESRI backend** 🔶 (short-lived, referer-bound tokens per the
> repo rule; the Delta-Sharing move). Declarations are manually triggered in the template; the FEMA
> declarations feed is the production trigger. Legal pre-clearance is institutional work the app
> operationalizes but cannot sign — ship the EMAC/London-DSA model-agreement appendix. Zero-copy is
> narrated truthfully: shares reference live services; nothing is ever emailed as a zip.

---

## 1. Study — the market beyond ESRI (research 2026-07-20)

**The failure record:** 20+ years of GAO findings on fragmented geospatial sharing (GAO-03-874T);
Katrina's classification walls between NGA and civilian agencies; Harvey coordination working only
because of relationships formed at a 2015 workshop; GAO-19-256 (Puerto Rico: partners lacked even a
shared view of applicable policies). The documented anti-pattern: **data-sharing agreements negotiated
during the disaster** ("the worst time to negotiate a DSA with your DMV is during an active
recovery"); London's standing pre-signed emergency DSA is the existence proof of the alternative.
Today's practice: the sharing fabric is **rebuilt per incident** (ad-hoc AGOL groups, emailed
shapefiles, credentials by phone); FEMA GeoPlatform and state hubs (GATOR, Cal OES) are
discovery-and-catalog with **no grant/activation/subscription machinery**; NAPSG's SARCOP shows a
working multi-agency live exchange but single-mission (SAR).

**The solved halves, waiting to be assembled:** **EMAC** — all 50 states pre-ratified the law; Mission
Ready Packages make offers "in less than a minute"; activation is selection, not negotiation. **ICS
213RR** — the tracked request→approval→fulfillment→demobilization workflow (that data requests lack
entirely). **Copernicus EMS** — authorized triggerers, a standard product portfolio, an SLA clock,
default-open dissemination (570+ activations). **HDX** — org-level trust (verified organizations,
18k datasets, 1.4M users), **Data Grids** (per-crisis red/green completeness of six core categories),
**SDC** (a quantified <3% re-identification gate with ISP fallback), **Signals** (the exchange watches
its own data). **Delta Sharing / Snowflake** — the "share, don't ship" wire model: zero-copy,
revocable recipient grants, short-lived pre-signed URLs, usage telemetry, open protocol. **TAK
federation** — sovereign servers, cert-based revocable trust, policy-brokered boundaries.
**GeoCollaborate** — leader/follower sessions synchronizing trusted references across platforms.
**HXL's caution** — even a successful lightweight standard lost sponsorship (support sunset Jan 2026).

**The 12 gaps this design occupies:** fabric-per-incident · mid-disaster agreements · copies-not-shares
· no 213RR-for-data · no completeness board · no freshness signaling · access outliving incidents ·
improvised sensitivity · discovery≠governance · vendor-boundary sharing · no usage visibility ·
unsolved cross-org trust. **No one has built the "EMAC for data." This is it.**

## 2. UI design spec (front-loaded)

### 2.1 Layout (Template: `open-design` — "data-compact")
- Charter/manifest §10; three §10.3 candidates (`grant-matrix`, `data-passport`, `request-tracker`)
  with day-1 fallbacks (§2.6). Anti-collision: v1 `tabbed-workbench` retired; sharply distinct from
  `geo-atlas` (public discovery — no grants/activation/audit), `incident-anchor` (consumes the
  picture), `lifeline-board` (status fusion). Harvest candidate: **`data-compact`**.
- **Three pages = the EM lifecycle** (`page-nav` is the mode switch):

**BLUE-SKY** — the compact registry: verified orgs (⬡), their **Data MRP cards** (name, layers,
schema, SLA, tier, auto-share conditions), the **grant matrix** (providers × subscribers; hollow ◻ =
dormant, badge = drilled), [+ register MRP] and [drill] actions.

**ACTIVATION** (the demo page):
```
┌ ⬡ The Compact · [BLUE-SKY]·[ACTIVATION]·[STAND-DOWN] · incident ▾ DR-4999 · ● LIVE ─────┐
│ GRANT MATRIX (360px)              │            INCIDENT MAP                              │
│         Cnty-A  CalOES  RedCross  │  AOI boundary · live shared layers, each wearing a  │
│ DWR      ◼lv     ◼lv     ◻dorm    │  PASSPORT chip (⬡org · vintage ● · SLA · tier) ·    │
│ CalFire  ◼lv     ◼lv     ◼lv      │  hatched placeholders where a request is pending    │
│ Cnty-B   ◻dorm   ◼lv     —        │  ┌ DATA GRID — core layers for this AOI ┐           │
│ (cells animate ◻→◼ on activation) │  │ shelters ✓ damage ✓ debris ✗ roads ✓ │           │
│ ── REQUEST QUEUE (213RR-style) ── │  │ utilities ◐ stale (42m > SLA) ┘                  │
│ #12 debris estimate → steward ⏳   │                                                     │
│ #11 shelter capacity → LIVE ✓0:42 │                                                     │
├───────────────────────────────────┴───────────────────────────────────────────────────────┤
│ LEDGER: 14 grants live · 2 requests pending · 3 core layers missing · all access logged   │
└───────────────────────────────────────────────────────────────────────────────────────────┘
```
**STAND-DOWN** — demobilization: grants auto-revoke on declaration close (+recovery tail), the audit
trail renders as the **after-action data annex** (who shared what, to whom, when, how fresh) — export.
- Phone: the Data Grid is the page; matrix/queue as sheets.

### 2.2 Theme
Institutional slate; status palette: live `#2b8a3e` · dormant hollow · pending `#f5b83d` ·
expired/denied `#8a93a6` · missing-core `#e03131`; passports as monospace chips with freshness dots
(token inherited from The Picture); the **◻→◼ activation flip animates** (the demo moment);
stand-down prints calm. Wall mode; EN+AR/RTL. *A treaty room with a live map.*

### 2.3 The differentiators
1. **Dormant grants that activate on declaration** (EMAC × Delta Sharing): auto-share conditions
   evaluated against the incident object (geography · type · requester role); matching cells flip
   live — no calls, no new credentials; grants scoped to the AOI and the incident clock.
2. **Data MRPs**: pre-typed layer bundles with schema/SLA/tier — offers become one-click.
3. **The Data Grid** (HDX): per-AOI completeness of core categories; a missing chip becomes a
   prefilled request; a stale chip (SLA breach) flags automatically (Signals).
4. **213RR for data**: submit → steward approval → time-boxed grant → fulfilled as a **live layer,
   never a zip** → demobilized. Every step tracked.
5. **Provenance passports + usage visibility**: every layer wears org/vintage/SLA/tier; providers see
   consumers — the trust incentive that makes agencies willing to share.
6. **Stand-down semantics**: access dies with the incident; the audit log is the after-action annex.

### 2.4 Wiring (→ `AppLayout.connections`; §5 implements)
incident ▾ → evaluate grants + `navigate`(activation) + matrix flip; matrix cell `featureSelect` →
layer + passport spotlight; grid chip `categorySelect` → prefilled request / stale detail; request
`rowSelect` → detail + `zoomTo`; approve `buttonClick` → `updateRecord` 🔶/`message`; `timer` →
`refresh` passports/grid; [drill] → `flash` + badge; stand-down → `export` annex + revoke; map
`featureSelect` → passport.

### 2.5 Capabilities (sweep in DESIGN-PROPOSAL §8)
page-nav lifecycle · table (matrix/queue/audit) · card/feature-info (MRPs, passports) · kpi+arcade
(grid, ledger) · processing `pointsWithin` (AOI scoping) · timer→refresh · draw (AOI adjust) · export
(annex, grant certificates, incident layers.json) · timeslider (audit replay) · RestDataSource ·
i18n/theme · auth/feature-arcgis 🔶. Skipped, justified: routing, swipe/story, media-pager, reporter.

### 2.6 §10.3 New-widget blocks (fallbacks ship day 1)
1. **`grant-matrix`** → *fallback* `table` + arcade cell tinting + `featureSelect`.
2. **`data-passport`** → *fallback* `card` + `feature-info` on the share record + freshness `timer`.
3. **`request-tracker`** → *fallback* `table` + status arcade + approve `button` (🔶/local).

## 3. Data model (source of truth)
**Layer substrate:** the shared CA catalog spine — MRP offerings reference real role-tagged services
in `../emergency-management_common-operating-picture/cop-catalog-ca.json` (perimeters, shelters, flood, roads, hospitals, DINS).
**Exchange objects** (template: seeded local stores; production: Serve/ESRI layers 🔶):
`org {id, name, verified, contacts}` · `mrp {id, org, name, layers[], schema, sla_min, tier
(open|by-request|restricted-ISP), conditions}` · `grant {mrp, subscriber, state
(dormant|live|expired|revoked), drilled_at, incident?}` · `incident {id, name, aoi, declared_at,
closed_at?, triggerer}` · `request {id, layer_need, justification, requester, steward, state, sla}` ·
`audit {ts, actor, verb, object}`. Core-layer checklist for the Data Grid (shelters, damage, debris,
utilities, roads, boundaries) configurable per compact.

## 4. Verify first (terminal)
```bash
# the substrate services are the COP's — reverify the workhorses:
curl -s "https://services.arcgis.com/BLN4oKB0N1YSgvY8/arcgis/rest/services/CalOESShelters3/FeatureServer/0/query?where=1%3D1&returnCountOnly=true&f=json"
curl -s "https://services1.arcgis.com/jUJYIo9tSA7EHvfZ/arcgis/rest/services/CA_Perimeters_NIFC_FIRIS_public_view/FeatureServer/0/query?where=1%3D1&returnCountOnly=true&f=json"
# production trigger source (note; needs no key):
#   FEMA declarations API — https://www.fema.gov/api/open/v2/DisasterDeclarationsSummaries
```
Gotchas: grant *enforcement* is a backend concern — the template demonstrates semantics, not security;
never store credentials in the app; Serve short-lived referer-bound tokens are the production path.

## Guided wizard

| # | Question | Options → **default** | Assigns |
|---|---|---|---|
| 1 | Compact name? | → **"The Compact — California Data Mutual Aid"** | header |
| 2 | Seeded orgs? | **6 (DWR, CAL FIRE, Cal OES, 2 counties, Red Cross)** · my roster | registry |
| 3 | Core-layer checklist? | **shelters·damage·debris·utilities·roads·boundaries** · custom | Data Grid |
| 4 | Sensitivity tiers? | **open / by-request / restricted-ISP (HDX)** · custom | MRP + grant flow |
| 5 | Trigger? | **manual (template)** · FEMA declarations feed | incident creation |
| 6 | Exchange backend? | **local (template)** · Serve/ESRI 🔶 | writes + tokens |
| 7 | Recovery tail? | **14 days after close** · custom | auto-revoke clock |
| 8 | Outputs? `[multi]` | **after-action annex + grant certificates + incident layers.json** | exports |
| 9 | Theme? | **institutional slate / EN+AR / wall** | ThemeSpec |

## 5. Prompt-script (run in order)
```
A. /new-app — "The Compact", open-design per §2.1: pages bluesky/activation/standdown (page-nav =
   lifecycle); institutional-slate ThemeSpec (§2.2); seed the compact stores (6 orgs, 14 MRPs across
   the CA substrate services, drill history, a closed prior incident for the annex demo).
B. BLUE-SKY: org registry cards; MRP cards (name, layers→catalog refs, schema, SLA, tier, auto-share
   conditions); grant matrix (grant-matrix fallback: table + arcade) with [register MRP] + [drill]
   (flash + drilled badge + audit entry).
C. ACTIVATION: incident ▾ (manual trigger creates the incident object: name, AOI polygon, clock) →
   evaluate every dormant grant's conditions (geography intersects AOI · type · subscriber role) →
   matching cells animate ◻→◼; each live grant adds its real substrate layer to the incident map
   wearing a data-passport chip (org · vintage dot · SLA · tier).
D. DATA GRID: per-AOI completeness of the core checklist — present ✓ (a live grant covers it),
   missing ✗ (click → prefilled 213RR request), stale ◐ (passport vintage > SLA, flagged by timer).
E. REQUEST QUEUE (request-tracker fallback): submit (layer need, justification, requester) → steward
   approve/deny (updateRecord 🔶 else local+toast) → time-boxed grant minted → fulfilled as a live
   layer (hatched placeholder becomes solid) → demob with the incident. Full status trail.
F. LEDGER + audit: every verb logged (activate, access, approve, revoke); ledger strip totals; all
   consumption visible to providers (usage-visibility panel per MRP).
G. STAND-DOWN: close the incident → grants auto-revoke (+ recovery tail Q7) → render the after-action
   data annex (who/what/whom/when/freshness) → export PDF + incident layers.json; audit replay via
   timeslider.
H. Wall mode; EN/AR; model-agreement appendix (EMAC articles + London standing DSA) linked from
   BLUE-SKY.
I. Verify §6; log gaps to §7.
```

## 6. Verify
| Check | Pass |
|---|---|
| Blue-sky: orgs verified-badged; MRPs typed (schema/SLA/tier/conditions); drills logged + badged | ☐ |
| Declaration → matching dormant grants flip live in <5 s, cells animate, layers appear with passports | ☐ |
| Grants scope to the AOI (pointsWithin) and the incident clock; non-matching stay dormant | ☐ |
| Data Grid truthful: ✓ only when a live grant covers the core layer; ✗ click → prefilled request; ◐ on SLA breach | ☐ |
| Requests: tracked submit→approve→fulfill→demob; fulfilled as live layers, never files | ☐ |
| Passports on every shared layer (org · vintage · SLA · tier); providers see consumers | ☐ |
| Stand-down: auto-revoke + recovery tail honored; annex export lists every share and access | ☐ |
| Writes gated 🔶; template semantics honest (local store + toast); no credential ever stored | ☐ |
| Distinct from geo-atlas/COP/lifeline-board in silhouette and verbs | ☐ |
| Beats the bar: dormant-grant activation + data-213RR + Data Grid + passports + demob exist together in NO incumbent (HDX, GeoPlatform, GATOR, SARCOP, GeoCollaborate) | ☐ (judge) |

## 7. Harvest
- **`grant-matrix`**, **`data-passport`**, **`request-tracker`** widgets — §10 candidates; silhouette
  **`data-compact`** (utilities mutual aid, health exchanges, port consortiums).
- **"Delta Sharing for layers"** on Strata Serve: GeoParquet + STAC manifest + short-lived
  referer-bound URLs + revocable recipient objects — the open-protocol roadmap item this recipe wants.
- The **incident object + activation evaluator** belongs in a shared package (COP and Lifelines want
  the same declaration trigger).
- FEMA declarations feed parser (production trigger).
- The audit-annex exporter generalizes (every EOC product wants "the annex writes itself").

## 8. Sources
- **Humanitarian:** HDX (data.humdata.org FAQ; 2024 stats) · Centre for Humanitarian Data · HDX Data
  Grids · HDX Signals (CERF $46M) · HDX SDC (<3% threshold; ISP) · HXL (hxlstandard.org; support
  sunset 2026-01-31) · IATI.
- **US practice:** GAO-03-874T · GAO-04-703 · GAO-19-256 · Katrina case studies (PSU GEOG-882;
  Springer) · Harvey/Maria studies (PMC) · NAPSG guidance + SARCOP · GISCorps · EMAC (emacweb.org;
  MRPs; REQ-A) · ICS 213RR (FEMA training) · disasters.geoplatform.gov + FEMA GRC · FDEM GATOR ·
  Cal OES Geospatial Coordination Hub · NIFS practice (GovTech, LA fires).
- **Exchange paradigm:** Snowflake Secure Data Sharing + Marketplace · Databricks Delta Sharing (open
  protocol) · Source Cooperative (Radiant Earth) · GeoCollaborate (NOAA tech partnerships; NWS best
  practices) · TAK federation (cert trust; Federation Hub).
- **Activation model:** Copernicus EMS Rapid Mapping (portfolio; authorized users; 570+ activations).
- **Pre-agreement model:** Marcman disaster-recovery-data 2025 · ICO emergency data-sharing code ·
  London standing emergency DSA (GLA).
- **Internal:** `DESIGN-PROPOSAL.md` · `RECIPE-v1-tabbed-workbench.md` ·
  `../emergency-management_common-operating-picture/cop-catalog-ca.json` (substrate) · manifest §10 · app-design guide.

---

## Modernization (parity release)
> Native: page-nav as lifecycle · RestDataSource exchange objects · arcade status/epistemic classing ·
> timer→refresh freshness (Signals) · pointsWithin AOI scoping · export-from-state annex ·
> the shared-catalog substrate (4th product on the spine).
