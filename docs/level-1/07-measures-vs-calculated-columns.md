---
description: "The key Power BI distinction: calculated columns are computed per row at refresh, measures are computed per filter context at query time. Worked examples."
---

# 07 · Measures vs Calculated Columns

If you remember one idea from Level 1, make it this one. DAX formulas come in two main
forms that look almost identical in the formula bar but run at completely different times
and in completely different ways:

- A **calculated column** is computed **once per row, at refresh**, and stored in the table.
- A **measure** is computed **at query time, once per cell of a visual**, in whatever filter
  context that cell has. Nothing is stored.

## Creating each

- **Calculated column:** select the `Sales` table in the Data pane → **Table tools → New
  column** (or right-click the table → New column).
- **Measure:** select the table you want it to live in → **Home → New measure** (or
  right-click → New measure). Measures are identified in the Data pane by a calculator icon.

## Worked example 1: revenue both ways

Suppose the `Revenue` column did *not* exist. As a calculated column:

```dax
Line Revenue = Sales[Units] * Sales[UnitPrice]
```

Each row gets a value (240, 240, 125, 120, 100, 160, 80, 360), stored in the model.
Summing it in a visual gives 1,425.

As a measure, without any stored column:

```dax
Total Revenue = SUMX ( Sales, Sales[Units] * Sales[UnitPrice] )
```

`SUMX` iterates the rows of `Sales` that are visible in the current filter context,
computes the expression per row, and adds the results. Unfiltered, that's 1,425. In a
visual split by Region, it runs three times — once per region — and returns 445, 700, 280.

Both work. For a simple multiplication on a small table, the calculated column costs
memory but nothing else; on a 100-million-row table, the measure avoids storing a new
column. Level 3 discusses the trade-off properly.

## Worked example 2: where only a measure is correct

We want the **average selling price**: revenue divided by units.

```dax
Total Units = SUM ( Sales[Units] )
Avg Selling Price = DIVIDE ( [Total Revenue], [Total Units] )
```

Hand check, whole table: 1,425 ÷ 21 = **67.857…** (show as 67.86).

By category:

| Category | Revenue | Units | Avg Selling Price |
|----------|---------|-------|-------------------|
| Camping | 720 | 6 | 120.00 |
| Apparel | 480 | 6 | 80.00 |
| Accessories | 225 | 9 | 25.00 |
| **Total** | **1,425** | **21** | **67.86** |

Now try the "column" way: a calculated column `Price Col = Sales[UnitPrice]` averaged in
the visual gives, at total level, the average of the 8 row prices:
(120+80+25+120+25+80+80+120) ÷ 8 = 650 ÷ 8 = **81.25**. That's the unweighted average of
order lines — a different question. And *summing* a ratio column would be worse. The
measure is right because it divides the totals **after** filtering, at each level of the
visual, including the grand total.

`DIVIDE(a, b)` is used instead of `a / b` because it returns BLANK (or an optional
alternate result) when `b` is zero or blank, instead of an error or infinity.

## Worked example 3: where only a column makes sense

We want to put orders into size bands and use the band **on an axis or slicer**:

```dax
Order Size =
SWITCH (
    TRUE (),
    Sales[Revenue] >= 200, "Large",
    Sales[Revenue] >= 100, "Medium",
    "Small"
)
```

Rows: 240 Large, 240 Large, 125 Medium, 120 Medium, 100 Medium, 160 Medium, 80 Small,
360 Large → Large 3 orders, Medium 4, Small 1.

A measure can't be put on an axis or in a slicer, because a measure has no rows — it only
returns a value for a given context. Anything you want to **group by or filter by** must
be a column (calculated in DAX, or better, created in Power Query).

## Decision rule

| You need… | Use |
|-----------|-----|
| A value to aggregate that depends on the filters (totals, ratios, % of total, YTD) | Measure |
| A category to slice/filter/group by | Column (prefer Power Query) |
| A per-row value that later gets summed | Either; prefer Power Query column or a SUMX measure |
| Something that must respond to a slicer selection | Measure |

## How It Actually Works

A calculated column is evaluated during **refresh**, after Power Query has loaded the table.
The engine creates a **row context** — an iteration over every row — and evaluates your
expression once per row. The result is compressed and stored like any imported column. It
never changes until the next refresh, which is why a calculated column **cannot react to
slicers**: slicers act at query time, long after the column was computed.

A measure is stored only as its formula. When a visual asks for `[Avg Selling Price]` by
Category, the engine evaluates it once per category in a **filter context** containing
that category (and any slicers/filters), plus once more for the total with no category
filter. Inside the measure, `SUM(Sales[Units])` has no row context — it aggregates whatever
rows survive the current filters.

That is also why a bare column reference is illegal in a measure: `= Sales[UnitPrice]`
fails with an error about a single value not being determined, because in a filter context
there may be many rows and DAX refuses to guess which one. Inside a calculated column, the
same reference is fine — the row context supplies "the current row." Level 2 names these
two contexts formally and shows how `CALCULATE` connects them.

## Common mistakes

- **Writing ratios as calculated columns** and averaging or summing them in visuals.
- **Creating a calculated column to "store a total"** such as `SUM(Sales[Revenue])` —
  every row gets 1,425, which is useless and costs memory.
- **Putting measures in a random table.** Create a dedicated measures table
  (Home → Enter data with one dummy column, then hide the column) so people can find them.
- **Using `/` instead of `DIVIDE`** and showing `Infinity` or `NaN` on a category with zero
  units.

## Exercise

1. Create `Total Revenue`, `Total Units` and `Avg Selling Price` as measures and reproduce
   the category table above exactly.
2. Create the `Order Size` column and a table with `Order Size` and `[Total Revenue]`.
   Expected: Large 840, Medium 505, Small 80 (hand-check them).
3. Try to create a measure `= Sales[UnitPrice]`. Copy the error message and explain in one
   sentence why it happens.
