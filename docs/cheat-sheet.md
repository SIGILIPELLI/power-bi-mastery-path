---
title: "Power BI Cheat Sheet: Quick Reference, Beginner to Advanced"
description: "A free Power BI cheat sheet: 34 quick-reference tables from every level of the Power BI Mastery Path, each linked to its full lesson."
---

# Power BI Cheat Sheet

Quick-reference tables from **34 lessons** of the free Power BI Mastery Path, in course order from beginner to advanced. Each section links to the full lesson with explanations and runnable examples.

## Level 1 · Entry

### [01 · What Power BI Is: Desktop, Service &amp; Mobile](level-1/01-what-is-power-bi.md)

*The pieces*

| Piece | Where it runs | What you do there |
|-------|---------------|-------------------|
| **Power BI Desktop** | Windows application (free download) | Connect to data, transform it in Power Query, build the model, write DAX, design report pages. Produces a `.pbix` file (or a PBIP folder, covered in Level 4). |
| **Power BI service** | Browser, at app.powerbi.com | Publish, share, schedule refresh, apply security, build dashboards, organize content in workspaces and apps. |
| **Power BI Mobile** | iOS and Android apps | Read reports and dashboards, with layouts optimized for phones. |
| **Power BI Report Builder** | Windows application | Author *paginated* (pixel-perfect, printable) reports. Covered in Level 3. |
| **On-premises data gateway** | A Windows machine inside your network | Lets the cloud service refresh data that lives behind your firewall. Covered in Level 2. |
| **Power BI Report Server** | Your own servers | An on-premises alternative to the service for organizations that cannot use the cloud. It follows its own release cadence. |

### [02 · Getting Data](level-1/02-getting-data.md)

*Connector families*

| Family | Examples | Notes |
|--------|----------|-------|
| Files | Text/CSV, Excel workbook, XML, JSON, Parquet, PDF | Path is stored in the query; moving the file breaks refresh. |
| Folders | Folder, SharePoint folder | Combine many files with the same shape into one table. |
| Databases | SQL Server, PostgreSQL, MySQL, Oracle, Snowflake, Databricks | Usually support **query folding** and often DirectQuery. |
| Online services | SharePoint Online list, Dataverse, Salesforce, Google Analytics | Authentication through an organizational or OAuth account. |
| Power Platform / Fabric | Semantic models, dataflows, lakehouses, warehouses | Reuse data other people already prepared. |
| Other | Web, OData, blank query | Blank query lets you type M directly. |

### [04 · Data Types &amp; Cleaning](level-1/04-data-types-cleaning.md)

*The types that matter*

| Power Query type | Model type | Use for |
|------------------|-----------|---------|
| Whole Number (`Int64.Type`) | Whole number | Counts, IDs, quantities |
| Decimal Number (`type number`) | Decimal number | Measurements, ratios |
| Fixed Decimal Number (`Currency.Type`) | Fixed decimal (4 places) | Money; avoids binary floating-point artifacts |
| Date / Date/Time / Time | Same | Dates; prefer **Date** unless you really need time |
| Text (`type text`) | Text | Labels, codes (including ZIP codes and phone numbers) |
| True/False (`type logical`) | Boolean | Flags |

### [05 · Building Your First Visuals](level-1/05-first-visuals.md)

*Step by step: revenue by region*

| Region | Sum of Revenue |
|--------|---------------|
| South | 700 |
| North | 445 |
| East | 280 |

### [06 · Filters &amp; Slicers](level-1/06-filters-slicers.md)

*The four layers*

| Layer | Set in | Applies to | Visible to readers? |
|-------|--------|-----------|---------------------|
| Visual-level filter | Filters pane → "Filters on this visual" | One visual | Optional (can be hidden/locked) |
| Page-level filter | Filters pane → "Filters on this page" | All visuals on the page | Optional |
| Report-level filter | Filters pane → "Filters on all pages" | Every page | Optional |
| Slicer | A visual on the canvas | By default, every visual on the same page | Always |

