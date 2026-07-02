# My Organization (Snowflake Inc.) - Snowflake Value Map Study

**Vertical:** Technology / SaaS  
**Assessment Date:** July 2, 2026  
**Deployment Scope:** MDM.MDM_INTERFACES — 71 tables + 69 views (Gold interface layer)  
**Assessed By:** Snowflake Value Map Skill | Cortex Code

---

## Executive Summary

My Organization's MDM_INTERFACES schema is a mature, production-grade **Master Data Management hub** serving as the authoritative source for customer, vendor, employee, and party data across Snowflake's internal operational systems (Salesforce, Workday, ServiceNow, GitHub). The deployment is exceptionally strong for **Third-Party Risk Management**, **360 Account Intelligence**, and **MDM/Entity Resolution** — three use cases where it is genuinely best-in-class. Against the 10 canonical Technology/SaaS use cases (which skew toward product analytics and billing), the fit score is 2.96/10, reflecting that this schema is a **data operations backbone**, not a product telemetry layer. The three highest-priority deployment opportunities are: (1) activating an end-to-end Customer Health Score by connecting MDM account data to consumption/support layers, (2) building a Revenue Intelligence / EACV waterfall from the rich DIM_USE_CASE + FACT_USE_CASE_STAGE_MOVEMENT data already present, and (3) formalizing the Vendor Risk posture (Panorays + SecurityScorecard + SOE screening) into a consolidated Snowflake-native dashboard.

**Fit Score (Canonical Technology/SaaS): 2.96 / 10** — 0% deployable today, 39% near-term strategic value, 61% gap  
**Deployment-Specific Strengths: 4 READY, 1 NEAR** (outside canonical catalog — see Section 3.5)

| Category | Count | Weighted Value % | Priority Use Cases |
|----------|-------|-----------------|---------------------|
| READY (deploy now) | 0 canonical / **4 deployment-specific** | 0% canonical | MDM/Entity Resolution, 360 Account Intelligence, Third-Party Risk, Reverse ETL |
| NEAR (minor gaps) | 4 canonical + 1 deployment-specific | 39% | Customer Health, Revenue Intelligence, Feature Adoption, Partner Ecosystem |
| GAP (significant) | 6 canonical | 61% | Product-Led Growth, Billing/Metering, Trial Conversion, Support Prediction, Cost Attribution, Perf Analytics |
| N/A | 0 | — | — |

---

## 1. Entity Strategic Goals (Inferred)

Based on data investment patterns observed in the deployment:

| # | Strategic Goal | Evidence from Deployment | Confidence |
|---|----------------|--------------------------|------------|
| 1 | **Unified party identity across all GTM systems** | ADM_GLOBAL_PARTY_SNAPSHOT (3.7B rows, SCD Type 2), SNOWMESH_ENTITY_HIERARCHY_FLAT (1.8M), ULTIMATE_PARTY_MAP — massive investment in entity resolution across SFDC, Workday, ServiceNow, D&B | High |
| 2 | **Third-party and vendor risk at scale** | TPCS_SOE_SCREENING_RESULTS (32K), SECURITY_SCORECARD_HISTORY, PANORAYS_HISTORY, LEGAL_ENTITY_DNB_SOURCE (324K DUNS), TPCS_SANCTIONS_PORTFOLIO_ASSESSMENT_RESULTS — end-to-end vendor compliance pipeline | High |
| 3 | **Real-time enrichment of SFDC account data** | CRUNCHBASE_MAP_TO_ACCOUNT_SFDC (696K), FACTSET_MAP_TO_ACCOUNT_SFDC (49.5M), SFDC_ACCOUNT_CONSOLIDATED_INDUSTRY (360K), SAI_PILOT_TECH_STACK (23K), 15+ reverse ETL interfaces to SFDC | High |
| 4 | **Sales pipeline intelligence by use case** | DIM_USE_CASE (97.7K with EACV), FACT_USE_CASE_STAGE_MOVEMENT (252K stage transitions), SE assignment tracking, MOVEIN_USE_CASE_EACV — Snowflake-specific GTM analytics | Medium |
| 5 | **Engineering productivity visibility** | GITHUB_PULL_REQUEST (4.2M PRs), GITHUB_METRIC (daily per-employee metrics), DIM_EMPLOYEE (Workday + GitHub crosswalk) — R&D analytics for a software company | Medium |

---

## 2. Current Deployment Assessment

### 2.1 Gold Layer Inventory (MDM_INTERFACES — interface/output layer)

