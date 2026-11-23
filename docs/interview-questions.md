---
title: "Power BI Interview Questions and Answers (179 Questions)"
description: "179 Power BI interview questions with answers and explanations, from beginner to advanced, organised by level. Free, from the Power BI Mastery Path."
---

# Power BI Interview Questions and Answers

**179 questions** with the correct answer and a short explanation, organised from beginner (Level 1) to advanced (Level 4). Every question comes from a lesson in the free Power BI Mastery Path, so you can open the lesson to go deeper.

!!! tip "Practise, don't just read"
    Each lesson also ends with an interactive quiz, and the [final quiz](quiz.md) tests all four levels. Try to answer each question below before reading the answer.

## Level 1 · Entry

### [01 · What Power BI Is: Desktop, Service &amp; Mobile](level-1/01-what-is-power-bi.md)

**Q1.** What are the main parts of the Power BI toolset?

**Answer:** Power BI Desktop (author), the Service (share/refresh) and Mobile (consume)

Typical workflow: build in Desktop, publish to the Service.

**Q2.** What does Power BI do at a high level?

**Answer:** Connects to data, models it and presents interactive reports and dashboards

Get data, model, visualize, share.

**Q3.** What is the difference between a report and a dashboard?

**Answer:** A report has multiple pages of interactive visuals; a dashboard is a single canvas of pinned tiles in the Service

Tiles link back to reports.

**Q4.** Which language is used for calculations in the model?

**Answer:** DAX

M is for Power Query.

### [02 · Getting Data](level-1/02-getting-data.md)

**Q5.** What does Import mode do?

**Answer:** Copies data into the model's in-memory engine for fast queries (refresh needed for new data)

Fast but stale until refresh.

**Q6.** What is DirectQuery?

**Answer:** Queries the source live at report time instead of storing the data

Fresher but slower and limited.

**Q7.** Why prefer a query-foldable source step?

**Answer:** The work is pushed to the source database instead of done in Power Query locally

Folding speeds refresh.

**Q8.** A CSV path points to your Downloads folder. What breaks after publishing?

**Answer:** Scheduled refresh can't reach a personal path without a gateway; use a shared/cloud location

Plan data access early.

### [03 · Power Query Basics](level-1/03-power-query-basics.md)

**Q9.** Which pane records the steps of a query?

**Answer:** Applied Steps in Power Query

Each step is M code.

**Q10.** What does 'Use First Row as Headers' do?

**Answer:** Promotes the first data row to column names

A common first step.

**Q11.** Why remove unneeded columns early in Power Query?

**Answer:** Smaller model, faster refresh and less to maintain

Also helps folding.

**Q12.** Where does data transformation belong relative to DAX?

**Answer:** Shape and clean in Power Query; calculate business logic in DAX measures

Right tool for each.

### [04 · Data Types &amp; Cleaning](level-1/04-data-types-cleaning.md)

**Q13.** A Price column loaded as text and won't sum. What is the fix?

**Answer:** Change the column data type to a number in Power Query

Types drive behaviour.

**Q14.** What does 'Replace Errors' help with?

**Answer:** Handling cells that fail a type conversion instead of failing the refresh

Decide deliberately.

**Q15.** Why trim and clean text before relating tables?

**Answer:** Stray spaces make 'Acme' and 'Acme ' different keys

Relationships need exact matches.

**Q16.** A date arrives as '03/04/2026'. What's the risk?

**Answer:** Day/month ambiguity depending on locale; set the column type using the correct locale

Use 'Change type with locale'.

### [05 · Building Your First Visuals](level-1/05-first-visuals.md)

**Q17.** Which visual shows a trend over time best?

**Answer:** A line chart

Time on the axis.

**Q18.** When is a pie chart a poor choice?

**Answer:** With many categories or when values must be compared precisely

Bars compare better.

**Q19.** What does a card visual show?

**Answer:** A single key number

KPI headline.

**Q20.** What do you do to see exact values behind a visual?

**Answer:** Use 'Show as a table' or data labels/tooltips

Accessibility too.

### [06 · Filters &amp; Slicers](level-1/06-filters-slicers.md)

**Q21.** What's the difference between a slicer and a visual-level filter?

**Answer:** A slicer is an on-canvas control users change; a filter pane filter is set by the author (and can be locked)

Choose by who should control it.

**Q22.** What does 'Sync slicers' do?

**Answer:** Applies the same slicer selection across selected pages

Consistent filtering.

**Q23.** What is the filter scope order from widest to narrowest?

**Answer:** Report → page → visual

Narrower scopes add to wider ones.

**Q24.** Why avoid dozens of slicers on one page?

**Answer:** Clutter and slow rendering; use filters or a smaller set of well-chosen slicers

Design for the question.

### [07 · Measures vs Calculated Columns](level-1/07-measures-vs-calculated-columns.md)

**Q25.** Sales rows (units, price): (2,10),(3,5),(1,20). A measure SUMX(units*price) total is?

```python
print(2*10+3*5+1*20)
```

**Answer:** 55

Measures evaluate per filter context over rows; a calculated column would store row values. Verified by running it.

**Q26.** When use a measure instead of a calculated column?

**Answer:** When the result depends on filters/slicers (totals, ratios, YoY)

Measures are computed at query time.

