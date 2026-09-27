---
description: "Power BI capstone: design, build, secure, test, version and ship an enterprise sales analytics solution with a verified test suite."
---

# 10 · Capstone — An Enterprise Sales Analytics Solution

The capstone asks you to act as the BI lead for Trailhead: take the pieces built across the course
and deliver them the way an organization would — a shared, secured, tested semantic model under
source control, thin reports designed for specific readers, a refresh strategy that scales, and
documentation someone else could operate. The data is still small enough to verify by hand; the
practices are the ones you'd use on a billion rows.

## The brief

- **Readers**: the executive team (company view), three regional managers (own region only), and
  finance analysts (build their own analyses in Excel).
- **Questions**: Are we on target this quarter? Which regions and categories are behind? How do we
  compare with last year?
- **Constraints**: one definition of sales and target; managers must never see other regions'
  sales *or* targets; the model must support five years of history eventually; changes go through
  review.

## Deliverables

1. Architecture diagram and a one-page design document.
2. A semantic model saved as **PBIP** in a Git repository.
3. Two thin reports: *Executive Overview* and *Regional Detail* (with drill-through).
4. RLS (dynamic), tested for each persona.
5. A test suite (DAX queries) with expected values and a reconciliation note.
6. An incremental refresh policy (configured if you have a SQL source; otherwise specified).
7. A release checklist and a deployment plan (pipeline stages or equivalent).
8. An accessibility check record.

## Data

Use the Level 2 tables — `DimProduct`, `DimStore`, the eight 2025 `FactSales` rows — plus the three
2024 rows from Level 2, lesson 05 (Jan 380 West, Feb 400 South, Mar 290 East). Replace the
company-wide `FactTarget` with a regional one, allocating each category-month target 50% West,
30% South, 20% East (the Level 2 project exercise). Add:

- `DimRegion` (West, South, East) related 1:* to `DimStore[Region]` and to `FactTarget[Region]`.
- `DimCategory` related 1:* to `DimProduct[Category]` and `FactTarget[Category]`.
- `Date` (2024–2025), marked, related to `FactSales[OrderDate]` and `FactTarget[MonthStart]`.
- `SecurityRegion` (Maria → West, South; Dev → East; plus an executive group handled by a separate
  role with no filter).

Move the RLS filter to `DimRegion` so it reaches **both** facts — solving the limitation you found in
the Level 2 project.

## Step 1 — Design document (write before building)

Cover, briefly: readers and questions (lesson 04); layers and where each piece of logic lives
(lesson 01); storage mode and refresh (Import with incremental refresh on `FactSales`; targets are
small and fully refreshed); security design (roles, what each persona sees, who has Build
permission); lifecycle (PBIP + Git, pipeline stages); tests; ownership and support.

## Step 2 — Build the model

Measures (in a `_Measures` table, with descriptions and display folders):

```dax
Sales Amount      = SUM ( FactSales[Amount] )
Target Amount     = SUM ( FactTarget[Target] )
Variance          = [Sales Amount] - [Target Amount]
Achievement %     = DIVIDE ( [Sales Amount], [Target Amount] )
Sales PY          = CALCULATE ( [Sales Amount], SAMEPERIODLASTYEAR ( 'Date'[Date] ) )
Sales YoY %       = DIVIDE ( [Sales Amount] - [Sales PY], [Sales PY] )
```

Optionally replace the PY/YoY measures with the `Time Calc` calculation group from Level 3 and add a
`Metric` field parameter for the Regional Detail page.

Model hygiene: hide keys and raw numeric columns, set `summarizeBy: none` on keys, sort month names,
turn off Auto date/time, format every measure.

## Step 3 — Expected results (your test oracle)

Company, Q1 2025:

| Measure | Expected |
|---------|---------:|
| Sales Amount | 1,215 |
| Target Amount | 1,460 |
| Achievement % | 83.2% |
| Sales PY | 1,070 |
| Sales YoY % | 13.6% (145 ÷ 1,070 = 0.1355) |

By region (targets allocated 50/30/20 of 1,460):

| Region | Sales | Target | Achievement % | Sales PY | YoY % |
|--------|------:|-------:|--------------:|---------:|------:|
| West | 670 | 730 | 91.8% | 380 | 76.3% |
| South | 315 | 438 | 71.9% | 400 | −21.3% |
| East | 230 | 292 | 78.8% | 290 | −20.7% |

Hand checks: 670 ÷ 730 = 0.9178; 315 ÷ 438 = 0.7192; 230 ÷ 292 = 0.7877; (670 − 380) ÷ 380 = 0.7632;
(315 − 400) ÷ 400 = −0.2125 (exactly −21.25%; at one decimal the display may read −21.3% or
−21.2% depending on floating-point representation, so test it with a tolerance);
(230 − 290) ÷ 290 = −0.2069.