| Object | Domain | Table Type | Grain | Key Columns | Rows | Health | Notes |
|--------|--------|-----------|-------|-------------|------|--------|-------|
| ADM_GLOBAL_PARTY_SNAPSHOT | Party Universe | DIMENSION (SCD2) | 1 row per party per validity period | GLOBAL_PARTY_ID, DBT_VALID_FROM/TO | 3.7B | HEALTHY | Backbone of all entity resolution; last updated 2026-06-29 |
| SNOWMESH_ENTITY_HIERARCHY_FLAT | Party Hierarchy | REFERENCE | 1 row per entity-ancestor pair | PARTY_ID, parent chain | 1.84M | HEALTHY | Powers ultimate parent rollups |
| SNOWMESH_ULTIMATE_PARTY | Party | DIMENSION | 1 row per ultimate party | PARTY_ID | 1.6K | SPARSE | Reference table, expected small |
| SNOWMESH_COMPANY_CUSTOMER | Customer | DIMENSION | 1 row per customer company | ACCOUNT_ID, CONTRACT_END_DATE_MIN | 27.3K | HEALTHY | |
| SNOWMESH_COMPANY_VENDOR | Vendor | DIMENSION | 1 row per vendor company | PARTY_ID | 6.5K | HEALTHY | |
| CUSTOMER_PARTY_ACCOUNT_MAP_SFDC | Customer-CRM | REFERENCE | 1 row per customer-SFDC mapping | PARTY_ID, SFDC_ACCOUNT_ID, CUSTOMER_ID_WD | 20.1K | HEALTHY | Bridges MDM party to SFDC + Workday |
| VENDOR_MASTER_MAP | Vendor | DIMENSION | 1 row per vendor with all system IDs | PARTY_ID, VENDOR_ID_SN, VENDOR_ID_WD, partner_flag | 29.9K | HEALTHY | Cross-system golden record |
| VENDOR_MASTER_HIERARCHY | Vendor | REFERENCE | 1 row per hierarchy node | PARTY_ID, parent | 6.8K | HEALTHY | |
| VENDOR_MASTER_CONTRACT | Vendor-Legal | REFERENCE | 1 row per vendor contract | CONTRACT_ID | 19.1K | HEALTHY | |
| DIM_EMPLOYEE (view) | Employee | DIMENSION | 1 row per employee | EMPLOYEE_ID, WD_ID, GITHUB_LOGIN | 19.9K | HEALTHY | Workday+GitHub crosswalk |
| DIM_USE_CASE (view) | Sales Pipeline | FACT | 1 row per Salesforce use case | USE_CASE_ID, EACV, STAGE, SE_ID | 97.7K | HEALTHY | Source of truth for Snowflake GTM |
| FACT_USE_CASE_STAGE_MOVEMENT (view) | Sales Pipeline | FACT | 1 row per use case per stage entry | USE_CASE_ID, MOVEIN_STAGE_NUM, MOVEIN_DATE, MOVEIN_USE_CASE_EACV | 252K | HEALTHY | Full pipeline waterfall history |
| CRUNCHBASE_MAP_TO_ACCOUNT_SFDC | Customer Enrichment | REFERENCE | 1 row per SFDC account Crunchbase match | ACCOUNT_ID, ACCOUNT_ID_CRUNCHBASE, ACCOUNT_TYPE | 695.8K | HEALTHY | |
| FACTSET_MAP_TO_ACCOUNT_SFDC | Customer Financial Enrichment | REFERENCE | 1 row per SFDC account FactSet match | ACCOUNT_ID, FACTSET_ID | 49.5M | HEALTHY | Massive financial data linkage |
| LEGAL_ENTITY_DNB_SOURCE | Identity/Enrichment | REFERENCE | 1 row per DUNS-enriched entity | DUNS_EID, TAX_EID, LEGAL_ADDRESS | 324.4K | HEALTHY | D&B authoritative identity |
| PRODUCT_TERMS_ACCEPTANCE_ORGANIZATION | Product/Legal | AGGREGATE | 1 row per org per product terms | ORGANIZATION_ID, TERMS_TYPE | 67.1K | HEALTHY | Dynamic table, change-tracked |
| PRODUCT_DATA_SHARING_TERMS_ORGANIZATION | Data Governance | REFERENCE | 1 row per org data-sharing terms | ORGANIZATION_ID | 56.8K | HEALTHY | |
| PRODUCT_MIN_MAX_EXCLUDE_ORGANIZATION | Product Access | REFERENCE | 1 row per org with billing flag | ORGANIZATION_ID, ORGANIZATION_DPO_FLAG | 67K | HEALTHY | Controls BLOCK_MIN_MAX_ACCESS |
| TPCS_SOE_SCREENING_RESULTS | Compliance | AGGREGATE | 1 row per entity SOE screening | PARTY_ID, SOE_STATUS_FINAL, COUNTRY | 32.5K | HEALTHY | SOE = State-Owned Enterprise check |
| SECURITY_SCORECARD_HISTORY (view) | Vendor Security | FACT (SCD2) | 1 row per vendor per score period | PARENT_DOMAIN, TOTAL_SCORE, DBT_VALID_FROM/TO | varies | HEALTHY | |
| PANORAYS_HISTORY (view) | Vendor Security | FACT (SCD2) | 1 row per vendor per score period | PANORAYS_COMPANY_ID, PANORAYS_RATING_FINAL, DBT_VALID_FROM/TO | varies | HEALTHY | |
| GITHUB_PULL_REQUEST | Engineering | FACT | 1 row per GitHub PR | PR_ID, AUTHOR, STATUS, CREATED_AT | 4.2M | HEALTHY | |
| GITHUB_METRIC | Engineering | AGGREGATE | Daily metrics per employee | EMPLOYEE_ID, DATE, activity counts | 21K | HEALTHY | |
| SAI_PILOT_TECH_STACK | Customer Intelligence | REFERENCE | 1 row per org tech item | ACCOUNT_ID, TECH_PRODUCT | 23K | HEALTHY | Tech stack intelligence per account |
| SFDC_ACCOUNT_CONSOLIDATED_INDUSTRY | Customer Firmographic | AGGREGATE | 1 row per SFDC account | ACCOUNT_ID, INDUSTRY | 359.6K | HEALTHY | MDM-conformed industry classification |
| SFDC_ACCOUNT_CONSOLIDATED_EMPLOYEE_COUNT | Customer Firmographic | AGGREGATE | 1 row per SFDC account | ACCOUNT_ID, EMPLOYEE_COUNT | 359.6K | HEALTHY | |
| LEGAL_FEATURE_SECURITY_QUESTIONS | Customer Compliance | REFERENCE | 1 row per org × security question | ORGANIZATION_ID, question/answer pairs | 861 | SPARSE | Contract-level security posture |
| SNOWMESH_DATA | Override/Ops | AGGREGATE | 1 row per overrideable record | RECORD_ID | 1.48M | HEALTHY | Interface to Elementum Overrides App |
| SNOWMESH_EXCEPTIONS | Override/Ops | FACT | 1 row per exception to resolve | RECORD_ID | 19.5K | HEALTHY | |
| FORBES_GLOBAL_2000 (view) | Customer Intelligence | REFERENCE | 1 row per Forbes G2000 company | COMPANY_ID | varies | HEALTHY | Account targeting/ICP enrichment |
| VENDOR_MASTER_MAP_WD_TO_SN | Vendor | REFERENCE | 1 row per WD-SN vendor mapping | VENDOR_ID_WD, VENDOR_ID_SN | 3.2K | HEALTHY | |
| ULTIMATE_PARTY_MAP | Party | REFERENCE | 1 row per party to ultimate party | PARTY_ID, ULTIMATE_PARTY_ID | 27.2K | HEALTHY | |