**Q27.** When is a calculated column appropriate?

**Answer:** For a row-level attribute used in slicers/axes or relationships

Stored in the model.

**Q28.** Why can't you average a percentage column to get the overall rate?

**Answer:** The average of ratios ignores the weights; compute numerator and denominator in a measure

DIVIDE(SUM(a), SUM(b)).

### [08 · Report Formatting &amp; Layout](level-1/08-report-formatting.md)

**Q29.** Which formatting aids readability most?

**Answer:** Consistent alignment, limited colors, clear titles and labelled units

Simple wins.

**Q30.** What do themes give you?

**Answer:** Consistent colors and fonts across reports

JSON themes.

**Q31.** Why check the report on mobile layout?

**Answer:** Phone users see a different canvas; design a mobile view for key visuals

Mobile layout.

**Q32.** What is conditional formatting for?

**Answer:** Highlighting values by rules (e.g. red below target)

Draw attention.

### [09 · Publishing &amp; Sharing Basics](level-1/09-publishing-sharing.md)

**Q33.** What does publishing do?

**Answer:** Uploads the report and dataset to a workspace in the Power BI Service

Then share/refresh.

**Q34.** What do viewers typically need to open a shared report?

**Answer:** Appropriate license/access (Pro/PPU/Fabric capacity) per sharing method

Check licensing.

**Q35.** What is an app?

**Answer:** A packaged set of reports/dashboards published to a wider audience

Managed distribution.

**Q36.** Why use workspaces?

**Answer:** Collaborate on content with roles (Admin, Member, Contributor, Viewer)

Access control.

### [10 · Project — A Sales Dashboard from CSV](level-1/10-project-sales-dashboard.md)

**Q37.** What does the sales dashboard project combine?

**Answer:** Get data, Power Query cleaning, measures, visuals, slicers and publishing

Level 1 project.

**Q38.** How validate dashboard numbers?

**Answer:** Reconcile totals against the source system or a trusted report

Verification.

**Q39.** What is a good first page?

**Answer:** Key KPIs and one or two trends answering the main question

Story first.

**Q40.** What do you document?

**Answer:** Data sources, refresh schedule, measure definitions and owners

Maintainability.

## Level 2 · Intermediate

### [01 · Star Schema &amp; Data Modeling](level-2/01-star-schema.md)

**Q41.** Why use a star schema instead of one flat table?

**Answer:** Correct, fast aggregation, reusable dimensions and no duplicated attributes

Engine is optimized for it.

**Q42.** What is the grain of a fact table?

**Answer:** What one row represents (e.g. one order line)

Decide first.

**Q43.** A monthly target is copied onto each order line and then summed. What goes wrong?

**Answer:** The target is multiplied by the number of lines; targets need their own fact at a different grain

Don't mix grains.

**Q44.** Which table type holds descriptive attributes?

**Answer:** Dimension tables

Short and wide.

### [02 · Relationships, Cardinality &amp; Filter Direction](level-2/02-relationships.md)

**Q45.** What is the usual relationship direction between a dimension and a fact?

**Answer:** One-to-many, filtering from the dimension to the fact

Single direction is the default.

**Q46.** When is bidirectional filtering risky?

**Answer:** It can create ambiguity and unintended filtering paths and slow queries

Use sparingly.

**Q47.** What breaks if a dimension key has duplicates?

**Answer:** The one-side requirement fails and the relationship can't be one-to-many

Unique keys.

**Q48.** What does an inactive relationship need?

**Answer:** USERELATIONSHIP in a measure to activate it

Role-playing dates.

### [03 · DAX Fundamentals](level-2/03-dax-fundamentals.md)

**Q49.** If Sales are [100, 250, 50] across three rows, SUM gives and AVERAGE gives?

```python
s=[100,250,50]; print(sum(s), round(sum(s)/len(s),2))
```

**Answer:** 400 133.33

SUM and AVERAGE aggregate over the current filter context. Verified by running it.

**Q50.** What does DIVIDE(a, b) do when b is zero?

**Answer:** Returns BLANK (or a given alternate) instead of an error

Safe division.

**Q51.** Which function counts distinct values of a column?

**Answer:** DISTINCTCOUNT

Counts unique.

**Q52.** What is a measure's evaluation context made of?

**Answer:** Filters from slicers, visuals, rows/columns and relationships

Filter context.

### [04 · CALCULATE, Filter Context &amp; Row Context](level-2/04-calculate-contexts.md)

**Q53.** What does CALCULATE do?

**Answer:** Evaluates an expression in a modified filter context

The core DAX function.

**Q54.** What does CALCULATE([Sales], Product[Color] = "Red") do?

**Answer:** Computes Sales for red products, replacing any existing filter on Color

Filter arguments override.

**Q55.** What does ALL(Product) inside CALCULATE do?

**Answer:** Removes filters on the Product table so you can compute a total or denominator

Percent of total.

**Q56.** What's the difference between row context and filter context?

**Answer:** Row context iterates rows (calculated columns, iterators); filter context is the set of applied filters

CALCULATE converts one to the other.

### [05 · Time Intelligence &amp; the Date Table](level-2/05-time-intelligence.md)

**Q57.** What does time intelligence require?

**Answer:** A proper date table marked as the date table, with contiguous dates

No gaps.

