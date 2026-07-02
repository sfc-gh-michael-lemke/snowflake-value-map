# Snowflake Value Map

A [Cortex Code](https://docs.snowflake.com/en/user-guide/cortex-code/cortex-code) skill that analyzes a Snowflake schema and produces a scored, evidence-backed strategic study — classifying every canonical use case for a given vertical as **READY**, **NEAR**, or **GAP** based on what data is actually present.

## What It Produces

| Output | Description |
|--------|-------------|
| **Fit Score (0–10)** | Weighted score across all canonical use cases for the detected vertical |
| **Use Case Fit Matrix** | READY / NEAR / GAP classification with evidence for each use case |
| **Gap Analysis** | Data gaps, capability gaps, and architecture gaps with effort ratings |
| **Snowflake Value Proposition** | Feature-to-gap mapping — which Snowflake feature closes which gap |
| **Business Impact Assessment** | Industry benchmark ranges per use case |
| **3-Phase Roadmap** | Prioritized sequence: Quick Wins → Gap Closure → Strategic Investment |
| **Markdown study** | Full structured report saved to working directory |
| **HTML report** | Single-page app with sidebar nav, color-coded badges, and use case cards |
| **PPTX deck** | 22-slide Snowflake-branded presentation, one slide per use case |

## Installation

```bash
cortex skill add sfc-gh-michael-lemke/snowflake-value-map
```

Or install from the Snowflake Cortex Extension catalog (if your account has access):

```bash
cortex skill add snowflake-value-map --plugin-fqn USER$PMONTEIRO.SKILL_SHARING."snowflake-value-map"
```

## Usage

Type `/snowflake-value-map` in the Cortex Code chat panel.

The skill will ask:
1. **Data source** — live Snowflake account discovery (recommended) or DDL file / inline SQL
2. **Entity name** — the company or customer the study is for

Then it runs automatically. There are two confirmation checkpoints:
- After Step 1: confirm the deployment inventory and detected vertical
- After Step 4: validate findings before the final study is written

### Live account discovery (recommended)

```
/snowflake-value-map

> Analyze my account
> Database: ANALYTICS
> Schema: GOLD_LAYER
> Entity: Acme Corp
```

### DDL file input

```
/snowflake-value-map

> Provide DDL file or inline SQL
> Here's the file: /path/to/schema.sql
> Entity: Acme Corp
```

## Supported Verticals

The skill auto-detects the vertical using a weighted signal-scoring algorithm. It supports 12 pre-built catalogs:

| Vertical | Canonical Use Cases |
|----------|---------------------|
| Technology / SaaS | 10 |
| Financial Services | 10 |
| Healthcare | 10 |
| Insurance | 10 |
| Life Sciences & Pharma | 10 |
| Retail / CPG | 10 |
| Manufacturing | 10 |
| Energy & Utilities | 10 |
| Telecom | 10 |
| Logistics & Transportation | 10 |
| Media & Entertainment | 10 |
| Public Sector | 10 |

Unsupported verticals fall back to a best-effort analysis with a warning banner.

## How Classification Works

Each use case is scored against the deployment inventory:

| Classification | Meaning |
|----------------|---------|
| **READY** | Required data and patterns fully present — deployable now |
| **NEAR** | Most data exists; 1–2 gaps (missing joins or features to enable) |
| **GAP** | Fundamental data domain or architecture absent |
| **N/A** | Use case doesn't align with this entity's footprint |

**Fit Score formula:**
```
fit_score = (Σ impact_weight × classification_score) / Σ impact_weight × 10

Impact weights:  HIGH=3, MEDIUM=2, LOW=1
Classification scores: READY=1.0, NEAR=0.6, GAP=0.1, N/A=0
```

## Data Health Checks (live connection)

When analyzing a live Snowflake account, the skill checks row counts, freshness, and NULL density for each table. Tables are classified as:

- **HEALTHY** — sufficient rows, fresh data, low NULL density on key columns
- **STALE** — has rows but older than the staleness threshold for its table type
- **SPARSE** — below expected minimum row count
- **HOLLOW** — zero rows
- **DEGRADED** — key columns have >20% NULLs
- **MISLEADING** — sample data contradicts what the schema promises

HOLLOW and SPARSE tables are excluded from READY/NEAR classification. STALE tables downgrade readiness by one level.

## Output Files

All files are written to the current working directory (or `BASE_WORKING_DIR` if set):

```
<entity-slug>-value-map-study.md      # Markdown study (always generated)
<entity-slug>-value-map-study.html    # HTML report (if requested)
<entity-slug>-value-map.pptx          # PPTX deck (if requested)
```

## Example Output

The [`examples/`](examples/) directory contains a complete run against Snowflake's internal `MDM.MDM_INTERFACES` schema (71 tables + 69 views):

| File | Description |
|------|-------------|
| [`snowflake-mdm-value-map-study.md`](examples/snowflake-mdm-value-map-study.md) | Full Markdown study |
| [`snowflake-mdm-value-map-study.html`](examples/snowflake-mdm-value-map-study.html) | HTML report with sidebar navigation |
| [`snowflake-mdm-value-map.pptx`](examples/snowflake-mdm-value-map.pptx) | 22-slide PPTX deck |
| [`account_intelligence_360_semantic_view.yaml`](examples/account_intelligence_360_semantic_view.yaml) | Cortex Analyst semantic view built from the READY use cases |

**Results summary for that run:**
- **Vertical detected:** Technology / SaaS (HIGH confidence — SFDC, Workday, ServiceNow, GitHub integrations)
- **Canonical Fit Score:** 2.96 / 10 (expected for an MDM/operational hub schema)
- **READY (deployment-specific):** Third-Party Risk Dashboard, 360 Account Intelligence, RETL Observability, MDM Quality Monitoring
- **NEAR:** Customer Health Scoring, Revenue Intelligence, SE Pipeline Intelligence
- **GAP:** Product-Led Growth, Usage-Based Billing, Support Ticket Prediction, Trial Conversion

## Repository Structure

```
snowflake-value-map/
├── SKILL.md                          # Skill definition and full workflow
├── GUIDE.md                          # User guide and tips
├── README.md                         # This file
├── examples/
│   ├── snowflake-mdm-value-map-study.md
│   ├── snowflake-mdm-value-map-study.html
│   ├── snowflake-mdm-value-map.pptx
│   └── account_intelligence_360_semantic_view.yaml
└── references/
    ├── snowflake-capabilities.md     # Snowflake feature library
    ├── study-template.md             # Markdown output structure
    ├── study-template.html           # HTML design system (CSS + JS)
    ├── pptx-generation.md            # PPTX slide blueprint and Python helpers
    └── verticals/
        ├── technology-saas.md
        ├── financial-services.md
        ├── healthcare.md
        ├── insurance.md
        ├── life-sciences-pharma.md
        ├── retail-cpg.md
        ├── manufacturing.md
        ├── energy-utilities.md
        ├── telecom.md
        ├── logistics-transportation.md
        ├── media-entertainment.md
        └── public-sector.md
```

## Tips for Better Results

- **Use table and column comments.** The skill reads `COMMENT =` clauses from DDL and `INFORMATION_SCHEMA.COLUMNS`. Comments like `-- Source: Salesforce` or `COMMENT = 'grain: one row per customer per day'` improve both vertical detection accuracy and classification quality.
- **Include fact tables.** A schema of only reference/lookup tables will score lower. Including event, transaction, or fact tables enables more READY/NEAR classifications.
- **Use the real entity name.** The study personalizes the executive summary and PPTX to the entity name. "Acme Corp" produces a cleaner output than "My Organization".
- **Multiple schemas.** If your data is split (e.g., Silver in `MODELED`, Gold in `INTERFACES`), analyze both. The skill merges the inventory and classifies layers correctly.

## Important Notes

**Low fit score ≠ bad schema.** Canonical use cases for each vertical focus on analytics and AI workloads. An MDM hub, operational data store, or reverse ETL schema will score low on these because its value is powering other analytics layers. The skill surfaces this as "Deployment-Specific Strengths" outside the canonical catalog.

**GAP ≠ impossible.** GAP means the required data domain is absent from the assessed schema — not that the use case can't be built. The skill provides a specific path to close each gap.

**Impact figures are benchmarks, not guarantees.** All impact numbers are sourced from published analyst reports (Gartner, Forrester, McKinsey, vendor case studies) and prefixed with "Industry benchmarks suggest...". They are not calibrated to any specific entity's revenue or cost structure.

## Credits

Skill originally published by `USER$PMONTEIRO` to the Snowflake Cortex Extension catalog (`VERSION$2`). This repository contains the skill files with an added user guide (`GUIDE.md`), README, and example outputs from a production run.