### 2.2 Reverse ETL (Output) Interfaces

| Interface | Target System | Purpose | Rows |
|-----------|--------------|---------|------|
| RETL_SDFC_ENRICHMENT_INDUSTRY | Salesforce | Industry enrichment push | 32.5K |
| RETL_SDFC_ENRICHMENT_EMPLOYEE_COUNT | Salesforce | Employee count push | 33 |
| RETL_SDFC_ENRICHMENT_REVENUE | Salesforce | Revenue enrichment push | 28.4K |
| RETL_SFDC_SOE_STATUS_SYNC | Salesforce | SOE compliance status | 32.5K |
| RETL_SFDC_GPU_ENRICHMENT_* (views) | Salesforce | GPU enrichment for accounts | varies |
| RETL_SFDC_SAI_TECH_STACK (view) | Salesforce | Tech stack sync | varies |
| RETL_SFDC_TPCS_PD_* (views) | Salesforce | Third-party procurement compliance | varies |
| RETL_SERVICENOW_ADDRESS_SYNC | ServiceNow | Vendor address master sync | 3.2K |
| RETL_SERVICENOW_WORKDAYID_SYNC | ServiceNow | WD-SN ID linkage | 84 |
| RETL_SERVICENOW_WORKDAYSERVICESTATUS | ServiceNow | Service status sync | 9.5K |

### 2.3 Data Domain Coverage

| Domain | Gold Objects | Completeness | Gaps |
|--------|-------------|--------------|------|
| Customer / Party Master | 8 tables + 5 views | HIGH | No consumption/usage data |
| Vendor / Supplier | 10 tables | HIGH | No spend/PO data |
| Employee / HR | 2 tables + 2 views | MEDIUM | No performance review or comp data |
| Sales Pipeline / GTM | 2 views (DIM_USE_CASE, FACT_USE_CASE_STAGE_MOVEMENT) | HIGH for pipeline; LOW for post-close | No deployment/consumption post-close |
| External Enrichment | 5 tables (Crunchbase, FactSet, D&B, S&P, Forbes) | HIGH | No NPS/survey, no support ticket data |
| Legal / Compliance | 4 tables + 3 views | HIGH | No DPA/GDPR consent tracking |
| Vendor Security | 2 views (SCS, Panorays) | HIGH | No continuous monitoring alerts |
| Engineering Activity | 2 tables + 1 view | MEDIUM | No CI/CD, deployment frequency, or infra cost data |
| Product Adoption | 3 tables | LOW | Terms acceptance only; no feature usage events |

---

## 3. Use Case Fit Matrix

### 3.1 Summary Matrix (Canonical Technology/SaaS)

| # | Use Case | Impact | Classification | Rationale |
|---|----------|--------|----------------|-----------|
| 1 | Product-Led Growth Analytics | HIGH | **GAP** | No product event/telemetry or activation data |
| 2 | Customer Health Scoring & Churn Prediction | HIGH | **NEAR** | Strong account master + contracts + use case EACV; missing consumption, NPS, support |
| 3 | Usage-Based Billing & Metering | HIGH | **GAP** | Has product terms/access controls; no meter events, invoices, or credit burn-down |
| 4 | Feature Adoption & Engagement | MEDIUM | **NEAR** | Feature-level org data (legal/product terms); no product event stream |
| 5 | Revenue Intelligence (ARR/MRR Analytics) | HIGH | **NEAR** | DIM_USE_CASE + FACT_USE_CASE_STAGE_MOVEMENT provide full EACV waterfall; missing net revenue retention / actual payments |
| 6 | Infrastructure Cost Attribution | MEDIUM | **GAP** | No cloud billing, resource tagging, or cost center data |
| 7 | Support Ticket & Escalation Prediction | MEDIUM | **GAP** | ServiceNow integration exists for vendor HR; no customer support ticket data |
| 8 | Trial-to-Paid Conversion Optimization | HIGH | **GAP** | PARTY_CREATION_FUNNEL exists (20 rows, reference); no trial behavior data |
| 9 | Multi-Tenant Performance Analytics | LOW | **GAP** | O11Y tables are MDM data quality observability, not platform performance metrics |
| 10 | Partner & Marketplace Ecosystem Analytics | LOW | **NEAR** | VENDOR_MASTER_MAP has partner_flag; SNOWMESH_COMPANY_VENDOR; RETL third-party approval flows |