**Q58.** What does TOTALYTD compute?

**Answer:** A running total from the start of the year to the current date in context

Year to date.

**Q59.** How compare to last year?

**Answer:** CALCULATE([Sales], SAMEPERIODLASTYEAR('Date'[Date]))

Shifted dates.

**Q60.** Sales were 120 this year and 100 last year. What is YoY growth in percent?

```python
print(round((120-100)/100*100,1))
```

**Answer:** 20.0

(This - last) / last. Verified by running it.

### [06 · Power Query M: Merges, Appends &amp; Parameters](level-2/06-power-query-m.md)

**Q61.** What language is used in the Power Query Editor?

**Answer:** M

Functional language behind the UI.

**Q62.** What does a query parameter allow?

**Answer:** Reusing a value (like a server or date) across queries and environments

Dev/prod switching.

**Q63.** When merge vs append queries?

**Answer:** Merge joins columns by key; append stacks rows

Like JOIN vs UNION.

**Q64.** What does disabling 'Enable load' do?

**Answer:** Keeps the query as a staging step without loading it into the model

Cleaner models.

### [07 · Drill-through, Tooltips &amp; Bookmarks](level-2/07-drillthrough-bookmarks.md)

**Q65.** What does drillthrough let users do?

**Answer:** Right-click a value to jump to a detail page filtered to it

Context passes to the target page.

**Q66.** What does a bookmark capture?

**Answer:** The state of a page: filters, visibility and selections

Guided navigation.

**Q67.** When use buttons with bookmarks?

**Answer:** To switch views or toggle panes in a report

Interactivity.

**Q68.** Why name pages and buttons clearly?

**Answer:** Maintainability and accessibility (screen readers)

Good habits.

### [08 · Row-Level Security](level-2/08-row-level-security.md)

**Q69.** What is row-level security (RLS)?

**Answer:** Roles with DAX filters restricting which rows each user can see

Defined in Desktop.

**Q70.** Where are users assigned to RLS roles?

**Answer:** In the Power BI Service on the dataset's security settings

Roles are authored in Desktop.

**Q71.** What does USERPRINCIPALNAME() enable?

**Answer:** Dynamic RLS by filtering with the logged-in user's email

Mapping table.

**Q72.** Do workspace Admins/Members/Contributors have RLS applied?

**Answer:** No; RLS applies to viewers, so use the Viewer role for testing

Common surprise.

### [09 · Refresh &amp; Gateways](level-2/09-refresh-gateways.md)

**Q73.** What does a gateway do?

**Answer:** Lets the Service refresh data from on-premises sources

Install on a reliable machine.

**Q74.** What is the difference between personal and standard gateway?

**Answer:** Personal is for one user's Import refresh; standard serves many users and DirectQuery

Choose for the org.

**Q75.** How often can Pro schedule refresh?

**Answer:** Up to 8 times per day (Premium: up to 48)

Plan within limits.

**Q76.** What do you check when scheduled refresh fails?

**Answer:** Credentials, gateway status, source availability and the refresh history error

Diagnose from history.

### [10 · Project — A Multi-Table Sales Model](level-2/10-project-sales-model.md)

**Q77.** What does the sales model project combine?

**Answer:** Star schema, relationships, DAX measures, time intelligence and RLS

Level 2 project.

**Q78.** How test RLS?

**Answer:** Use 'View as role' in Desktop and as a Viewer in the Service

Roles don't apply to editors.

**Q79.** How verify time intelligence?

**Answer:** Compare YTD/YoY values against manual totals for a few periods

Check boundaries.

**Q80.** What do you ship besides the report?

**Answer:** Documentation of model, measures and refresh

Handover.

## Level 3 · Advanced

### [01 · Iterators &amp; Variables](level-3/01-iterators-variables.md)

**Q81.** What does SUMX do?

**Answer:** Iterates a table row by row evaluating an expression, then sums it

Row context.

**Q82.** Why use VAR in DAX?

**Answer:** Readability and evaluating an expression once (and fixing context)

Cleaner measures.

**Q83.** Why is SUMX(Sales, Qty * Price) correct for revenue when there is no Amount column?

**Answer:** Each row's Qty*Price must be computed before summing

Sum of products ≠ product of sums.

**Q84.** Qty/Price rows: (2,10),(3,5). Product of sums vs sum of products?

```python
print((2+3)*(10+5), 2*10+3*5)
```

**Answer:** 75 35

Iterators multiply per row. Verified by running it.

### [02 · Context Transition](level-3/02-context-transition.md)

**Q85.** What is context transition?

**Answer:** CALCULATE turning the current row context into an equivalent filter context

Why measures work in iterators.

**Q86.** What happens when a measure is referenced in a calculated column?

**Answer:** It is evaluated with the row converted to a filter (implicit CALCULATE)

Context transition.

**Q87.** What's a common performance trap?

**Answer:** Iterating large tables and using context transition per row

Push work to storage engine.

**Q88.** What does a measure reference do inside SUMX?

**Answer:** Evaluates per row with that row's values as filters

Powerful but costly.

### [03 · ALL, REMOVEFILTERS &amp; KEEPFILTERS](level-3/03-filter-modifiers.md)

**Q89.** What does ALLEXCEPT do?

**Answer:** Removes all filters from a table except on the listed columns