### [07 · Measures vs Calculated Columns](level-1/07-measures-vs-calculated-columns.md)

*Worked example 2: where only a measure is correct*

| Category | Revenue | Units | Avg Selling Price |
|----------|---------|-------|-------------------|
| Camping | 720 | 6 | 120.00 |
| Apparel | 480 | 6 | 80.00 |
| Accessories | 225 | 9 | 25.00 |
| **Total** | **1,425** | **21** | **67.86** |

### [08 · Report Formatting &amp; Layout](level-1/08-report-formatting.md)

*Number formats — set them on the measure*

| Measure | Format | Displays as |
|---------|--------|-------------|
| Total Revenue | Currency, 0 decimals | $1,425 |
| Total Units | Whole number | 21 |
| Avg Selling Price | Currency, 2 decimals | $67.86 |

### [09 · Publishing &amp; Sharing Basics](level-1/09-publishing-sharing.md)

*Workspaces*

| Role | Can do |
|------|--------|
| Admin | Everything, including managing membership and deleting the workspace |
| Member | Publish, edit, share, add Contributors/Viewers |
| Contributor | Publish and edit content; cannot share items or manage access by default |
| Viewer | View and interact; cannot edit |

### [10 · Project — A Sales Dashboard from CSV](level-1/10-project-sales-dashboard.md)

*Step 3 — The expected answers*

| Order | Gross | Net | | Order | Gross | Net |
|-------|------:|----:|-|-------|------:|----:|
| 2001 | 120 | 120.00 | | 2009 | 95 | 95.00 |
| 2002 | 150 | 135.00 | | 2010 | 220 | 209.00 |
| 2003 | 160 | 160.00 | | 2011 | 120 | 120.00 |
| 2004 | 190 | 180.50 | | 2012 | 96 | 86.40 |
| 2005 | 120 | 120.00 | | 2013 | 110 | 110.00 |
| 2006 | 240 | 216.00 | | 2014 | 285 | 270.75 |
| 2007 | 80 | 80.00 | | 2015 | 55 | 55.00 |
| 2008 | 75 | 75.00 | | 2016 | 240 | 240.00 |

## Level 2 · Intermediate

### [02 · Relationships, Cardinality &amp; Filter Direction](level-2/02-relationships.md)

*Cardinality*

| Cardinality | Meaning | Typical use |
|-------------|---------|-------------|
| One-to-many (1:*) | Key unique on the "one" side | Dimension → fact. The default and the one you want 95% of the time. |
| One-to-one (1:1) | Unique on both sides | Usually a sign two tables should be merged. |
| Many-to-many (*:*) | Neither side unique | Facts at different grains sharing a dimension attribute; use deliberately. |

### [03 · DAX Fundamentals](level-2/03-dax-fundamentals.md)

*Aggregators*

| Function | Returns | Sample result (Level 2 data) |
|----------|---------|-----|
| `SUM(FactSales[Amount])` | Sum of a column | 1,215 |
| `AVERAGE(FactSales[Amount])` | Mean of non-blank values | 1,215 ÷ 8 = 151.875 |
| `MIN` / `MAX(FactSales[Amount])` | Smallest / largest | 80 / 240 |
| `COUNTROWS(FactSales)` | Rows in a table | 8 |
| `DISTINCTCOUNT(FactSales[ProductKey])` | Distinct values (blank counts as a value) | 4 |
| `COUNTBLANK`, `COUNT`, `COUNTA` | Variants that treat blanks differently | — |

### [04 · CALCULATE, Filter Context &amp; Row Context](level-2/04-calculate-contexts.md)

*Worked example 1: a fixed filter*

| Region | Total Amount | Camping Amount | Hand check (Camping rows in region) |
|--------|-------------:|---------------:|-------------------------------------|
| East | 230 | *(blank)* | none |
| South | 315 | 215 | row 4 (120) + row 8 (95) |
| West | 670 | 430 | row 1 (240) + row 5 (190) |
| Total | 1,215 | 645 | |

