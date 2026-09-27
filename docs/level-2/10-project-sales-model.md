---
description: "Level 2 project: a verified Power BI star schema with sales and targets at two grains, a date table, time intelligence, drill-through and RLS."
---

# 10 · Project — A Multi-Table Sales Model

This project turns the Level 2 sample into a model you could hand to a colleague: two fact
tables at different grains, shared dimensions, a marked date table, time-intelligence
measures, a drill-through page, and dynamic row-level security. As in Level 1, every
expected number is worked out by hand so you can prove the model is right.

## The brief

Sales leadership wants to compare **actual sales with monthly targets set per category**,
see progress year-to-date, drill into a category, and give each regional manager a
filtered view. Targets are set at **category × month**; sales are recorded per order line.
Two grains — so two fact tables.

## Tables

Start from the Level 2 `DimProduct`, `DimStore` and the eight 2025 `FactSales` rows (see the
[Level 2 overview](index.md)). Add:

**DimCategory**

| Category |
|----------|
| Camping |
| Apparel |
| Accessories |

**FactTarget** (grain: one row per category per month; `MonthStart` is the first day of the month)

| MonthStart | Category | Target |
|------------|----------|-------:|
| 2025-01-01 | Camping | 250 |
| 2025-01-01 | Apparel | 100 |
| 2025-01-01 | Accessories | 80 |
| 2025-02-01 | Camping | 300 |
| 2025-02-01 | Apparel | 100 |
| 2025-02-01 | Accessories | 100 |
| 2025-03-01 | Camping | 300 |
| 2025-03-01 | Apparel | 150 |
| 2025-03-01 | Accessories | 80 |

**SecurityRegion** — as in lesson 08 (Maria: West + South; Dev: East).

**Date** — the DAX date table from lesson 05, covering 2025 (`CALENDAR(DATE(2025,1,1),
DATE(2025,12,31))` is enough here), with `Year Month` sorted by `Year Month Number`.

## Step 1 — Relationships

| From (one) | To (many) | Direction |
|------------|-----------|-----------|
| `DimCategory[Category]` | `DimProduct[Category]` | Single |
| `DimCategory[Category]` | `FactTarget[Category]` | Single |
| `DimProduct[ProductKey]` | `FactSales[ProductKey]` | Single |
| `DimStore[StoreKey]` | `FactSales[StoreKey]` | Single |
| `Date[Date]` | `FactSales[OrderDate]` | Single |
| `Date[Date]` | `FactTarget[MonthStart]` | Single |

Mark `Date` as a date table. Hide all key columns and the `SecurityRegion` table from report
view. `DimCategory → DimProduct → FactSales` is a small snowflake hop; that's acceptable here
because it lets one Category slicer filter both facts.

Design note: `FactTarget` has **no store dimension**, so a Region filter can't reach it. Decide
what that means for the business (next step).

## Step 2 — Measures

```dax
Sales Amount = SUM ( FactSales[Amount] )
Target Amount = SUM ( FactTarget[Target] )
Variance = [Sales Amount] - [Target Amount]
Achievement % = DIVIDE ( [Sales Amount], [Target Amount] )

Sales YTD  = CALCULATE ( [Sales Amount],  DATESYTD ( 'Date'[Date] ) )
Target YTD = CALCULATE ( [Target Amount], DATESYTD ( 'Date'[Date] ) )
Achievement YTD % = DIVIDE ( [Sales YTD], [Target YTD] )

Target (hide when region filtered) =
IF ( ISFILTERED ( DimStore ), BLANK (), [Target Amount] )
```

The last measure answers the design note: targets are company-wide per category, so when a
region is selected we show no target rather than a misleading full-company one. (Another
valid design is to allocate targets to regions — but that's a business decision, not a DAX
one.)

## Step 3 — Expected results

Actual sales by category × month (from the eight fact rows):

| Category | Jan | Feb | Mar | Total |
|----------|----:|----:|----:|------:|
| Camping | 240 | 310 | 95 | 645 |
| Apparel | 80 | — | 240 | 320 |
| Accessories | 100 | 150 | — | 250 |
| **Total** | **420** | **460** | **335** | **1,215** |

Against targets:

| Category | Sales | Target | Variance | Achievement % |
|----------|------:|-------:|---------:|--------------:|
| Camping | 645 | 850 | −205 | 75.9% |
| Apparel | 320 | 350 | −30 | 91.4% |
| Accessories | 250 | 260 | −10 | 96.2% |
| **Total** | **1,215** | **1,460** | **−245** | **83.2%** |

By month:

| Month | Sales | Target | Achievement % | Sales YTD | Target YTD | Achievement YTD % |
|-------|------:|-------:|--------------:|----------:|-----------:|------------------:|
| 2025-01 | 420 | 430 | 97.7% | 420 | 430 | 97.7% |
| 2025-02 | 460 | 500 | 92.0% | 880 | 930 | 94.6% |
| 2025-03 | 335 | 530 | 63.2% | 1,215 | 1,460 | 83.2% |

