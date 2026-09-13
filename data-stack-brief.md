# Data Stack & Touchpoints — Briefing for the Portfolio Build

**Purpose.** Background for whoever is building the portfolio. This is not copy to paste. It explains, in accurate terms, what data systems Neha works in and what she actually does in each one, so data vocabulary can be placed through the site correctly rather than decoratively.

**The rule for using it.** Every term below has a real referent. Use a term only where the underlying work is described in this doc. If a phrase would need a footnote to defend in an interview, it does not go on the site.

---

## 1. The stack, named correctly

Precision matters here, because these words are not interchangeable and a data-literate reader will notice. "Database" is wrong for almost everything below.

| Layer | System | The correct term | What it actually is | Neha's touchpoint |
|---|---|---|---|---|
| Source — ERP | **NetSuite** | ERP / system of record | General ledger, sales orders, subscriptions, billing | Pulls IS Detail exports and sales-order / subscription extracts; reconciles them against mastered records |
| Source — commerce | **Shopify, Amazon** | Commerce platforms | Order and customer data for a consumer subsidiary | Reconciles customer counts across both platforms |
| Master data | **Profisee** ("the Hub") | Master Data Management (MDM) platform | Where customer records are matched, merged and mastered into a single **golden record** with a stable ID | Queries mastered data; traced a reporting break to identifier changes during the migration onto this platform |
| File ingestion | **Azure Blob Storage** | Object storage / ingestion landing zone | Where client financial workbooks land before processing | Owns the filename convention (`Company Name_CV-<golden record ID>.xlsx`) that determines whether a file is picked up downstream at all |
| Analytics platform | **Microsoft Fabric** | Unified analytics platform | The umbrella product containing storage, warehouse, pipelines and the semantic layer — Fabric is the *platform*, not a database | Sits on the weekly reporting-migration working group |
| Storage foundation | **OneLake** | Fabric's single logical data lake | The storage layer underneath every Fabric item; data held as Delta/Parquet | Consumes data held here via the Warehouse |
| Query surface | **Fabric Warehouse** | Data warehouse (SQL endpoint over OneLake) | Where governed, queryable tables and views are exposed | **This is where Neha writes SQL.** Queries views she has been granted; specifies views that do not yet exist |
| Lakehouse | **Fabric Lakehouse** | Lakehouse | Delta tables plus files, used elsewhere in the estate for operational reporting | Downstream consumer only |
| Orchestration | **Fabric Data Pipelines** | Data pipelines | Scheduled loads that populate warehouse tables and downstream sheets | Consumer; established the rule that no automation may write to a pipeline-fed surface |
| Semantic / BI | **Power BI** | Semantic model + reports; **DAX** measures; scheduled refresh | The reporting layer leadership consumes; built and owned by the BI team | Consumption and extraction only — see §3 |
| Internal delivery | **10X Hub** | Internal reporting application | Where BizOps-built dashboards are surfaced to the business | Uses Hub dashboards as the reconciliation reference |
| Finance-owned automation | **Google Apps Script** | Scheduled job / data-sync pipeline | Time-triggered scripts syncing source workbooks into a central financials sheet, with failure alerting | Built and owned these; later retired the affected one when Fabric pipelines took over the same surface |
| Finance-owned integration | **Google Sheets `IMPORTRANGE`** | Live cross-workbook reference | The sanctioned way to read Fabric-fed data without an automation conflict | Migrated her reporting onto this pattern; hit and designed around the 10,000-row ceiling |

---

## 2. SQL: what she writes, and against what

**She writes SQL against the Fabric Warehouse.** Not the lakehouse, not notebooks — the warehouse's SQL surface, querying governed views exposed by the platform team.

Three views she has queried, described by grain, because grain is what tells a data reader whether someone has actually done this:

**Client counts by deal type.** Customer-level, classified new business vs. renewal. This is the view that replaced spreadsheet logic which had been *inferring* that classification from identifier lookups — the inference broke during the MDM migration, and the fix was to source the classification directly rather than derive it downstream.

**Event attendance, forward-looking and daily.** Attendance by event across future events, queried repeatedly to track day-over-day movement rather than a single snapshot, so a registration trend is visible before the event rather than reconstructed after it.

**Trailing-twelve-month client revenue, point-in-time.** Revenue by client across multiple platforms, resolvable as of any given date rather than only as of today. Sourced from and keyed on the mastered customer records in the MDM layer, so revenue attaches to the golden record rather than to a platform-specific customer ID.

That last one matters when placing vocabulary: **point-in-time** and **trailing twelve months** are not the same idea, and a view supporting both is a meaningfully harder thing than a report showing a current total.

---

## 3. Where the ownership line sits

This is the most important section for keeping the portfolio honest, and it is also what makes it read as senior rather than inflated.

**BizOps / the platform team owns:** OneLake, the Fabric Warehouse and Lakehouse, the tables inside them, the pipelines that load them, the governed views exposed on top, and the Power BI semantic models and DAX measures built over those.

**Neha owns:** the financial meaning of what comes out. Specifically —