**Canonical Fit Score: 2.96 / 10**  
`fit_score = (Σ impact_weight × classification_score) / Σ impact_weight × 10`  
= (1.8 + 1.2 + 1.8 + 0.6 + 0.3×4 + 0.2×2 + 0.1) / 23 × 10 ≈ **2.96**

> Note: Low canonical fit score is expected for an MDM operations schema. The schema's value lies in powering *other* analytics layers, not in direct product telemetry.

---

### 3.2 READY Use Cases — Canonical: None / Deployment-Specific: 4

#### [DS-1] Master Data Management & Entity Resolution

- **Supporting Objects:** ADM_GLOBAL_PARTY_SNAPSHOT (3.7B rows, SCD2), SNOWMESH_ENTITY_HIERARCHY_FLAT (1.84M), SNOWMESH_ULTIMATE_PARTY, SNOWMESH_NAME_MATCH_RESULTS, ULTIMATE_PARTY_MAP, SNOWMESH_EXCEPTIONS
- **Why Ready:** This IS the MDM system. The platform performs entity resolution, hierarchy management, deduplication, and golden record creation across SFDC, Workday, ServiceNow, D&B, Crunchbase, FactSet, and S&P. Snowflake is the system of record for party identity.
- **Recommended Implementation:** Expose SNOWMESH_DATA + SNOWMESH_EXCEPTIONS via Cortex Analyst semantic view for real-time override decision-making; build Dynamic Table-based freshness monitoring on ADM_GLOBAL_PARTY_SNAPSHOT using Alerts for data quality SLAs.
- **Snowflake Features:** Dynamic Tables (incremental party resolution), Alerts (data quality thresholds), Cortex Analyst (natural language MDM queries), Data Sharing (distribute golden records to downstream teams)
- **Industry Benchmark:** MDM platforms reduce duplicate records by 20-40%; data governance programs cut downstream analytics rework by 30-50%

#### [DS-2] 360 Account Intelligence with Multi-Source Enrichment

- **Supporting Objects:** CRUNCHBASE_MAP_TO_ACCOUNT_SFDC (696K), FACTSET_MAP_TO_ACCOUNT_SFDC (49.5M), LEGAL_ENTITY_DNB_SOURCE (324K), SFDC_ACCOUNT_CONSOLIDATED_INDUSTRY (360K), SFDC_ACCOUNT_CONSOLIDATED_EMPLOYEE_COUNT (360K), SAI_PILOT_TECH_STACK (23K), FORBES_GLOBAL_2000
- **Why Ready:** Six external enrichment sources are joined to SFDC accounts with live RETL pushback — firmographic, financial, tech stack, employee count, industry, and investor data are all present and refreshed.
- **Recommended Implementation:** Build a Cortex Analyst semantic layer over the enrichment tables for AE/SDR natural language queries ("show me all accounts with tech stack using Databricks and over 1000 employees"); add Snowflake Cortex AI for anomaly flagging on enrichment drift.
- **Snowflake Features:** Cortex Analyst (NL query over account intelligence), Dynamic Tables (live enrichment aggregates), Cortex AI (firmographic anomaly detection)
- **Industry Benchmark:** Account enrichment programs improve outbound pipeline conversion by 15-25%; tech stack intelligence reduces competitive sales cycle by 10-20%

#### [DS-3] Third-Party Risk Management & Vendor Compliance

- **Supporting Objects:** SECURITY_SCORECARD_HISTORY (SCD2), PANORAYS_HISTORY (SCD2), TPCS_SOE_SCREENING_RESULTS (32.5K), TPCS_SANCTIONS_PORTFOLIO_ASSESSMENT_RESULTS, LEGAL_ENTITY_DNB_SOURCE, VENDOR_MASTER_MAP, LEGAL_FEATURE_SECURITY_QUESTIONS
- **Why Ready:** SecurityScorecard + Panorays provide historical security scoring (SCD2 — full trend history); SOE/sanctions screening covers 32.5K entities; D&B DUNS provides identity anchoring; all linked to the party master.
- **Recommended Implementation:** Build a Snowflake-native vendor risk dashboard using Dynamic Tables to flag score drops week-over-week; create Cortex AI–powered risk narrative summaries per vendor; use Alerts for SOE status changes or SecurityScorecard score drops below threshold.
- **Snowflake Features:** Dynamic Tables (rolling risk deltas), Alerts (score threshold breaches), Cortex AI (risk narrative summaries, AI_COMPLETE), Data Sharing (publish vendor risk posture to procurement teams)
- **Industry Benchmark:** Proactive vendor risk monitoring reduces third-party breach exposure costs by 20-35%; automated SOE screening reduces manual review time by 60-80%

#### [DS-4] Reverse ETL & Operational Data Sync

