# Snowflake Value Map Skill — User Guide

## What It Does

The **Snowflake Value Map** skill analyzes a Snowflake schema and produces a strategic study that answers:

- What use cases is this data deployment already capable of supporting?
- What is missing, and what would it take to close the gaps?
- What is the prioritized roadmap from quick wins to strategic investments?

It combines live schema discovery (table/column metadata, row counts, freshness) with a library of vertical-specific use case catalogs to produce a scored, evidence-backed assessment — not a generic AI pitch deck.

The output is a **Fit Score** (0–10), a **READY / NEAR / GAP classification** for each use case, and a three-phase implementation roadmap.

---

## Supported Verticals

The skill has pre-built use case catalogs for 12 verticals:

| Vertical | File |
|----------|------|
| Technology / SaaS | `references/verticals/technology-saas.md` |
| Financial Services | `references/verticals/financial-services.md` |
| Healthcare | `references/verticals/healthcare.md` |
| Insurance | `references/verticals/insurance.md` |
| Life Sciences & Pharma | `references/verticals/life-sciences-pharma.md` |
| Retail / CPG | `references/verticals/retail-cpg.md` |
| Manufacturing | `references/verticals/manufacturing.md` |
| Energy & Utilities | `references/verticals/energy-utilities.md` |
| Telecom | `references/verticals/telecom.md` |
| Logistics & Transportation | `references/verticals/logistics-transportation.md` |
| Media & Entertainment | `references/verticals/media-entertainment.md` |
| Public Sector | `references/verticals/public-sector.md` |

Verticals outside this list are supported in a best-effort mode with a warning banner on the output.

---

## How to Invoke It

Type `/snowflake-value-map` in the CoCo chat panel. The skill will ask you two questions:

1. **Data source** — live Snowflake account discovery or DDL file/inline SQL
2. **Entity name** — the company or customer the study is for

Then it runs automatically through 5 steps.

---

## The Five-Step Workflow

### Step 1 · Parse & Catalog Deployment

The skill discovers every table and view in your schema, extracts column names and data types, reads all table/column comments from `INFORMATION_SCHEMA`, and classifies each object:

- **Layer** (Silver vs. Gold) — inferred from naming conventions, schema names, and comments
- **Table type** (FACT, DIMENSION, AGGREGATE, REFERENCE)
- **Grain** — what one row represents
- **Domain** — customer, vendor, financial, engineering, etc.

It uses a weighted signal-scoring algorithm to detect the vertical automatically. If two verticals score within 20% of each other, it asks you to confirm before continuing.

**→ Hard stop:** The skill presents the inventory table and detected vertical and waits for your confirmation.

---

### Step 1.5 · Data Grounding (live connection only)

If you have an active Snowflake connection, the skill runs health checks on each cataloged table:

```sql
SELECT COUNT(*), MAX(<timestamp_col>) FROM <table>;
```

Each table is classified as:

| Status | Meaning |
|--------|---------|
| **HEALTHY** | Sufficient rows, fresh data, low NULL density on key columns |
| **STALE** | Has rows but older than the threshold for its type |
| **SPARSE** | Below the expected minimum row count |
| **HOLLOW** | Zero rows — table exists but is empty |
| **DEGRADED** | Key columns have >20% NULLs |
| **MISLEADING** | Sample data contradicts what the schema promises |

HOLLOW and SPARSE tables are treated as non-existent for classification. STALE tables downgrade readiness by one level (READY → NEAR).

---

### Step 2 · Load Vertical Use Case Reference

The skill loads the matching vertical reference file, which contains:

- **Canonical use cases** with required data domains, required data patterns, Snowflake features, and business outcome category
- **Impact weights** (HIGH = 3, MEDIUM = 2, LOW = 1)
- **Industry benchmark ranges** from published analyst reports
- **Marketplace opportunities** for filling data gaps

---

### Step 3 · Cross-Reference & Match