Group-level totals.

**Q90.** What does KEEPFILTERS do?

**Answer:** Adds to existing filters instead of overriding them

Intersects.

**Q91.** What does REMOVEFILTERS do?

**Answer:** Clears filters (clearer alias of ALL as a CALCULATE modifier)

Readable.

**Q92.** Which function computes percent of grand total?

**Answer:** DIVIDE([Sales], CALCULATE([Sales], REMOVEFILTERS()))

Denominator ignores filters.

### [04 · Performance: VertiPaq &amp; Performance Analyzer](level-3/04-performance-vertipaq.md)

**Q93.** What is VertiPaq?

**Answer:** The in-memory columnar compression engine behind Import models

Column store.

**Q94.** What reduces model size most?

**Answer:** Removing unused columns and lowering cardinality (e.g. split datetime)

Fewer unique values compress better.

**Q95.** What tool analyses model size and DAX performance?

**Answer:** DAX Studio / VertiPaq Analyzer and Performance Analyzer in Desktop

Measure first.

**Q96.** Which data type compresses best?

**Answer:** Integers with low cardinality

Prefer ints for keys.

### [05 · Import, DirectQuery &amp; Composite Models](level-3/05-storage-modes.md)

**Q97.** What is composite model?

**Answer:** A model mixing Import and DirectQuery tables

Flexibility.

**Q98.** What is Dual storage mode?

**Answer:** A table that can act as Import or DirectQuery depending on the query

Optimizes relationships.

**Q99.** When choose DirectQuery?

**Answer:** Very large or near-real-time data where import isn't feasible

Trade-offs in speed.

**Q100.** What do aggregations do?

**Answer:** Serve common queries from a small imported summary instead of the big DirectQuery table

Speed at scale.

### [06 · Calculation Groups](level-3/06-calculation-groups.md)

**Q101.** What do calculation groups solve?

**Answer:** Many near-identical measures (e.g. YTD, YoY for every measure) with one reusable item

Created in Tabular Editor.

**Q102.** Which tool typically authors calculation groups?

**Answer:** Tabular Editor (or the model view in recent Desktop)

External tool.

**Q103.** What is SELECTEDMEASURE()?

**Answer:** The measure currently being evaluated by a calculation item

Used in items.

**Q104.** What's the caution?

**Answer:** Precedence and format strings need design to avoid surprising results

Test combos.

### [07 · Field Parameters](level-3/07-field-parameters.md)

**Q105.** What are field parameters?

**Answer:** Slicers that let users switch which measures or columns a visual uses

Dynamic visuals.

**Q106.** What benefit do field parameters give?

**Answer:** One flexible visual instead of many duplicates

Self-service.

**Q107.** What is created behind the scenes?

**Answer:** A calculated table with NAMEOF references

Generated DAX.

**Q108.** What's a limitation?

**Answer:** Not all visuals/features behave identically with parameters; test them

Check behaviors.

### [08 · Advanced Visuals, Custom Visuals &amp; Paginated Reports](level-3/08-advanced-visuals-paginated.md)

**Q109.** What are paginated reports for?

**Answer:** Pixel-perfect, printable multi-page documents (invoices, statements)

Report Builder.

**Q110.** What is a decomposition tree good for?

**Answer:** Interactive root-cause drilling across dimensions

AI visual.

**Q111.** When choose a matrix over a table?

**Answer:** When you need row/column hierarchies and subtotals

Pivot-like.

**Q112.** What do tooltips pages add?

**Answer:** Custom hover details with visuals

Context at a glance.

### [09 · Deployment Pipelines](level-3/09-deployment-pipelines.md)

**Q113.** What do deployment pipelines do?

**Answer:** Promote content through Dev, Test and Prod workspaces with rules

Release management.

**Q114.** What are deployment rules for?

**Answer:** Changing parameters/data sources per stage (e.g. dev DB vs prod DB)

Environment config.

**Q115.** Why test in a separate stage?

**Answer:** Catch issues before users see them

Safe releases.

**Q116.** What should be compared before promoting?

**Answer:** Differences between stages (the pipeline shows changes)

Review changes.

### [10 · Project — An Optimized Finance Model](level-3/10-project-finance-model.md)

**Q117.** What does the finance model project combine?

**Answer:** Calculation groups, iterators, variables, performance tuning and deployment

Level 3 project.

**Q118.** How validate finance numbers?

**Answer:** Reconcile to the ledger and audit measure logic with test cases

Accuracy matters.

**Q119.** How keep measures understandable?

**Answer:** Descriptive names, VARs, comments and a measure table

Readable.

**Q120.** What would you add for governance?

**Answer:** Sensitivity labels, certified datasets and ownership docs

Trust.

## Level 4 · Master

### [01 · Enterprise BI Architecture](level-4/01-enterprise-architecture.md)

**Q121.** What's a common enterprise pattern?

**Answer:** Shared certified datasets with thin reports built on top

One version of truth.

**Q122.** Why separate data prep from the model?

**Answer:** Reuse via dataflows and keep the semantic model lean

Layered architecture.

**Q123.** What does a semantic model provide?

**Answer:** Business definitions: relationships, measures and security reused by many reports

Single source.

**Q124.** What does 'thin report' mean?

