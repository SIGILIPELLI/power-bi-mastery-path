---
description: "Level 1 project: build, verify, and publish a one-page Power BI sales dashboard from a 16-row CSV, with every expected number computed by hand."
---

# 10 · Project — A Sales Dashboard from CSV

This project pulls together everything in Level 1: import, clean, add measures, build
visuals, filter, format and publish. The data is small on purpose — 16 orders — so that
you can check **every** number on the finished page against the answers below. A
dashboard you have verified is worth more than a prettier one you haven't.

## The brief

Trailhead's operations lead wants a single page for Q2 2025 that answers:

1. What were gross and net revenue (after discounts), and how many units did we sell?
2. How did revenue move month by month?
3. Which regions and categories drove it?
4. What's the discount doing to us?

## The data

Save as `trailhead_q2.csv` (UTF-8):

```text
OrderID,OrderDate,Region,Product,Category,Units,UnitPrice,Discount
2001,2025-04-02,North,Trail Tent,Camping,1,120.00,0.00
2002,2025-04-05,South,Headlamp,Accessories,6,25.00,0.10
2003,2025-04-11,East,Rain Jacket,Apparel,2,80.00,0.00
2004,2025-04-19,West,Sleeping Bag,Camping,2,95.00,0.05
2005,2025-04-27,North,Water Bottle,Accessories,10,12.00,0.00
2006,2025-05-03,South,Trail Tent,Camping,2,120.00,0.10
2007,2025-05-09,West,Rain Jacket,Apparel,1,80.00,0.00
2008,2025-05-15,East,Headlamp,Accessories,3,25.00,0.00
2009,2025-05-22,North,Sleeping Bag,Camping,1,95.00,0.00
2010,2025-05-30,South,Fleece,Apparel,4,55.00,0.05
2011,2025-06-04,East,Trail Tent,Camping,1,120.00,0.00
2012,2025-06-10,West,Water Bottle,Accessories,8,12.00,0.10
2013,2025-06-16,North,Fleece,Apparel,2,55.00,0.00
2014,2025-06-21,South,Sleeping Bag,Camping,3,95.00,0.05
2015,2025-06-25,East,Fleece,Apparel,1,55.00,0.00
2016,2025-06-29,West,Trail Tent,Camping,2,120.00,0.00
```

`Discount` is a fraction of the line price (0.10 = 10% off).

## Step 1 — Import and shape (Power Query)