Every canonical use case is evaluated against the deployment inventory:

| Classification | Meaning |
|----------------|---------|
| **READY** | Required data and patterns are fully present — deploy now |
| **NEAR** | Most data exists; 1–2 gaps (missing joins, new features to enable) |
| **GAP** | Fundamental data domain or architecture is absent |
| **N/A** | Use case doesn't align with the entity's footprint |

**Fit Score formula:**

```
fit_score = (Σ impact_weight × classification_score) / Σ impact_weight × 10

Classification scores: READY=1.0, NEAR=0.6, GAP=0.1, N/A=0
```

A score of 10 means every HIGH-impact canonical use case is READY to deploy. A low score on a MDM or operational schema is expected — it means the schema powers analytics, not that it lacks value.

---

### Step 4 · Research Feasibility & Impact

For each READY/NEAR use case, the skill maps Snowflake features to the specific gaps and cites industry benchmark ranges. For GAP use cases, it checks whether Snowflake Marketplace data products can close the missing data domain (reducing effort from High to Medium).

**→ Hard stop:** The skill presents the findings summary and asks you to confirm or adjust before generating the final study.

---

### Step 5 · Produce Strategic Study

The skill writes three output files:

| Format | File | What's in it |
|--------|------|--------------|
| **Markdown** | `<entity>-value-map-study.md` | Full structured study with inventory, fit matrix, gap analysis, value proposition, impact assessment, and roadmap |
| **HTML** | `<entity>-value-map-study.html` | Same content with sidebar navigation, color-coded badges (READY/NEAR/GAP), and use case cards — opens in any browser |
| **PPTX** | `<entity>-value-map.pptx` | 22-slide Snowflake-branded deck: Cover → Exec Summary → Strategic Goals → Fit Matrix → one slide per use case → Roadmap → Business Impact → Next Steps |

Output format is your choice — you can request Markdown only, Markdown + HTML, or all three.

---

## Output Files Explained

### Markdown Study

Eight sections:

1. **Executive Summary** — fit score, classification counts, top 3 opportunities
2. **Entity Strategic Goals** — inferred from data investment patterns, with evidence
3. **Current Deployment Assessment** — full inventory table with rows, freshness, health, and domain
4. **Use Case Fit Matrix** — READY/NEAR/GAP per use case with rationale
5. **Gap Analysis** — data gaps, capability gaps, architecture gaps with effort ratings
6. **Snowflake Value Proposition** — feature-to-gap mapping (which Snowflake feature closes which gap)
7. **Business Impact Assessment** — industry benchmark ranges with a disclaimer that they are not entity-calibrated
8. **Roadmap** — Phase 1 (READY quick wins), Phase 2 (NEAR gap closure), Phase 3 (GAP strategic investment)

### HTML Study

Same content as the Markdown, rendered as a single-page app with:
- Fixed sidebar navigation with scroll-spy
- Color-coded classification badges: green (READY), amber (NEAR), red (GAP), gray (N/A)
- Use case cards with feature pill badges
- Phase-colored roadmap headers

### PPTX Deck

- **Slides 1–4**: Cover, Executive Summary, Strategic Goals, Fit Matrix overview
- **Slides 5–N**: One slide per READY use case (green accent, benchmark callout, feature pills)
- **Slides N+1–M**: One slide per NEAR use case (amber accent, two-column have-vs-gaps layout)
- **Slides M+1–P**: One slide per GAP use case (red accent, numbered path, Marketplace callout)
- **Slide P+1**: Roadmap (three-phase chevron: Phase 1 → Phase 2 → Phase 3)
- **Slide P+2**: Business Impact (2×2 stat grid: Revenue Growth, Cost Reduction, Risk Mitigation, Operational Efficiency)
- **Slide P+3**: Next Steps / Thank You

---

## Inputs the Skill Accepts

### Option 1 — Live Account Discovery (recommended)