### [05 · Time Intelligence &amp; the Date Table](level-2/05-time-intelligence.md)

*Sample data for this lesson*

| SalesKey | OrderDate | ProductKey | StoreKey | Units | Amount |
|---------:|-----------|-----------:|---------:|------:|-------:|
| 9 | 2024-01-15 | 1 | 1 | 3 | 380 |
| 10 | 2024-02-15 | 2 | 2 | 5 | 400 |
| 11 | 2024-03-15 | 3 | 3 | 12 | 290 |

### [06 · Power Query M: Merges, Appends &amp; Parameters](level-2/06-power-query-m.md)

*Merges are joins*

| Join kind | Keeps |
|-----------|-------|
| Left Outer | All rows from the first table, matches from the second |
| Right Outer | All rows from the second, matches from the first |
| Full Outer | All rows from both |
| Inner | Only matching rows |
| Left Anti | Rows in the first **with no match** in the second |
| Right Anti | Rows in the second with no match in the first |

### [08 · Row-Level Security](level-2/08-row-level-security.md)

*Step by step*

| Email | Region |
|-------|--------|
| maria@contoso.com | West |
| maria@contoso.com | South |
| dev@contoso.com | East |

### [09 · Refresh &amp; Gateways](level-2/09-refresh-gateways.md)

*What can refresh where*

| Source location | Refresh in the service needs |
|-----------------|------------------------------|
| Cloud source reachable from the internet (Azure SQL, SharePoint Online, Snowflake, a web API) | Credentials entered in the semantic model's settings |
| On-premises (a SQL Server in your data centre, a file share, a local CSV) | An **on-premises data gateway** plus credentials |
| Files on your own laptop (`C:\Users\...`) | Not realistic — move the file to SharePoint/OneDrive or a share the gateway can reach |

### [10 · Project — A Multi-Table Sales Model](level-2/10-project-sales-model.md)

*Tables*

| Category |
|----------|
| Camping |
| Apparel |
| Accessories |

## Level 3 · Advanced

### [02 · Context Transition](level-3/02-context-transition.md)

*Example 1: a calculated column, with and without CALCULATE*

| Product | Color | Qty | Price | Qty Plain | Qty Transitioned |
|---------|-------|----:|------:|----------:|-----------------:|
| A | Red | 2 | 10 | 12 | 2 |
| A | Blue | 1 | 10 | 12 | 1 |
| B | Red | 3 | 20 | 12 | 3 |
| C | Blue | 5 | 4 | 12 | 5 |
| C | Red | 1 | 4 | 12 | 1 |

### [03 · ALL, REMOVEFILTERS &amp; KEEPFILTERS](level-3/03-filter-modifiers.md)

*What about filters on the dimension?*

| Brand | Revenue | `REMOVEFILTERS ( Sales[Product] )` | `REMOVEFILTERS ( Sales )` |
|-------|--------:|-----------------------------------:|--------------------------:|
| North | 90 | 90 | 114 |
| Summit | 24 | 24 | 114 |

### [04 · Performance: VertiPaq &amp; Performance Analyzer](level-3/04-performance-vertipaq.md)

*Encoding: dictionary or value*

| Row | Color | Stored ID |
|----:|-------|----------:|
| 1 | Red | 0 |
| 2 | Blue | 1 |
| 3 | Red | 0 |
| 4 | Blue | 1 |
| 5 | Red | 0 |

### [05 · Import, DirectQuery &amp; Composite Models](level-3/05-storage-modes.md)

*The storage modes*

| Mode | Data lives | Queries answered by | Typical use |
|------|-----------|---------------------|-------------|
| **Import** | In the model (VertiPaq) | Local storage engine | Most tables; fastest |
| **DirectQuery** | In the source | The source database, via generated SQL | Very large or near-real-time facts |
| **Dual** | Both | Import cache *or* source, whichever the query needs | Dimensions shared by Import and DirectQuery facts |
| **Direct Lake** | Delta tables in OneLake (Fabric) | VertiPaq, loading column data on demand from Delta files | Fabric lakehouse/warehouse models (Level 4) |