Hand checks: 645 ÷ 850 = 0.7588; 880 ÷ 930 = 0.9462; 335 ÷ 530 = 0.6321.

Note that Apparel in February shows sales **blank** but target 100. In a matrix, the row
appears because *Target* is non-blank; Achievement % is `DIVIDE(BLANK, 100)` = BLANK. If the
business wants to see 0% there, use `DIVIDE ( [Sales Amount] + 0, [Target Amount] )` on
that measure only.

## Step 4 — Report pages

**Overview page**

- Cards: Sales YTD, Target YTD, Achievement YTD %.
- Line and clustered column chart: `Date[Year Month]` on X, Sales Amount and Target Amount as
  columns, Achievement % as the line.
- Matrix: `DimCategory[Category]` on rows, `Date[Year Month]` on columns, Sales Amount and
  Achievement % as values.
- Slicers: `DimCategory[Category]`, `DimStore[Region]`.

**Category Details page (drill-through)**

- Drill-through field: `DimCategory[Category]`.
- Table of `DimProduct[Product]`, `DimStore[Store]`, `Date[Year Month]`, Sales Amount.
- Card with `SELECTEDVALUE ( DimCategory[Category] )` as the page title.

Verification: drill through on Camping → rows Trail Tent / Denver / 2025-01 / 240,
Trail Tent / Austin / 2025-02 / 120, Sleeping Bag / Denver / 2025-02 / 190, Sleeping Bag /
Austin / 2025-03 / 95; total 645.

## Step 5 — Dynamic RLS

Create the role `Region Managers` exactly as in lesson 08 (filter on `DimStore` and on
`SecurityRegion`). Test **View as → Other user = maria@contoso.com + Region Managers**:

| Measure | Expected for Maria |
|---------|-------------------:|
| Sales Amount | 985 |
| Camping sales | 645 (430 West + 215 South) |
| Apparel sales | 240 |
| Accessories sales | 100 |
| Target Amount | 1,460 (targets aren't region-secured) |

That last row is a real finding: RLS on `DimStore` doesn't reach `FactTarget`, so Maria sees
company-wide targets next to her regional sales. Is that acceptable? Probably not in the
Achievement % measure. Options: switch visuals to `Target (hide when region filtered)` —
but `ISFILTERED ( DimStore )` looks at *query* filters (slicers, visual filters), and a
role's security filter is not a query filter, so it won't fire for Maria. A tempting
alternative is to compare visible stores with all stores:

```dax
Target (secured) =
VAR VisibleStores = COUNTROWS ( DimStore )
VAR AllStores = CALCULATE ( COUNTROWS ( DimStore ), REMOVEFILTERS ( DimStore ) )
RETURN IF ( VisibleStores = AllStores, [Target Amount] )
```

For Maria, `VisibleStores` = 2 and `AllStores` = 2 as well — because `REMOVEFILTERS` can't
remove a security filter. So even this doesn't work! The honest conclusion: if targets must
be secured, they need a **store or region column** so RLS can filter them. Add `Region` to
`FactTarget` (allocating targets) and relate it through a `DimRegion` table. Write this
limitation up rather than papering over it — that's what the exercise asks.

## Step 6 — Refresh plan (written)

Document: where each table comes from (targets from a finance-owned SharePoint list, sales
from the warehouse), whether a gateway is needed, the refresh schedule, the owner who gets
failure notifications, and the "Data as of" card from lesson 09.

## How It Actually Works

Selecting **Camping** in the Category slicer filters `DimCategory`. The engine propagates that
filter down two paths: `DimCategory → FactTarget` directly (targets 250, 300, 300) and
`DimCategory → DimProduct → FactSales` (ProductKeys 1, 4, 5 → rows 1, 4, 5, 8). Each measure
is evaluated against its own fact table with the filter that reached it, and `DIVIDE`
combines the two scalars. The two facts are never joined to each other — which is exactly why
different grains coexist happily: they only meet in the formula engine as aggregated numbers
under a shared filter context.

The security filter works differently from the slicer in one crucial way: it's applied to the
model **before** the query runs, so from inside DAX it's indistinguishable from "these are all
the stores that exist." `REMOVEFILTERS` and `ALL` operate on query filters, not on the rows the
role has hidden. That's the mechanism behind the Step 5 surprise.

## Exercise

1. Build the model and reproduce every table in Step 3, the drill-through check in Step 4,
   and Maria's numbers in Step 5.
2. Extend `FactTarget` with a `Region` column (allocate each category target 50/30/20 to
   West/South/East), add a `DimRegion` table related to both `DimStore[Region]` and
   `FactTarget[Region]`, and move the RLS filter to `DimRegion`. Recompute Maria's Target
   Amount by hand: 80% of 1,460 = **1,168**.
3. Write a half-page README for the model: grain of each fact, relationships, measure
   definitions, RLS design and its limitations, and the refresh plan.