- **Supporting Objects:** 15+ RETL_* tables/views to SFDC, ServiceNow, and Workday
- **Why Ready:** A complete operational sync layer already exists. Snowflake is already the master pushing enriched data back to operational systems.
- **Recommended Implementation:** Migrate RETL interfaces to Snowflake Tasks + Dynamic Tables for declarative, observable pipelines; add error tables on RETL outputs for automated failure detection; expose RETL sync status in a Snowsight dashboard.
- **Snowflake Features:** Tasks (scheduled reverse ETL), Dynamic Tables (declarative RETL pipelines), Error Tables (fault detection), Alerts (sync failure notifications)
- **Industry Benchmark:** Operational data synchronization failures cost $100K-$500K/yr in manual remediation; automated monitoring reduces incident response time by 50-70%

---

### 3.3 NEAR Use Cases (Minor Gaps)

#### [UC-2] Customer Health Scoring & Churn Prediction

- **Supporting Objects:** ADM_GLOBAL_PARTY_SNAPSHOT, SNOWMESH_COMPANY_CUSTOMER, CUSTOMER_PARTY_ACCOUNT_MAP_SFDC, DIM_USE_CASE (use case EACV/stage), PRODUCT_TERMS_ACCEPTANCE_ORGANIZATION, FACTSET_MAP_TO_ACCOUNT_SFDC (financial signals), CRUNCHBASE_MAP_TO_ACCOUNT_SFDC
- **Gaps Identified:**
  - [ ] **Missing: consumption/usage metrics** — Snowflake platform consumption data (warehouse credits, query volume) not in MDM_INTERFACES; needs join to Snowhouse or ACCOUNT_USAGE views
  - [ ] **Missing: NPS/CSAT data** — no customer survey or support sentiment data present
  - [ ] **Missing: support ticket volume** — ServiceNow integration is vendor-only; customer support tickets not exposed
  - [ ] **Missing: champion contact tracking** — DIM_USE_CASE has SE assignment but no executive sponsor/champion field
- **Effort to Close:** Medium — consumption data join is the critical path; rest is additive
- **Snowflake Features:** Cortex ML (health score classification), Dynamic Tables (rolling health computation), Alerts (churn risk threshold), Snowpark (feature engineering for multi-source health model)
- **Industry Benchmark:** 20-40% reduction in gross churn; 5-15 point improvement in net dollar retention with proactive health monitoring

#### [UC-4] Feature Adoption & Engagement

- **Supporting Objects:** PRODUCT_TERMS_ACCEPTANCE_ORGANIZATION (67K — features/terms accepted per org), PRODUCT_DATA_SHARING_TERMS_ORGANIZATION, PRODUCT_MIN_MAX_EXCLUDE_ORGANIZATION, LEGAL_FEATURE_SECURITY_QUESTIONS, LEGAL_FEATURE_EPIC_THREEWAY, LEGAL_FEATURE_LOGO_USAGE
- **Gaps Identified:**
  - [ ] **Missing: product event stream** — terms acceptance ≠ feature usage; need actual platform telemetry (feature clicks, API calls per feature)
  - [ ] **Missing: feature release dates** — no product changelog/release table to anchor adoption curves
  - [ ] **Missing: user-level adoption** — current data is org-level; need user-level feature events for activation analysis
- **Effort to Close:** Medium-High — requires connecting to platform telemetry layer
- **Snowflake Features:** Cortex AI (feedback analysis from Gong/GONG_USER_HISTORY), Dynamic Tables (adoption dashboards), Cortex ML (clustering by feature engagement)
- **Industry Benchmark:** Data-driven feature prioritization improves feature ROI by 20-40%

#### [UC-5] Revenue Intelligence (ARR/MRR Analytics)

- **Supporting Objects:** DIM_USE_CASE (97.7K — EACV, stage, cloud provider, SE assignment), FACT_USE_CASE_STAGE_MOVEMENT (252K — full waterfall history with MOVEIN_USE_CASE_EACV), AE_CONTRACT_MAPPING_SFDC, AE_CONTRACT_MAPPING_WD, VENDOR_MASTER_CONTRACT, SNOWMESH_COMPANY_CUSTOMER (CONTRACT_END_DATE_MIN)
- **Gaps Identified:**
  - [ ] **Missing: actual invoiced/paid ARR** — EACV is estimated; need billing/invoice data to compute actual net dollar retention
  - [ ] **Missing: expansion/contraction decomposition** — stage movement captures pipeline flow, not closed-won to consumption ramp
  - [ ] **Missing: cohort tagging** — no explicit customer cohort or segment dimension on use cases
- **Effort to Close:** Low-Medium — EACV waterfall analysis deployable today; billing join is additive for full ARR bridge
- **Snowflake Features:** Dynamic Tables (live ARR tracking), Snowpark (cohort waterfall models), Cortex ML (revenue forecasting), Data Sharing (board reporting packages)
- **Industry Benchmark:** Real-time ARR visibility vs. monthly manual; 10-20% improvement in forecast accuracy

#### [UC-10] Partner & Marketplace Ecosystem Analytics

- **Supporting Objects:** VENDOR_MASTER_MAP (partner_flag column), SNOWMESH_COMPANY_VENDOR (6.5K vendors including partners), RETL_SFDC_TPCS_PD_THIRDPARTY_APPROVED/REJECTED_SYNC, CRUNCHBASE_ORGANIZATIONS (partner firmographics)
- **Gaps Identified:**
  - [ ] **Missing: co-sell pipeline linkage** — partner_flag exists on vendor master but no explicit co-sell opportunity tracking
  - [ ] **Missing: integration install data** — no API call volumes or partner integration health metrics
  - [ ] **Missing: partner tier/revenue share** — no partner tier classification or revenue attribution