Personas:

| Persona | Sales | Target | Achievement % |
|---------|------:|-------:|--------------:|
| Executive (no filter role) | 1,215 | 1,460 | 83.2% |
| Maria (West + South) | 985 | 1,168 | 84.3% |
| Dev (East) | 230 | 292 | 78.8% |

(985 ÷ 1,168 = 0.8433.) Note that Maria's targets are now secured too: 730 + 438 = 1,168.

## Step 4 — Reports

**Executive Overview** (thin report on the published model):

- Finding-style title driven by a measure, e.g. `"Q1 sales " & FORMAT ( 1 - [Achievement %], "0%" )
  & " below target"` — which reads "Q1 sales 17% below target" for the company view.
- KPI row with comparisons (Sales vs Target, YoY %).
- Achievement by region (bars, sorted), sales vs target by month.
- Data-as-of card.

**Regional Detail**:

- Region slicer (managers only see their own regions anyway).
- Category × month matrix with conditional formatting plus icons.
- Drill-through to Category Details.
- Field parameter to switch Sales / Target / Variance.

Apply accessibility practices from lesson 08: tab order, measure-driven alt text, contrast-checked theme,
no colour-only encoding.

## Step 5 — Tests

Build a DAX test query (lesson 09) covering every number in Step 3 that doesn't depend on a role, plus
structural tests (no orphaned keys; unique dimension keys). Run the persona tests with **View as** (or a
role-specific connection in DAX Studio) and record the results in a table: persona, expected, actual,
pass. Write one reconciliation note: which source figure you'd compare `Sales Amount` for March 2025
(335) against, and the agreed tolerance.

## Step 6 — Source control and lifecycle

1. Save as PBIP; create the repository with a README (grain, relationships, measure list, RLS design,
   refresh plan, test instructions).
2. Commit on `main`; create a feature branch; change one measure (e.g. zero-fill Achievement %); open a
   pull request whose description lists the expected changes in the test oracle.
3. Add a CI lint (lesson 07) that fails if a measure lacks a format string.
4. Describe (or configure) the pipeline: Dev ← Git sync; Test with parameter rules; Prod with an app for
   readers. List what must be configured per stage (credentials, refresh, role membership, app).

## Step 7 — Incremental refresh specification

For `FactSales`: `RangeStart`/`RangeEnd` filter on `OrderDate` (folded), archive 5 years, refresh the
last 2 months, detect data changes on `LastModified` if available. Explain what happens to a correction
posted to a 14-month-old order (it's in an archived partition: plan a targeted partition refresh).

## Release checklist

- [ ] All tests pass in Test; results attached to the release.
- [ ] Persona tests pass (executive, Maria, Dev, unknown user sees nothing).
- [ ] Impact analysis reviewed; downstream owners notified of breaking changes.
- [ ] Refresh succeeded in Test with production-like credentials.
- [ ] Accessibility checks recorded.
- [ ] Endorsement status reviewed (promote/certify per the lesson 03 checklist).
- [ ] Rollback plan: redeploy the previous Git tag.

## How It Actually Works

It's worth tracing one number end to end, because every layer of the course participates. When Maria
opens *Regional Detail* and hovers over **South / Camping / February**:

1. The thin report's visual sends a DAX query over its live connection to the certified model.
2. The service connects as Maria's identity with the `Region Managers` role. The role's filter on
   `DimRegion` has already been evaluated for her session: {West, South}.
3. The query's own filters — Region = South (from the matrix row), Category = Camping, the February
   `Year Month` — are combined with the security filter by the formula engine.
4. Filters propagate through relationships: `DimRegion → DimStore → FactSales` (store Austin), `DimCategory
   → DimProduct → FactSales` (products 1, 4, 5), `Date → FactSales` (February dates); and `DimRegion`,
   `DimCategory`, `Date → FactTarget`.
5. The storage engine scans the compressed `FactSales[Amount]` column for surviving rows — row 4 (Trail
   Tent, Austin, 2025-02-04): **120** — and `FactTarget[Target]` for South/Camping/February: 30% of 300 =
   **90**.
6. The formula engine computes `DIVIDE ( 120, 90 )` = **133.3%** and returns it with the measure's format
   string.

Six mechanisms from four levels, in a few milliseconds, for one cell. If any one of them is configured
wrongly, the tests in Step 5 are what tell you.

## Exercise

1. Complete all seven steps and the release checklist. Keep every artifact in the repository.
2. Verify the hover example above in your model: South / Camping / February should show Sales 120,
   Target 90, Achievement 133.3%.
3. Write a one-page handover note for the person who will run this solution next year: what to monitor,
   how to add a new region, how to add a measure safely, and whom to contact.