**Answer:** A report connected to a shared model without its own data

Live connection.

### [02 · Power BI in Microsoft Fabric](level-4/02-microsoft-fabric.md)

**Q125.** What is Microsoft Fabric?

**Answer:** A unified analytics platform (OneLake, lakehouse, warehouse, pipelines, Power BI)

Integrated.

**Q126.** What is OneLake?

**Answer:** A single logical data lake for the organization

Delta Parquet.

**Q127.** What is Direct Lake mode?

**Answer:** Reading Delta tables from OneLake directly, with import-like speed without copying

Fabric feature.

**Q128.** When use a lakehouse vs warehouse?

**Answer:** Lakehouse for flexible files/Spark; warehouse for SQL-centric structured analytics

Choose by workload.

### [03 · Governance &amp; Endorsed Semantic Models](level-4/03-governance.md)

**Q129.** What does governance cover?

**Answer:** Ownership, access, quality, lifecycle and compliance of data and reports

Trust and safety.

**Q130.** What do sensitivity labels do?

**Answer:** Classify and protect content, following it when exported

Information protection.

**Q131.** What does endorsement (certified/promoted) signal?

**Answer:** Which content is trusted and recommended

Discoverability.

**Q132.** Why audit usage?

**Answer:** See who uses what, retire unused content and detect risk

Admin portal and metrics.

### [04 · Data Storytelling &amp; Dashboard Design](level-4/04-storytelling-design.md)

**Q133.** What makes a report tell a story?

**Answer:** A clear question, logical flow and highlighted insights with context

Audience first.

**Q134.** What is the data-ink principle?

**Answer:** Remove decoration that doesn't carry information

Clarity.

**Q135.** Why consistent layout across pages?

**Answer:** Users learn once and find things faster

Predictability.

**Q136.** What do titles that state the takeaway do?

**Answer:** Tell viewers what to notice ('Sales up 12%') rather than just labelling

Insight titles.

### [05 · Embedding Power BI](level-4/05-embedding.md)

**Q137.** What does 'embed for your customers' mean?

**Answer:** App-owns-data embedding for external users without Power BI licenses

Service principal + capacity.

**Q138.** What does 'embed for your organization' mean?

**Answer:** User-owns-data: internal users sign in with their own license

Secure embed.

**Q139.** What embeds a report in a web app?

**Answer:** The Power BI JavaScript embedding SDK with an embed token

Programmatic.

**Q140.** Why handle RLS with embedding?

**Answer:** So each external user sees only their data via the effective identity

Security.

### [06 · Incremental Refresh](level-4/06-incremental-refresh.md)

**Q141.** What does incremental refresh do?

**Answer:** Refreshes only recent partitions instead of the whole table

RangeStart/RangeEnd.

**Q142.** Which parameters does incremental refresh need?

**Answer:** RangeStart and RangeEnd (datetime) filtering the source

Folded filters.

**Q143.** Why is query folding vital for it?

**Answer:** The date filter must reach the source so only needed rows are read

Else full reads.

**Q144.** What's the benefit?

**Answer:** Faster, cheaper refreshes of large tables

Scale.

### [07 · CI/CD with PBIP &amp; Git Integration](level-4/07-cicd-pbip-git.md)

**Q145.** What is the PBIP format for?

**Answer:** A folder-based project format (TMDL/PBIR) friendly to source control

Git diffs.

**Q146.** Why use Git for Power BI?

**Answer:** Version history, review of changes and collaboration

Engineering practice.

**Q147.** What does Fabric Git integration do?

**Answer:** Syncs a workspace with a Git repository branch

Source of truth.

**Q148.** What goes in CI checks?

**Answer:** Best-practice analyzer, model validation and DAX tests

Quality gates.

### [08 · Accessible Reports](level-4/08-accessibility.md)

**Q149.** What helps screen reader users?

**Answer:** Alt text, logical tab order, titles and 'show as table'

WCAG basics.

**Q150.** What contrast is recommended for text?

**Answer:** At least 4.5:1 against background

Readable.

**Q151.** Why not use color alone to convey meaning?

**Answer:** Color-blind users can't distinguish; add labels/patterns

Redundant cues.

**Q152.** What does tab order control?

**Answer:** The sequence keyboard users move through visuals

Set deliberately.

### [09 · Testing &amp; Validating Semantic Models](level-4/09-testing-models.md)

**Q153.** How test a semantic model?

**Answer:** Compare measure results with known values and check edge cases (blanks, filters)

Automated where possible.

**Q154.** What tools help?

**Answer:** DAX Query View, Tabular Editor Best Practice Analyzer, and DAX Studio

Static and dynamic checks.

**Q155.** What do regression tests catch?

**Answer:** Unintended changes in numbers after model edits

Compare before/after.

**Q156.** What should test data cover?

**Answer:** Typical, boundary and empty-filter cases

Coverage.

### [10 · Capstone — An Enterprise Sales Analytics Solution](level-4/10-capstone.md)

**Q157.** What does the capstone combine?

**Answer:** Architecture, governance, performance, security, deployment and storytelling

Production-ready BI.

**Q158.** How prove readiness?

**Answer:** Reconciled numbers, performance checks, RLS tests and a release plan

Evidence.

**Q159.** What documentation helps?