- **Effort to Close:** Medium
- **Snowflake Features:** Data Sharing (partner portals), Dynamic Tables (partner contribution dashboards), Snowpark (co-sell attribution)
- **Industry Benchmark:** 10-25% of SaaS revenue attributable to ecosystem; 15-30% lower churn for customers using 2+ integrations

#### [DS-5] Sales Engineering Pipeline Intelligence (Deployment-Specific NEAR)

- **Supporting Objects:** DIM_USE_CASE (USE_CASE_EACV, USE_CASE_STAGE, CLOUD_PROVIDER, MOVEIN_USE_CASE_LEAD_SE_ID, technical use case tags), FACT_USE_CASE_STAGE_MOVEMENT (252K stage movements with SE assignments, decision dates, go-live dates)
- **Gaps Identified:**
  - [ ] **Missing: actual deployment/consumption post-close** — pipeline stages end at "Deployed" but no consumption data follows
  - [ ] **Missing: SE-to-win correlation** — SE performance vs. win rate requires closed-won flag + deal attribution
  - [ ] **Missing: technical use case tag taxonomy** — tag quality/completeness needs audit
- **Effort to Close:** Low — most data is present; needs Cortex Analyst semantic view + SE performance dashboard
- **Snowflake Features:** Cortex Analyst (NL queries over pipeline), Dynamic Tables (SE performance dashboards), Cortex ML (win probability scoring)
- **Industry Benchmark:** SE-assisted deals close at 2-3x higher rates; pipeline visibility reduces forecast error by 15-25%

---

### 3.4 GAP Use Cases (Significant Work Required)

#### [UC-1] Product-Led Growth Analytics

- **Missing Data Domains:** Product event stream, activation milestones, self-serve signup funnel, experiment assignments, viral/referral loops
- **Missing Capabilities:** Real-time event ingestion, funnel analysis tables, cohort experiment attribution
- **Path to Deployment:**
  1. Ingest Snowflake platform product events via Snowpipe Streaming into a new PRODUCT_EVENT table
  2. Define activation milestones in a PRODUCT_MILESTONE dimension table
  3. Build Dynamic Table–based funnel from event → activation → conversion
  4. Layer Cortex ML propensity scoring on funnel cohorts
- **Effort:** High
- **Marketplace Opportunities:** Product analytics benchmark data (OpenView SaaS metrics, Bessemer State of the Cloud)
- **Strategic Value:** 15-30% improvement in activation rates; critical for Snowflake's own PLG motion

#### [UC-3] Usage-Based Billing & Metering

- **Missing Data Domains:** Tenant-level meter events, plan/tier dimensions, usage aggregates by billing period, invoice/payment records, credit commitment burn-down
- **Path to Deployment:**
  1. Expose Snowflake ACCOUNT_USAGE.METERING_HISTORY or billing API data into MDM layer
  2. Build USAGE_AGGREGATE Dynamic Table at org/billing period grain
  3. Join to PRODUCT_TERMS_ACCEPTANCE_ORGANIZATION for plan entitlements
  4. Trigger Alerts for commitment threshold warnings
- **Effort:** High
- **Marketplace Opportunities:** None needed — data is internal
- **Strategic Value:** 99.9%+ billing accuracy; 10-20% revenue uplift through usage transparency

#### [UC-6] Infrastructure Cost Attribution

- **Missing Data Domains:** Cloud billing data (AWS/GCP/Azure), tenant-to-resource mapping, cost center allocation, budget tracking
- **Path:** Connect Snowflake ACCOUNT_USAGE + cloud billing exports → tenant allocation model → Dynamic Table dashboards
- **Effort:** High
- **Marketplace Opportunities:** Cloud cost benchmark datasets

#### [UC-7] Support Ticket & Escalation Prediction

- **Missing Data Domains:** Customer support ticket metadata, SLA tracking, resolution history, CSAT scores, escalation history
- **Path:** Expose Salesforce Service Cloud or Zendesk ticket data → join to CUSTOMER_PARTY_ACCOUNT_MAP_SFDC → Cortex ML escalation classifier
- **Effort:** High
- **Marketplace Opportunities:** None

#### [UC-8] Trial-to-Paid Conversion Optimization

- **Missing Data Domains:** Trial behavior events, feature usage during trial, conversion timestamps, plan selection, A/B experiment assignments
- **Path:** Ingest trial telemetry → join to PARTY_CREATION_FUNNEL → Cortex ML conversion model
- **Effort:** High
- **Marketplace Opportunities:** None

#### [UC-9] Multi-Tenant Performance Analytics

- **Missing Data Domains:** Per-tenant request latency, error rates, resource consumption, SLA compliance records
- **Path:** Expose Snowflake ACCOUNT_USAGE.QUERY_HISTORY at tenant grain → Dynamic Table SLA monitors → Alerts
- **Effort:** Medium (data is accessible; mostly an architecture/permissions question)
- **Marketplace Opportunities:** None

---

## 4. Gap Analysis

### 4.1 Data Gaps

| Gap ID | Description | Use Cases Blocked | Resolution Path | Effort |
|--------|-------------|-------------------|-----------------|--------|
| DG-1 | Product telemetry / event stream absent | UC-1, UC-4, UC-8 | Snowpipe Streaming from platform | High |
| DG-2 | Consumption/usage metrics not in MDM | UC-2, UC-3, UC-9 | Join to Snowhouse ACCOUNT_USAGE | Medium |
| DG-3 | Customer NPS/CSAT/support tickets missing | UC-2, UC-7 | Ingest Salesforce Service Cloud or Zendesk | High |
| DG-4 | Billing/invoice/payment records absent | UC-3, UC-5 | Expose billing API data | High |
| DG-5 | Cloud infrastructure cost data absent | UC-6 | Cloud billing exports + resource tagging | High |
| DG-6 | Champion/executive sponsor contact tracking | UC-2 | Enrich DIM_USE_CASE from SFDC contact roles | Low |

