# strata-recipes

**Production-grade recipes for building real GIS web applications with AI.**

Each recipe in this repository is a complete, reproducible path from a business question to a working,
map-centric web application — the research behind the design, the layout, **curl-verified live data
sources**, a verification checklist, and a prompt-script an AI agent runs end to end.

Pair a recipe with **[strata-app-builder](https://github.com/tabaqatdev/strata-app-builder)** and an AI
assistant. You supply the recipe; the assistant builds the app.

```
recipe (what to build, on which verified data)
      +
strata-app-builder (how to build it — React + MapLibre + ESRI Web Map JSON)
      +
AI assistant (does the building)
      =
a finished application
```

---

## Quick start

**1. Get the builder.**

```bash
git clone https://github.com/tabaqatdev/strata-app-builder.git
cd strata-app-builder
```

**2. Get the recipes** — cloned *into* the builder as `recipes/`, so the two sit together:

```bash
git clone https://github.com/tabaqatdev/strata-recipes.git recipes
```

**3. Build.** Open the project in Claude Code and point `/recipe` at the one you want:

```
/recipe recipes/hydrology/watershed-and-water-resources-portal/recipe.md
```

Or just describe it — *"build the watershed portal"* — and the assistant will find the recipe.

The build flow is: **wizard** (a short, grouped interview with sensible defaults) → **verify the data**
(§4 is run against the live services *before* any code is written) → **build** → **test** → **run**. The
app is built into `app/` beside the recipe, so the two travel together.

> **Not using Claude Code?** A recipe is plain Markdown and self-contained. Paste it into any capable
> assistant along with the strata-app-builder README and ask it to follow §5.

---

## What a recipe contains

Every finished recipe carries the same five sections. This is the contract — a file missing §3 or §4 is a
**scaffold**, not a recipe, and must not be built.

| Section | What it holds |
|---|---|
| **§1 Study** | How the market frames this problem — page-verified competitive research, the incumbent products, and where this design goes further. Never written from memory. |
| **§2 Layout** | The silhouette and the app template assignment — the app's shape before a line of code. |
| **§3 Data sources** | Real services with **real field names**, curl-checked live. A fabricated endpoint is worse than a blank. |
| **§4 Verify** | The probe to run *before* building: identity, counts, page size, field population, value shape, extent, CORS posture. |
| **§5 Prompt-script** | The consecutive commands that create the map, add and style layers, write popups, assemble panels and widgets, and export. |
| **Guided wizard** | An Instant-App-style configuration interview run before §5. Collects properties only — it never invents data or widens scope. |

The rule that matters most: **§3 and §4 are measured, not remembered.** Every field name, count and
extent in these files came out of a real response.

---

## The catalog

**20 recipes, in 15 of the 16 published categories.** Every recipe here is researched, data-verified, and
has been built into a working application at least once.

Categories — and the order they appear in below — follow the published catalog at
**[tabaqat.net/solutions](https://tabaqat.net/solutions)**. Every category listed there has a section
here, including the ones still empty, so you can see what is landing next.

### Emergency Management and Response · 6

| Recipe | What it answers |
|---|---|
| [`common-operating-picture`](emergency-management-and-response/common-operating-picture/) | **The Picture** — pick an incident and its picture assembles itself: perimeter, damage, shelters, evacuation zones, closures, cameras. |
| [`live-dashboard`](emergency-management-and-response/live-dashboard/) | **The Watch Wall** — an unattended EOC wall that self-rotates through views, shows how fresh every feed is, and breaks rotation to escalate. |
| [`critical-infrastructure`](emergency-management-and-response/critical-infrastructure/) | **Lifelines** — the 8 FEMA community lifelines as a status ledger, coloured by live feeds and confirmed by humans. |
| [`hazard-impact-analyzer`](emergency-management-and-response/hazard-impact-analyzer/) | Draw or import a flood / fire / storm / hazmat zone and answer *"who and what is inside it?"* |
| [`avl-tracker`](emergency-management-and-response/avl-tracker/) | Where is every unit, what is its status, and which one is nearest to this incident? |
| [`disaster-data-exchange`](emergency-management-and-response/disaster-data-exchange/) | **The Compact** — agencies register data-sharing packages in blue skies, so sharing is instant when it matters. |

### Geodata Management · 1

| Recipe | What it answers |
|---|---|
| [`atlas-open-data-hub`](geodata-management/atlas-open-data-hub/) | **Geo Atlas** — one open-data atlas product serving many catalogs from a single build. |

### Financial Services · 1

| Recipe | What it answers |
|---|---|
| [`branch-atm-dashboard`](financial-services/branch-atm-dashboard/) | Is the estate healthy, will we beat the SLA clock on this dead ATM, and who runs out of cash when? |

### Real Estate · 1

| Recipe | What it answers |
|---|---|
| [`site-selection`](real-estate/site-selection/) | **The Swing Slate** — which site leads, what its score is made of, and how far your assumptions must move before another wins. |

### Utilities · 1

| Recipe | What it answers |
|---|---|
| [`network-operations`](utilities/network-operations/) | A shared, keyless network picture — grid posture, customers out, active PSPS, peak wind. |

### Transportation · 1

| Recipe | What it answers |
|---|---|
| [`airports-aviation`](transportation/airports-aviation/) | **The Altimeter** — every ceiling that governs this spot, which one bites first, and by how many feet. |

### Health · 1

| Recipe | What it answers |
|---|---|
| [`access-to-care`](health/access-to-care/) | **Who's Outside** — how many people fall beyond the standard that actually applies here, and who are they? |

### Environment · 1

| Recipe | What it answers |
|---|---|
| [`agriculture-land-use`](environment/agriculture-land-use/) | **The Ledger** — acre by acre, class by class, what this farmland became between two survey cycles. |

### Education · 1

| Recipe | What it answers |
|---|---|
| [`campus-operations`](education/campus-operations/) | **The Standards Rack** — buildings, assets, space and safety on one campus map, measured against the lines somebody else drew. |

### Industry · 1

| Recipe | What it answers |
|---|---|
| [`mining-and-concession-compliance`](industry/mining-and-concession-compliance/) | **The Rule Cascade** — may this mine lawfully sell to a public agency, and what can the public record *not* show? |

### Marketing · 1

| Recipe | What it answers |
|---|---|
| [`catchment-and-market-share-analyzer`](marketing/catchment-and-market-share-analyzer/) | How much of this share number is the market, and how much is our own choice of denominator? |

### Energy · 1

| Recipe | What it answers |
|---|---|
| [`pipeline-integrity-evidence-pack`](energy/pipeline-integrity-evidence-pack/) | **The Alignment Strip** — milepost by milepost: what can be proved today, and which record only the operator holds. |

### Telecom · 1

| Recipe | What it answers |
|---|---|
| [`underserved-gap-and-take-rate`](telecom/underserved-gap-and-take-rate/) | Which gap is this — the one nobody *can* buy from, or the one nobody *does* buy from? |

### Local Government · 1

| Recipe | What it answers |
|---|---|
| [`public-information-map`](local-government/public-information-map/) | *There's a crew outside with a saw — what is it, how long will it be there, and can I still act on it?* |

### Hydrology · 1

| Recipe | What it answers |
|---|---|
| [`watershed-and-water-resources-portal`](hydrology/watershed-and-water-resources-portal/) | The question every water figure hides: *is this measured, or is it modelled?* |

### GeoAI · 0

*No recipes published yet.*

---

## Repository layout

```
recipes/
  <category>/
    <solution>/
      recipe.md
```

Categories follow the published catalog at **https://tabaqat.net/solutions** — lowercase, hyphenated.

---

## Roadmap

This catalog is growing to cover **every solution published at
[tabaqat.net/solutions](https://tabaqat.net/solutions)** — all 16 categories — and further solutions
beyond that list.

Recipes land **one at a time**, each into its category section above. A recipe is published only once §1
Study is page-verified and §3/§4 are curl-checked against live services. That bar is the point: a recipe
naming a service that does not exist, or a field that is never populated, costs more than no recipe at
all — so a category filling up slowly is the intended pace, not a gap.

---

## Related repositories

- **[tabaqatdev/strata-app-builder](https://github.com/tabaqatdev/strata-app-builder)** — the Claude Code
  template these recipes build on. React + MapLibre GL JS, ESRI Web Map JSON as the map contract. MIT.
  **You need this repository too** — recipes describe *what* to build; the builder is *how*.
- **[Strata Server](https://tabaqat.net/platform)** — serves GeoParquet as a FeatureServer.
- **ArcGIS REST** — recipes read public and secured FeatureServers directly.

---

## License

<!-- TODO: confirm before making this repository public. -->

© Tabaqat. See `LICENSE`.