1. **Get data → Text/CSV → Transform Data**. Rename the query `Sales`.
2. Set types explicitly (remove the auto step if you prefer and add your own):
   `OrderID` Text (it's an identifier), `OrderDate` Date, `Region`/`Product`/`Category`
   Text, `Units` Whole Number, `UnitPrice` Fixed Decimal, `Discount` Decimal Number.
3. Add two custom columns, with types:

```powerquery
#"Added Gross" = Table.AddColumn(#"Changed Type", "GrossAmount",
    each [Units] * [UnitPrice], Currency.Type),
#"Added Net" = Table.AddColumn(#"Added Gross", "NetAmount",
    each [GrossAmount] * (1 - [Discount]), Currency.Type)
```

4. **Close & Apply**. Data view should show 16 rows.

Why columns in Power Query and not DAX? They are per-row facts that never depend on a
slicer — exactly what the query layer is for.

## Step 2 — Measures

Create an empty table for measures (**Home → Enter data**, name it `_Measures`, load it,
then hide its single column). Add:

```dax
Gross Revenue = SUM ( Sales[GrossAmount] )
Net Revenue   = SUM ( Sales[NetAmount] )
Units Sold    = SUM ( Sales[Units] )
Orders        = COUNTROWS ( Sales )
Discount Cost = [Gross Revenue] - [Net Revenue]
Discount %    = DIVIDE ( [Discount Cost], [Gross Revenue] )
Avg Order Value = DIVIDE ( [Net Revenue], [Orders] )
```

Format: revenue measures and Discount Cost as currency with 2 decimals, Units and Orders
as whole numbers, Discount % as percentage with 1 decimal.

## Step 3 — The expected answers

Compute these before you look at your report. Line amounts (gross → net):

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

Headline numbers:

| Measure | Expected |
|---------|---------:|
| Gross Revenue | 2,356.00 |
| Net Revenue | 2,272.65 |
| Discount Cost | 83.35 |
| Discount % | 3.5% (83.35 ÷ 2,356 = 0.03538) |
| Units Sold | 49 |
| Orders | 16 |
| Avg Order Value | 142.04 (2,272.65 ÷ 16 = 142.040625) |

By month (net): April **715.50**, May **675.00**, June **882.15**.

By region:

| Region | Gross | Net | Units |
|--------|------:|----:|------:|
| South | 895.00 | 830.75 | 15 |
| West | 606.00 | 586.90 | 13 |
| North | 445.00 | 445.00 | 14 |
| East | 410.00 | 410.00 | 7 |

By category:

| Category | Gross | Net | Units |
|----------|------:|----:|------:|
| Camping | 1,290.00 | 1,242.25 | 12 |
| Apparel | 625.00 | 614.00 | 10 |
| Accessories | 441.00 | 416.40 | 27 |

Discount % by region: South 64.25 ÷ 895 = **7.2%**, West 19.10 ÷ 606 = **3.2%**, North and
East **0.0%**. That's the insight the operations lead is after: discounting is
concentrated in the South.

## Step 4 — Build the page

Follow the layout grid from lesson 08 on a 16:9 page:

1. **Title** text box: "Q2 2025 sales — net revenue and discounting".
2. **Four cards** across the top: Net Revenue, Units Sold, Avg Order Value, Discount %.
3. **Line chart** (large, left): `OrderDate` at *Month* level on X, `Net Revenue` and
   `Gross Revenue` on Y. Remove the Year/Quarter levels from the hierarchy or drill down
   to month.
4. **Clustered bar chart** (right): Region on Y, `Net Revenue` on X, `Discount %` in
   Tooltips.
5. **Table** (bottom right): Category, Units Sold, Gross Revenue, Net Revenue, Discount %.
6. **Slicers**: Region (tile/buttons) and Category (dropdown) along the top.
7. **Edit interactions**: clicking a region bar filters the table and the line chart; decide
   deliberately whether it should also filter the cards. This project lets cards filter
   (they answer "for what I've selected"). Write your choice in a small footnote text box.

## Step 5 — Verify with filters

Check these combinations against hand-calculated answers:

| Selection | Net Revenue | Units | Discount % |
|-----------|------------:|------:|-----------:|
| Region = South | 830.75 | 15 | 7.2% |
| Category = Camping | 1,242.25 | 12 | 3.7% (47.75 ÷ 1,290) |
| South + Camping (orders 2006, 2014) | 486.75 | 5 | 7.3% (38.25 ÷ 525) |
| Month = June only (click June on the line) | 882.15 | 17 | 2.6% (23.85 ÷ 906) |

If any row disagrees, find the stage at fault: Power Query (check Data view), measure
definition, or a leftover visual-level filter.

## Step 6 — Format and publish

- Apply your theme from lesson 08 and consistent measure formats.
- Rename visuals in the Selection pane.
- **Publish** to your learning workspace, pin the four cards and the line chart to a
  dashboard named "Q2 Ops", and share the report with one colleague as read-only.

## How It Actually Works

When you click **South** in the Region slicer, Power BI regenerates the DAX query for every
visual on the page with a filter `'Sales'[Region] = "South"`. The engine's **storage
engine** scans only the columns each query needs — `Region`, `NetAmount`, `GrossAmount`,
`Units` — and because each column is stored separately and compressed, a filter on Region
is evaluated against the Region column's small dictionary (four values) before touching
the amount columns. The **formula engine** then evaluates `DIVIDE([Discount Cost], [Gross
Revenue])` on the two aggregated numbers it got back, not on rows. That's why
`Discount %` in the table's total row is 3.5% (computed from totals) and not the average
of the three category percentages (3.70%, 1.76% and 5.58%, which average about 3.7%). Grand totals are separate
evaluations, not sums of the visible rows.

## Common mistakes in this project

- `OrderID` left as a number and summed on a card by accident.
- Computing net as `UnitPrice * (1 - Discount)` and forgetting Units.
- Discount % built as an average of the `Discount` column — gives 0.028, the unweighted average of line
  rates (unweighted), which is not the revenue impact.
- Forgetting the date hierarchy level and seeing a single point for "2025."

## Exercise

1. Complete the dashboard and check all of the verification rows in Step 5.
2. Add a fifth card, `Orders`, and a Product table. Which product has the highest
   Discount Cost? (Work it out by hand first: Trail Tent 24.00, Sleeping Bag 23.75,
   Headlamp 15.00, Fleece 11.00, Water Bottle 9.60, Rain Jacket 0.)
3. Write a three-sentence summary for the operations lead based only on numbers you have
   verified.