### 4.2 Capability Gaps

| Gap ID | Description | Use Cases Blocked | Snowflake Solution | Effort |
|--------|-------------|-------------------|--------------------|--------|
| CG-1 | No real-time event ingestion pipeline | UC-1, UC-3 | Snowpipe Streaming | Medium |
| CG-2 | No health score computation layer | UC-2 | Cortex ML + Dynamic Tables | Medium |
| CG-3 | No semantic layer for NL querying | DS-2, DS-5 | Cortex Analyst semantic view | Low |
| CG-4 | No alerting on vendor risk score drops | DS-3 | Snowflake Alerts | Low |
| CG-5 | No RETL pipeline observability | DS-4 | Error Tables + Alerts | Low |

---

## 5. Snowflake Value Proposition

### 5.1 Feature-to-Gap Mapping

| Snowflake Feature | Gaps Addressed | Use Cases Enabled | Notes |
|-------------------|---------------|-------------------|-------|
| **Cortex Analyst** | CG-3 | DS-2, DS-5, DS-1 | NL queries over MDM enrichment tables and pipeline data |
| **Dynamic Tables** | CG-2, CG-5 | UC-2, UC-5, DS-3, DS-4 | Incremental health scores, RETL pipelines, risk deltas |
| **Cortex ML (Classification/Forecasting)** | CG-2, DG-2 | UC-2, UC-5, DS-5 | Churn risk, win probability, revenue forecasting |
| **Cortex AI (AI_COMPLETE)** | — | DS-3 | Vendor risk narrative summaries per entity |
| **Alerts** | CG-4, CG-5 | DS-3, DS-4, UC-2 | Score threshold triggers, RETL failure detection, health alerts |
| **Snowpipe Streaming** | CG-1 | UC-1, UC-3 | Real-time event and meter ingestion |
| **Error Tables** | CG-5 | DS-4 | Automated RETL fault detection |
| **Data Sharing** | — | DS-2, DS-3 | Share golden records and vendor risk posture to downstream teams |
| **Snowpark** | DG-2 | UC-2, UC-5 | Feature engineering for multi-source health model |

---

## 6. Business Impact Assessment

> ⚠ **All impact estimates use published industry benchmarks. Not calibrated to Snowflake Inc.'s revenue, cost structure, or operational maturity. Treat as directional.**

### 6.1 Impact Summary

| Use Case | Impact Category | Industry Benchmark Range | Calibration Data Needed |
|----------|----------------|--------------------------|------------------------|
| DS-2: 360 Account Intelligence | Revenue Growth | 15-25% outbound conversion improvement | Current outbound conversion baseline |
| DS-3: Third-Party Risk Mgmt | Risk Mitigation | 20-35% reduction in third-party breach exposure | Current vendor risk incident cost |
| DS-4: Reverse ETL Ops | Operational Efficiency | 50-70% reduction in sync incident response time | Current RETL failure rate and resolution time |
| UC-2: Customer Health | Revenue Retention | 20-40% reduction in gross churn | Current gross/net churn rate |
| UC-5: Revenue Intelligence | Revenue Predictability | 10-20% forecast accuracy improvement | Current forecast error rate |
| DS-5: SE Pipeline Intelligence | Revenue Growth | 15-25% reduction in forecast error | Current pipeline forecast MAPE |
| DS-1: MDM/Entity Resolution | Operational Efficiency | 20-40% reduction in duplicate records, 30-50% less rework | Current duplicate rate in SFDC |

### 6.2 Aggregate Impact (Directional)

| Category | Benchmark Range | Use Cases Contributing |
|----------|-----------------|----------------------|
| Revenue Growth | 15-30% improvement in conversion/pipeline metrics | DS-2, DS-5, UC-5 |
| Cost Reduction | 30-50% reduction in data quality rework | DS-1, DS-4 |
| Risk Mitigation | 20-35% reduction in third-party breach exposure | DS-3 |
| Operational Efficiency | 50-70% faster incident resolution | DS-3, DS-4 |

---

## 7. Recommended Roadmap

### Phase 1: Quick Wins — Activate Existing Strengths (READY)

| # | Use Case | Key Actions | Dependencies | Expected Impact |
|---|----------|-------------|--------------|-----------------|
| 1 | DS-3: Third-Party Risk Dashboard | Build Dynamic Table for weekly SecurityScorecard + Panorays deltas; create Alerts for score drops; Cortex AI vendor risk summaries | None | Proactive risk posture, 20-35% breach exposure reduction |
| 2 | DS-2: 360 Account Intelligence Semantic View | Build Cortex Analyst semantic view over Crunchbase + FactSet + industry + tech stack tables | None | AE/SDR NL querying over enriched accounts |
| 3 | DS-4: RETL Pipeline Observability | Add Error Tables and Alerts to RETL sync interfaces; build Snowsight RETL health dashboard | None | 50-70% faster RETL incident resolution |
| 4 | DS-1: MDM Data Quality Monitoring | Dynamic Table–based SLA monitoring on ADM_GLOBAL_PARTY_SNAPSHOT freshness; Alerts on SNOWMESH_EXCEPTIONS growth | None | Data quality SLA enforcement |