### [06 · Calculation Groups](level-3/06-calculation-groups.md)

*Worked example*

| Year Month | Current | YTD | PY | YoY % |
|------------|--------:|----:|---:|------:|
| 2025-01 | 420 | 420 | 380 | 10.5% |
| 2025-02 | 460 | 880 | 400 | 15.0% |
| 2025-03 | 335 | 1,215 | 290 | 15.5% |

### [07 · Field Parameters](level-3/07-field-parameters.md)

*Creating a field parameter*

| Region | Total Amount | Total Units | Avg Price per Unit |
|--------|-------------:|------------:|-------------------:|
| West | 670 | 7 | 95.71 |
| South | 315 | 6 | 52.50 |
| East | 230 | 7 | 32.86 |

### [08 · Advanced Visuals, Custom Visuals &amp; Paginated Reports](level-3/08-advanced-visuals-paginated.md)

*Paginated reports*

| Need | Power BI report | Paginated report |
|------|-----------------|------------------|
| Explore, slice, drill | ✔ | Limited (parameters) |
| Print/PDF a 40-page invoice list | Awkward | ✔ |
| Operational list with every row | Table visual (scroll) | ✔ |
| Pixel-exact regulatory form | ✗ | ✔ |
| Scheduled email of a PDF/Excel export | Subscriptions exist but render the canvas | ✔ core use case |

### [09 · Deployment Pipelines](level-3/09-deployment-pipelines.md)

*What is — and isn't — copied*