**Answer:** Data dictionary, measure definitions, refresh and support runbook

Handover.

**Q160.** What sustains it?

**Answer:** Ownership, usage review and regular model maintenance

Lifecycle.

## Mixed questions across all levels

**Q161.** A sales table has Units and Price columns and no Amount column. Which measure gives revenue?

**Answer:** SUMX(Sales, Sales[Units] * Sales[Price])

Multiply per row first, then sum.

**Q162.** Rows (units, price): (2,10),(3,5). What do SUMX(units*price) and SUM(units)*SUM(price) give?

```python
print(2*10+3*5, (2+3)*(10+5))
```

**Answer:** 35 75

Sum of products is not the product of sums. Verified by running it.

**Q163.** Slicer Category=Camping is applied and a measure uses CALCULATE([Sales], ALL(Product)). What does it return?

**Answer:** Total sales for all products (the Category filter is removed)

Used for denominators.

**Q164.** Why does a flat table hide a product with no sales while a star schema doesn't?

**Answer:** The dimension table still has the product row even without fact rows

Dimensions list everything.

**Q165.** A measure returns an overall rate by averaging a per-row percentage column. What's wrong?

**Answer:** It ignores weights; compute DIVIDE(SUM(numerator), SUM(denominator))

Ratio of sums.

**Q166.** TOTALYTD returns blanks for some dates. What do you check first?

**Answer:** That a contiguous, marked date table exists and is related to the fact

Time intelligence prerequisite.

**Q167.** Sales this year 150 versus 120 last year. Which expression gives growth?

**Answer:** DIVIDE([Sales] - [Sales LY], [Sales LY])

(150-120)/120 = 25%.

**Q168.** A user can see data for all regions in the Service, but the Desktop test with role 'West' worked. What's missing?

**Answer:** Users must be assigned to the role in the Service (and testing needs the Viewer role)

Assignments are separate.

**Q169.** A scheduled refresh from an on-premises SQL Server fails after publishing. What is needed?

**Answer:** A data gateway with valid credentials

On-prem access.

**Q170.** Two relationships connect Sales to Date (OrderDate, ShipDate). How use the inactive one?

**Answer:** CALCULATE([Sales], USERELATIONSHIP(Sales[ShipDate], 'Date'[Date]))

Role-playing dimension.

**Q171.** Which refresh strategy suits a 200-million-row fact table that only changes recently?

**Answer:** Incremental refresh with RangeStart/RangeEnd and folded date filters

Partitions.

**Q172.** A report is slow. What is the first diagnostic step?

**Answer:** Performance Analyzer to see which visual/query is slow, then DAX Studio if needed

Measure.

**Q173.** Which change typically shrinks a model the most?

**Answer:** Removing unused columns and reducing cardinality (e.g. split datetime into date and time)

VertiPaq compression.

**Q174.** Many measures need YTD, YoY and MTD variants. What reduces duplication?

**Answer:** Calculation groups

Reusable time calculations.

**Q175.** Which approach keeps content consistent for hundreds of reports?

**Answer:** Shared certified semantic models with thin reports

One version of truth.

**Q176.** What lets external customers view embedded reports without Power BI licenses?

**Answer:** Embed for your customers (app owns data) with an embed token

Capacity required.

**Q177.** Why store a Power BI project as PBIP in Git?

**Answer:** Text-based files give reviewable diffs and branching

CI/CD.

**Q178.** Which practice most improves accessibility?

**Answer:** Alt text, logical tab order, sufficient contrast and not relying on color alone

WCAG.

**Q179.** How do you confirm a model change didn't break numbers?

**Answer:** Compare measure outputs before and after against known values (regression tests)

Automate in CI.

## Practical coding questions

### A filter-context engine: CALCULATE, REMOVEFILTERS and DIVIDE

Power BI Desktop isn't available here, so model its core idea in Python: join facts to dimensions, filter rows by a context dictionary, then implement CALCULATE (override and remove filters), a safe DIVIDE and a percent-of-total measure. Also list products with no sales, which a star schema can answer but a flat table cannot.

Starter code:

```
# A tiny model of Power BI's filter context: dimensions filter a fact table through relationships.
FACT = [  # (date, product_key, store_key, units, amount)
    ("2026-01-05", 1, 1, 2, 200), ("2026-01-20", 2, 1, 1, 150), ("2026-02-03", 1, 2, 4, 400),
    ("2026-02-14", 3, 2, 2, 90),  ("2026-03-02", 2, 1, 3, 450), ("2026-03-15", 1, 2, 1, 100),
]
DIM_PRODUCT = {1: {"Product": "Tent", "Category": "Camping"}, 2: {"Product": "Boots", "Category": "Hiking"},
               3: {"Product": "Stove", "Category": "Camping"}, 4: {"Product": "Lamp", "Category": "Camping"}}
DIM_STORE = {1: {"Store": "Austin", "Region": "South"}, 2: {"Store": "Denver", "Region": "West"}}

def rows_in_context(filters):
    """filters = {"Category": "Camping", "Region": "West", "Month": "2026-02"}; no filters means all rows.
    Join each fact row to its product and store attributes (plus Month = first 7 chars of the date) and keep
    the rows where every filter column equals its value."""
    return []

def total_sales(filters):                      # SUM(Sales[Amount]) in the given context
    return 0

def divide(n, d, alt=None):                    # DIVIDE(n, d) returns BLANK (None) when d is 0
    return alt if not d else n / d

def calculate(measure, context, add=None, remove=None):
    """CALCULATE: copy the context, REMOVEFILTERS on the named columns, then set/override filters from `add`."""
    ctx = dict(context)
    for k in (remove or []): ctx.pop(k, None)
    ctx.update(add or {})
    return measure(ctx)

def pct_of_region(context):                    # DIVIDE([Sales], CALCULATE([Sales], REMOVEFILTERS(Product)))
    return divide(total_sales(context), calculate(total_sales, context, remove=["Category", "Product"]))

def products_without_sales():
    # product names that have no fact rows, sorted
    return []

print("All sales:", total_sales({}))
print("Camping:", total_sales({"Category": "Camping"}))
print("Camping, West:", total_sales({"Category": "Camping", "Region": "West"}))
print("Camping vs all categories (South):", total_sales({"Category": "Camping", "Region": "South"}), "/", calculate(total_sales, {"Category": "Camping", "Region": "South"}, remove=["Category"]))
print("Share of South sales that is Camping:", round(pct_of_region({"Category": "Camping", "Region": "South"}) * 100, 1), "%")
print("CALCULATE override (Hiking in a Camping context):", calculate(total_sales, {"Category": "Camping"}, add={"Category": "Hiking"}))
print("Divide by zero ->", divide(5, 0), "| empty context ->", total_sales({"Category": "Lamps"}))
print("Products with no sales:", products_without_sales())
```

Expected output:

```text
All sales: 1390
Camping: 790
Camping, West: 590
Camping vs all categories (South): 200 / 800
Share of South sales that is Camping: 25.0 %
CALCULATE override (Hiking in a Camping context): 600
Divide by zero -> None | empty context -> 0
Products with no sales: ['Lamp']
```

??? success "Show a solution"

    ```
    # A tiny model of Power BI's filter context: dimensions filter a fact table through relationships.
    FACT = [  # (date, product_key, store_key, units, amount)
        ("2026-01-05", 1, 1, 2, 200), ("2026-01-20", 2, 1, 1, 150), ("2026-02-03", 1, 2, 4, 400),
        ("2026-02-14", 3, 2, 2, 90),  ("2026-03-02", 2, 1, 3, 450), ("2026-03-15", 1, 2, 1, 100),
    ]
    DIM_PRODUCT = {1: {"Product": "Tent", "Category": "Camping"}, 2: {"Product": "Boots", "Category": "Hiking"},
                   3: {"Product": "Stove", "Category": "Camping"}, 4: {"Product": "Lamp", "Category": "Camping"}}
    DIM_STORE = {1: {"Store": "Austin", "Region": "South"}, 2: {"Store": "Denver", "Region": "West"}}

    def rows_in_context(filters):
        """filters = {"Category": "Camping", "Region": "West", "Month": "2026-02"}; no filters means all rows."""
        out = []
        for d, p, s, u, a in FACT:
            attrs = {**DIM_PRODUCT[p], **DIM_STORE[s], "Month": d[:7]}
            if all(attrs.get(k) == v for k, v in filters.items()):
                out.append((d, p, s, u, a))
        return out

    def total_sales(filters):                      # SUM(Sales[Amount])
        return sum(r[4] for r in rows_in_context(filters))

    def divide(n, d, alt=None):                    # DIVIDE(n, d) returns BLANK (None) when d is 0
        return alt if not d else n / d

    def calculate(measure, context, add=None, remove=None):
        """CALCULATE: copy the context, REMOVEFILTERS on the named columns, then set/override filters from `add`."""
        ctx = dict(context)
        for k in (remove or []): ctx.pop(k, None)
        ctx.update(add or {})
        return measure(ctx)

    def pct_of_region(context):                    # DIVIDE([Sales], CALCULATE([Sales], REMOVEFILTERS(Product)))
        return divide(total_sales(context), calculate(total_sales, context, remove=["Category", "Product"]))

    def products_without_sales():                  # a star schema keeps dimension rows with no facts
        sold = {r[1] for r in FACT}
        return sorted(v["Product"] for k, v in DIM_PRODUCT.items() if k not in sold)

    print("All sales:", total_sales({}))
    print("Camping:", total_sales({"Category": "Camping"}))
    print("Camping, West:", total_sales({"Category": "Camping", "Region": "West"}))
    print("Camping vs all categories (South):", total_sales({"Category": "Camping", "Region": "South"}), "/", calculate(total_sales, {"Category": "Camping", "Region": "South"}, remove=["Category"]))
    print("Share of South sales that is Camping:", round(pct_of_region({"Category": "Camping", "Region": "South"}) * 100, 1), "%")
    print("CALCULATE override (Hiking in a Camping context):", calculate(total_sales, {"Category": "Camping"}, add={"Category": "Hiking"}))
    print("Divide by zero ->", divide(5, 0), "| empty context ->", total_sales({"Category": "Lamps"}))
    print("Products with no sales:", products_without_sales())
    ```

    A filter context is just the set of dimension filters applied to the fact table; CALCULATE copies the context, removes or overrides filters, then evaluates the measure. This is a Python model of the behaviour (not DAX itself), verified by running it.