### Phase 2: Near-Term — Close Key Gaps (NEAR)

| # | Use Case | Gaps to Close | Key Actions | Expected Impact |
|---|----------|---------------|-------------|-----------------|
| 1 | UC-5: Revenue Intelligence | DG-4 (billing join), cohort tags | Build EACV waterfall in Snowpark; join AE_CONTRACT_MAPPING_SFDC; Dynamic Table ARR bridge | 10-20% forecast accuracy improvement |
| 2 | DS-5: SE Pipeline Intelligence | SE attribution model | Cortex Analyst semantic view over DIM_USE_CASE + FACT_USE_CASE_STAGE_MOVEMENT; SE win-rate model | 15-25% pipeline forecast improvement |
| 3 | UC-2: Customer Health Score | DG-2 (consumption join), DG-3 (NPS) | Join Snowhouse ACCOUNT_USAGE to CUSTOMER_PARTY_ACCOUNT_MAP_SFDC; Cortex ML health classifier | 20-40% churn reduction potential |
| 4 | UC-10: Partner Ecosystem | Co-sell linkage | Enrich VENDOR_MASTER_MAP partner tier; build co-sell attribution model | Partner revenue visibility |

### Phase 3: Strategic — Build Missing Layers (GAP)

| # | Use Case | Major Investments | Key Actions | Expected Impact |
|---|----------|-------------------|-------------|-----------------|
| 1 | UC-3: Usage-Based Billing | Billing API data pipeline | Connect ACCOUNT_USAGE.METERING_HISTORY; join to product terms | 10-20% revenue uplift from usage transparency |
| 2 | UC-1: Product-Led Growth | Product telemetry ingest | Snowpipe Streaming from platform events; activation milestone tables | 15-30% activation rate improvement |
| 3 | UC-7: Support Prediction | Customer support ticket data | Ingest Salesforce Service Cloud; join to CUSTOMER_PARTY_ACCOUNT_MAP_SFDC | 20-35% escalation reduction |
| 4 | UC-9: Multi-Tenant Perf Analytics | Query/compute data at tenant grain | ACCOUNT_USAGE.QUERY_HISTORY tenant-level Dynamic Tables | SLA compliance monitoring |

---

## 8. Appendix

### A. Full Object Count

| Type | Count |
|------|-------|
| Tables (regular) | 71 |
| Views | 69 |
| Dynamic Tables | 2 (LEGAL_ENTITY_DNB_SOURCE, PRODUCT_TERMS_ACCEPTANCE_ORGANIZATION) |
| **Total objects** | **142** |

### B. External Data Sources Integrated

| Source | Type | Volume | Purpose |
|--------|------|--------|---------|
| Salesforce (SFDC) | CRM | 360K+ accounts | Customer master, use case pipeline, enrichment target |
| Workday | HR/Finance | 20K employees, 6.8K vendors | Employee master, supplier data |
| ServiceNow | ITSM/VRM | varies | Vendor relationship management |
| GitHub | Engineering | 4.2M PRs, 21K daily metrics | Engineering activity |
| Crunchbase | Market Intel | 696K matches | Firmographic enrichment |
| FactSet | Financial Data | 49.5M matches | Financial intelligence |
| D&B (Dun & Bradstreet) | Identity | 324K DUNS | Legal entity identity |
| S&P Global | Financial | 25M matches | Financial intelligence |
| Forbes Global 2000 | Target Lists | varies | ICP account identification |
| SecurityScorecard | Vendor Security | SCD2 history | Continuous vendor security scoring |
| Panorays | Vendor Security | SCD2 history | Third-party risk scoring |
| Gong | Sales Intelligence | varies | Call intelligence via GONG_USER_HISTORY |

### C. Marketplace Opportunities for GAP Use Cases

| Data Product | Use Cases Enabled | Notes |
|--------------|------------------|-------|
| SaaS benchmarks (OpenView, Bessemer) | UC-1, UC-5 | Activation rate and ARR benchmarks for board reporting |
| Product analytics benchmark data | UC-1 | PLG funnel benchmarks |
| IP-to-company enrichment | DS-2 | Anonymous visitor → account matching |

### D. Methodology Notes

- **Vertical detection:** SFDC/Workday/ServiceNow/GitHub integration pattern + product terms/feature tables = Technology/SaaS (HIGH confidence). Score: PRIMARY (product, feature) × 3 + SECONDARY (user, session, engagement, pipeline, trial, ticket) × 7 = ~16/30 normalized
- **Layer classification:** MDM_INTERFACES is a Gold interface schema — published, downstream-facing. Upstream processing is in MDM_SOURCES and MDM_MODELED schemas (not assessed here)
- **Classification criteria:** READY = data + patterns fully present; NEAR = most data present, minor gaps (1-2 missing joins or features); GAP = fundamental data domain absent
- **Impact basis:** Published industry benchmarks from Gartner, Forrester, McKinsey, vendor case studies. No entity-specific calibration performed.
- **Limitations:** This assessment covers MDM_INTERFACES only. Snowflake likely has additional analytics schemas (FINANCE, SALES, SNOWHOUSE, etc.) with complementary or overlapping use cases. Connecting MDM party master to those schemas would substantially improve canonical fit scores.

---

*Generated by Snowflake Value Map skill | Cortex Code | July 2, 2026*