| Copied | Not copied |
|--------|------------|
| Report definitions, semantic model **metadata** (tables, measures, relationships, roles) | The **data** in the model (each stage refreshes itself) |
| Dashboards, paginated reports, dataflows (depending on type support) | Workspace membership and permissions |
| | RLS **role membership** (who's in each role) |
| | Scheduled refresh settings and credentials (configured per stage) |
| | App configuration |

### [10 · Project — An Optimized Finance Model](level-3/10-project-finance-model.md)

*Data*

| AccountKey | Account | Line | Sign | LineOrder |
|-----------:|---------|------|-----:|----------:|
| 1 | Product Sales | Revenue | 1 | 1 |
| 2 | Service Revenue | Revenue | 1 | 1 |
| 3 | Materials | COGS | −1 | 2 |
| 4 | Salaries | Opex | −1 | 3 |
| 5 | Rent | Opex | −1 | 3 |

## Level 4 · Master

### [01 · Enterprise BI Architecture](level-4/01-enterprise-architecture.md)

*The layers*

| Layer | Owns | Typical tools | Owner |
|-------|------|---------------|-------|
| Ingestion & storage | Raw and cleaned tables, history | Data warehouse, lakehouse, pipelines | Data engineering |
| Transformation | Business-ready dimensions and facts | SQL views, dbt-style transforms, Spark notebooks, dataflows | Data/analytics engineering |
| Semantic model | Relationships, measures, security, formatting | Power BI semantic models | BI / analytics engineering |
| Reports | Pages, visuals, narrative | Power BI reports (thin), paginated reports | Report authors, business analysts |

### [02 · Power BI in Microsoft Fabric](level-4/02-microsoft-fabric.md)

*The building blocks*

| Fabric item | What it is | Relevance to Power BI |
|-------------|-----------|------------------------|
| **OneLake** | One logical data lake per tenant, organized by workspace; stores tables as Delta/Parquet | The common storage everything reads from |
| **Lakehouse** | Files + Delta tables, engineered with Spark/notebooks; comes with a SQL analytics endpoint (read-only T-SQL) | A source for semantic models |
| **Warehouse** | A T-SQL data warehouse storing its tables in OneLake | A source for semantic models; familiar to SQL teams |
| **Dataflow Gen2** | Power Query at scale, with output destinations such as a lakehouse | Moves Power Query logic upstream |
| **Data pipeline** | Orchestration (copy, schedule, dependencies) | Runs loads before model refresh |
| **Semantic model** | The same Power BI model you know | Import, DirectQuery, or **Direct Lake** |
| **Shortcuts** | References to data in other OneLake locations or external storage, without copying | Share tables across workspaces/domains |

### [03 · Governance &amp; Endorsed Semantic Models](level-4/03-governance.md)

*Endorsement: promoted and certified*

| Level | Who can apply | Meaning |
|-------|---------------|---------|
| **Promoted** | Anyone with write permission on the item | "The owner thinks this is ready for others to use" |
| **Certified** | Only people/groups authorized by the tenant admin | "This meets the organization's quality standard" |
| **Master data** (in Fabric) | Authorized reviewers | Core reference data, the single source for entities |

### [04 · Data Storytelling &amp; Dashboard Design](level-4/04-storytelling-design.md)

*A layout that follows the questions*

| Title as a finding: "Q1 sales 17% below target; Camping lags" |
| [Sales YTD] [Target YTD] [Achievement YTD %]  [Data as of ...]|
|--------------------------------------------------------------|
| Achievement by month (bars vs target line)  | Achievement by  |
|                                             | category (bars) |
|--------------------------------------------------------------|
| Detail table: category x month, conditional colour, drill →  |

### [05 · Embedding Power BI](level-4/05-embedding.md)

*The options*

| Option | Readers sign in with | Licensing (general) | Coding |
|--------|---------------------|---------------------|--------|
| **Teams / SharePoint Online** web part | Their own organizational account | Readers need access like any Power BI user | None |
| **Secure embed** (Embed → Website or portal) | Their own account, prompted in the iframe | Same as above | Paste an iframe |
| **Embed for your organization** ("user owns data") | Their own Microsoft Entra ID account, via your app | Readers need Pro/PPU or content in capacity | JavaScript + Entra app registration |
| **Embed for your customers** ("app owns data") | *Your app's* identity; users never see Microsoft sign-in | Your organization provides capacity (an embedding-capable SKU) | Back-end + JavaScript |
| **Publish to web** | Nobody — public | — | Public link; never for private data |

### [06 · Incremental Refresh](level-4/06-incremental-refresh.md)

*Worked example: what partitions exist*

| Partition | Covers | Refreshed on 2025-09-27? |
|-----------|--------|--------------------------|
| 2023 | Year 2023 | No (archive) |
| 2024 | Year 2024 | No |
| 2025 Q1 | Jan–Mar 2025 | No |
| 2025 Q2 | Apr–Jun 2025 | No |
| 2025-07 | July | **Yes** |
| 2025-08 | August | **Yes** |
| 2025-09 | September (to date) | **Yes** |

### [09 · Testing &amp; Validating Semantic Models](level-4/09-testing-models.md)

*What to test*

| Test type | Question | Example |
|-----------|----------|---------|
| **Reconciliation** | Does the model agree with the system of record? | Total sales for March equals the ERP's March sales report |
| **Measure unit tests** | Does each measure return the known answer on known data? | `Achievement %` for Camping in Q1 = 75.9% |
| **Structural tests** | Is the model shaped as intended? | Every fact row matches a dimension row (no "(Blank)" members); keys unique |
| **Security tests** | Does each role see exactly what it should? | Maria sees 985, Dev sees 230 |
| **Best-practice rules** | Does the model follow team standards? | No bidirectional relationships; all measures formatted |
| **Performance budgets** | Is the key page fast enough? | Overview page visuals under a set time on a cold cache |

### [10 · Capstone — An Enterprise Sales Analytics Solution](level-4/10-capstone.md)

*Step 3 — Expected results (your test oracle)*

| Measure | Expected |
|---------|---------:|
| Sales Amount | 1,215 |
| Target Amount | 1,460 |
| Achievement % | 83.2% |
| Sales PY | 1,070 |
| Sales YoY % | 13.6% (145 ÷ 1,070 = 0.1355) |

---

Go deeper with the full [Power BI Mastery Path](index.md) curriculum or test yourself with the [interview questions](interview-questions.md).