- **Writes SQL against the Fabric Warehouse.** She queries governed views she has been granted. She does not create views, model tables, or own the warehouse. Where an FP&A sandbox has not been provisioned there is nothing to write against — an access boundary, not a capability one.
- **Specifies views that do not exist yet.** She defines what a view must return in business terms — grain, classification logic, date behaviour — the platform team builds it against tables they own, and she validates the output against source before anything reports off it.
- **Validates cross-system.** ERP extract vs. mastered record vs. warehouse view vs. dashboard: she is the person who checks those four agree, and finds out why when they do not.
- **Extracts into Excel for analysis.** Power BI is consumption-only for her — she reads the reports and pulls model data into Excel to work with it. She does not author reports, semantic models, or DAX measures. **Do not imply otherwise anywhere on the site.**
- **Brings the platform owner into the room deliberately.** When a reported figure is corrected, she has the tech owner who owns the underlying data review and validate the logic before the number goes out. This is not her needing help — it is what lets a stakeholder accept a restated number without relitigating it.

**Safe to say on the site:** writes SQL against governed warehouse views; works across ERP, master data and warehouse sources; specifies data requirements for the platform team; owns cross-system reconciliation and data-integrity controls.

**Not safe to say:** built the data warehouse; owns the lakehouse; data modelling; ETL or pipeline development; built SQL views; authors Power BI reports or DAX; data engineer.

---

## 4. What she actually does with data, by workstream

### Master data and reporting integrity *(the strongest data story)*
Reporting logic classified deals by looking up an ERP identifier against a central store and inferring a category from whether a prior record existed. When the customer master moved onto the MDM platform those lookups broke silently — helper columns stopped populating, and an entire category was misclassified while the output still looked plausible. She root-caused it to the identifier change rather than the formula, corrected the affected period, specified a replacement view returning the classification directly, and moved reporting onto the same query already feeding the enterprise dashboard — so finance and the business owner stopped deriving the same number two different ways.

### Pipeline reliability and the migration off finance-owned automation
She built scheduled Apps Script jobs syncing source workbooks into a central financials sheet on a daily trigger, with email alerting on failure. When Fabric pipelines began loading the same surfaces, concurrent execution produced stale, incomplete or blanked data. She disabled her own automation, moved the process to `IMPORTRANGE` against the pipeline-fed source, and the working group adopted that as the standard — no automation on any Fabric-connected surface, with a trailing-twelve-month customer-level pipeline output created to stay under the row ceiling.

### Cross-system reconciliation
Recurring work reconciling ERP subscription and sales-order extracts against mastered records: overdue-renewal reconciliation across a client population, deduplication where subsidiary reclassifications inflated counts, building a purchaser list directly from ERP sales-order data when a product configuration prevented it surfacing in the MDM layer, and customer-count reconciliation across two commerce platforms.

### Ingestion and the golden-record dependency
Client financial workbooks land in object storage and are only picked up into MDM if the filename carries the golden record ID. Without it the financials process correctly but the client silently disappears from the downstream customer-financials view. She identified this and owns the convention.

### Reporting logic diagnostics
Period-boundary defects from hidden timestamps in source dates, lookup ranges that drift, hardcoded dates in scorecards, and an upstream naming-convention standardisation that severed downstream joins. Each root-caused, corrected generically across periods, and regression-tested against known-good periods before release.

---

## 5. Vocabulary: say this, not that

| Don't say | Say | Why |
|---|---|---|
| Database | Warehouse / lakehouse / master data platform / ERP — whichever it is | "Database" is imprecise and reads as non-technical |
| Data | ERP extracts, mastered records, governed views, semantic model | Name the layer |
| Built a query | Wrote SQL against a governed view; specified a view | Accurate to who did what |
| Fabric database | Fabric Warehouse, Fabric Lakehouse, or Fabric pipeline | Fabric is a platform; these are different items inside it |
| Power BI data | Semantic model; DAX measures; scheduled refresh — *and only when describing the BI team's work* | The register a BI reader recognises |
| Customer ID | Golden record ID | MDM-specific and correct |
| Script | Scheduled pipeline / data-sync job | These ran on triggers with alerting |
| Cleaned the data | Reconciled across systems; root-caused to source | Cleaning is clerical; reconciling is analytical |
| Rolling 12 months and as-of date, used loosely | Trailing twelve months; point-in-time | Different concepts; conflating them signals inexperience |

---

## 6. Deliberately excluded

**Internal acronyms.** Several recurring internal report and feed names are deliberately not used in this doc or on the site. Do not introduce internal shorthand, and do not invent expansions for any acronym encountered elsewhere.

**Object names.** No table or view names appear here. The views in §2 are described by grain and business purpose, which is what a reader can actually evaluate.

**Employer identifiers.** No company, client, colleague or system-instance names. The portfolio remains anonymised and synthetic throughout.

---

## 7. One item for Neha to verify before publishing

**The SQL dialect.** The Fabric Warehouse's query language is T-SQL — that is the SQL surface it exposes, and there is no separate language called "Power BI SQL." So SQL written against the warehouse is T-SQL, even where the query editor or connection dialog never labels it that way.

This doc therefore says **"SQL against the Fabric Warehouse,"** which is correct and defensible exactly as written, and can go to the developer as-is. Naming the dialect outright — "T-SQL" — would be stronger and is very likely accurate, but confirm it on the query surface first. A dialect name is precisely the detail an interviewer follows up on.

---

*Prepared as background for the portfolio build.*