Tell the skill which database and schema to analyze. It queries `SHOW TABLES IN SCHEMA` and `INFORMATION_SCHEMA.COLUMNS` automatically. No SQL needed.

```
Analyze: MDM.MDM_INTERFACES
Entity: Acme Corp
```

### Option 2 — DDL File

Provide one or more `.sql` files with `CREATE TABLE` statements. The skill reads them directly.

```
Here's our DDL: /path/to/schema.sql
Entity: Acme Corp
```

### Option 3 — Inline SQL

Paste `CREATE TABLE` statements directly into the chat.

---

## Tips for Better Results

**Use table and column comments.** The skill reads `COMMENT =` clauses and inline SQL comments. A comment like `-- Source: Guidewire ClaimCenter` on an insurance table doubles the vertical detection confidence for that signal. Comments resolving ambiguity (e.g., "grain: one row per customer per day") prevent the skill from asking clarifying questions.

**Include your anchor/fact tables.** The skill scores use cases against what data exists. A schema of only reference tables will produce more GAP classifications than one that includes fact and event tables.

**Provide the entity name.** The study is personalized to the company. Using "My Organization" produces a generic output; using the actual entity name makes the executive summary and PPTX deck usable out of the box.

**Multiple schemas.** If your data is split across schemas (e.g., Silver in `MODELED` and Gold in `INTERFACES`), tell the skill to analyze both. It will merge the inventory and classify layers correctly.

---

## Common Misconceptions

**"A low fit score means the schema is bad."**
No. The canonical use cases for each vertical focus on analytics and AI workloads (product telemetry, billing metering, churn prediction). An MDM or operational data hub schema will score low on these because its value is powering other analytics layers — not as a direct analytics store. The skill surfaces this as "Deployment-Specific Strengths" outside the canonical catalog.

**"GAP means it can't be done."**
GAP means the required data domain is not present in the assessed schema. It does not mean the capability is impossible — it means you need to connect or ingest new data first. The skill provides the path to close each gap.

**"The impact numbers are guarantees."**
All impact figures are prefixed with "Industry benchmarks suggest..." and sourced from published analyst reports (Gartner, Forrester, McKinsey, vendor case studies). They are NOT calibrated to the specific entity's revenue, cost structure, or operational maturity. The study always states what entity-specific data would be needed to produce a real number.

---

## Example Output (this session)

This skill was run against `MDM.MDM_INTERFACES` (Snowflake's internal MDM schema, 71 tables + 69 views).

- **Vertical detected:** Technology / SaaS (HIGH confidence)
- **Canonical Fit Score:** 2.96 / 10 (expected — MDM is a data backbone, not telemetry)
- **READY (deployment-specific):** 4 use cases — Third-Party Risk Dashboard, 360 Account Intelligence, RETL Observability, MDM Quality Monitoring
- **NEAR (canonical):** Customer Health Scoring, Revenue Intelligence, SE Pipeline Intelligence
- **GAP:** Product-Led Growth, Usage-Based Billing, Support Ticket Prediction

The study, HTML report, and 22-slide PPTX were written to the workspace. A Cortex Analyst semantic view (`ACCOUNT_INTELLIGENCE_360`) was subsequently built and deployed to Snowflake from the READY use case findings.

---

## Files Reference

```
~/.snowflake/cortex/skills/snowflake-value-map/
├── SKILL.md                          ← Skill definition and workflow
├── references/
│   ├── snowflake-capabilities.md     ← Snowflake feature library
│   ├── study-template.md             ← Markdown output structure
│   ├── study-template.html           ← HTML design system
│   ├── pptx-generation.md            ← PPTX slide blueprint and helpers
│   └── verticals/
│       ├── technology-saas.md
│       ├── financial-services.md
│       ├── healthcare.md
│       └── ... (12 verticals total)
```

---

*Skill published by: USER\$PMONTEIRO | Version: VERSION\$2 | Installed via Cortex Code skill catalog*