### Date tables, time intelligence and incremental refresh

Model a contiguous date table, TOTALYTD, SAMEPERIODLASTYEAR and a safe growth calculation, then work out which monthly partitions an incremental refresh would reload.

Starter code:

```
from datetime import date, timedelta

def date_table(start, end):
    # every date from start to end inclusive (contiguous - required for time intelligence)
    return []

SALES = {date(2025, 1, 15): 100, date(2025, 2, 10): 120, date(2025, 3, 5): 90,
         date(2026, 1, 12): 130, date(2026, 2, 8): 150, date(2026, 3, 20): 60}

def sales_on(days):                            # SUM over the dates in the filter context
    return sum(SALES.get(d, 0) for d in days)

def ytd(table, as_of):
    # TOTALYTD: sales from Jan 1 of as_of's year up to as_of
    return 0

def same_period_last_year(table, as_of):
    # the same window shifted back one year
    return 0

def growth(now, before):
    # DIVIDE(now - before, before) as a percentage rounded to 1 decimal; None when before is 0
    return None

def month_partitions(start, end):
    # list of "YYYY-MM" strings from start's month to end's month inclusive
    return []

def partitions_to_refresh(all_parts, today, refresh_months):
    # incremental refresh: only the most recent `refresh_months` months up to today's month
    return []

table = date_table(date(2025, 1, 1), date(2026, 12, 31))
print("date table days:", len(table), "| contiguous:", all((b - a).days == 1 for a, b in zip(table, table[1:])))
as_of = date(2026, 2, 28)
now, before = ytd(table, as_of), same_period_last_year(table, as_of)
print("YTD:", now, "| same period last year:", before, "| growth:", growth(now, before), "%")
print("growth with no prior year:", growth(50, 0))
parts = month_partitions(date(2025, 1, 1), date(2026, 3, 1))
print("partitions:", len(parts), parts[0], "..", parts[-1])
print("refresh last 2 months:", partitions_to_refresh(parts, date(2026, 3, 10), 2))
```

Expected output:

```text
date table days: 730 | contiguous: True
YTD: 280 | same period last year: 220 | growth: 27.3 %
growth with no prior year: None
partitions: 15 2025-01 .. 2026-03
refresh last 2 months: ['2026-02', '2026-03']
```

??? success "Show a solution"

    ```
    from datetime import date, timedelta

    # 1. A contiguous date table (required for time intelligence).
    def date_table(start, end):
        d, out = start, []
        while d <= end:
            out.append(d); d += timedelta(days=1)
        return out

    SALES = {date(2025, 1, 15): 100, date(2025, 2, 10): 120, date(2025, 3, 5): 90,
             date(2026, 1, 12): 130, date(2026, 2, 8): 150, date(2026, 3, 20): 60}

    def sales_on(days):                            # SUM over the dates in the filter context
        return sum(SALES.get(d, 0) for d in days)

    def ytd(table, as_of):                         # TOTALYTD: Jan 1 .. as_of
        return sales_on(d for d in table if date(as_of.year, 1, 1) <= d <= as_of)

    def same_period_last_year(table, as_of):       # SAMEPERIODLASTYEAR: shift the YTD window back one year
        start, end = date(as_of.year - 1, 1, 1), as_of.replace(year=as_of.year - 1)
        return sales_on(d for d in table if start <= d <= end)

    def growth(now, before):                       # DIVIDE(now - before, before)
        return None if not before else round((now - before) / before * 100, 1)

    # 2. Incremental refresh: refresh only partitions inside [RangeStart, RangeEnd).
    def month_partitions(start, end):
        parts, y, m = [], start.year, start.month
        while (y, m) <= (end.year, end.month):
            parts.append(f"{y}-{m:02d}"); m += 1
            if m == 13: y, m = y + 1, 1
        return parts

    def partitions_to_refresh(all_parts, today, refresh_months):
        y, m = today.year, today.month
        keep = []
        for _ in range(refresh_months):
            keep.append(f"{y}-{m:02d}"); m -= 1
            if m == 0: y, m = y - 1, 12
        return [p for p in all_parts if p in keep]

    table = date_table(date(2025, 1, 1), date(2026, 12, 31))
    print("date table days:", len(table), "| contiguous:", all((b - a).days == 1 for a, b in zip(table, table[1:])))
    as_of = date(2026, 2, 28)
    now, before = ytd(table, as_of), same_period_last_year(table, as_of)
    print("YTD:", now, "| same period last year:", before, "| growth:", growth(now, before), "%")
    print("growth with no prior year:", growth(50, 0))
    parts = month_partitions(date(2025, 1, 1), date(2026, 3, 1))
    print("partitions:", len(parts), parts[0], "..", parts[-1])
    print("refresh last 2 months:", partitions_to_refresh(parts, date(2026, 3, 10), 2))
    ```

    Time intelligence needs a contiguous date table; YTD and last-year windows are date ranges over it; growth uses a safe divide; incremental refresh reloads only recent partitions. This is a Python model of the behaviour (not DAX or the Service), verified by running it.

---

More free practice: the [Power BI Mastery Path](index.md) curriculum, the [Interview Prep Pack](https://learnmasterypath.com/interview-pack/) and the [free resume ATS check](https://learnmasterypath.com/resume/).
