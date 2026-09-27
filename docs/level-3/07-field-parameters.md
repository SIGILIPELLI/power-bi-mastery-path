---
description: "Build Power BI field parameters so report readers can switch the measure or dimension a visual shows, and understand the DAX table behind them."
---

# 07 · Field Parameters

Readers often ask for "the same chart, but by product instead of region" or "the same chart,
but showing units instead of revenue." The old answers — duplicated visuals toggled with
bookmarks, or a disconnected table plus a `SWITCH` measure — work but multiply maintenance.
**Field parameters** let a slicer choose which fields a visual uses.

## Creating a field parameter

1. **Modeling → New parameter → Fields** (the "New parameter" dropdown also has "Numeric
   range," which is a different feature — what-if parameters).
2. Name it `Metric`.
3. Add fields: `[Total Amount]`, `[Total Units]`, `[Avg Price per Unit]` (measures from Level 2).
   Reorder them as you want them to appear.
4. Leave **Add slicer to this page** ticked. Create.

Power BI adds a table `Metric` and a slicer. Build a clustered column chart with
`DimStore[Region]` on the X-axis and the `Metric` **parameter column** (not the individual
measures) on the Y-axis. Choosing *Total Units* in the slicer switches the chart to units.

Expected, Level 2 sample (2025 rows):

| Region | Total Amount | Total Units | Avg Price per Unit |
|--------|-------------:|------------:|-------------------:|
| West | 670 | 7 | 95.71 |
| South | 315 | 6 | 52.50 |
| East | 230 | 7 | 32.86 |

(West units: rows 1, 5, 7 → 2 + 2 + 3 = 7. South: rows 2, 4, 8 → 4 + 1 + 1 = 6. East: rows 3, 6
→ 1 + 6 = 7. 670 ÷ 7 = 95.714…; 230 ÷ 7 = 32.857…)

## Dimension parameters

Create a second parameter `Breakdown` with fields `DimStore[Region]`, `DimProduct[Category]`,
`DimProduct[Product]`. Put `Breakdown` on the X-axis. Now readers can pick both the metric and
the breakdown: "Total Units by Category" gives Camping 6, Apparel 4, Accessories 10.

If the slicer allows **multi-select** on a dimension parameter, the visual shows all selected
fields as a hierarchy you can drill through — handy in a matrix for "build your own
breakdown."

## The DAX behind it

Open the `Metric` table's expression (select the table; the formula bar shows it):

```dax
Metric = {
    ( "Total Amount", NAMEOF ( '_Measures'[Total Amount] ), 0 ),
    ( "Total Units", NAMEOF ( '_Measures'[Total Units] ), 1 ),
    ( "Avg Price per Unit", NAMEOF ( '_Measures'[Avg Price per Unit] ), 2 )
}
```

Each row is (display name, field reference, order). You can edit this expression directly — for
example to rename "Avg Price per Unit" to "Avg price" — which is often faster than recreating the
parameter. The table name where your measures live (`_Measures` here) must match your model.

To show the selected metric's name in a title:

```dax
Chart Title =
"Q1 2025 — " & SELECTEDVALUE ( Metric[Metric], "multiple metrics" ) & " by region"
```

Use it in **Format → Title → fx** (conditional formatting of the title text) → Field value →
`[Chart Title]`.

## Field parameters vs the alternatives

| Approach | Pros | Cons |
|----------|------|------|
| Field parameters | Native, each measure keeps its own format, works for dimensions too | Visual formatting is per visual, not per selected field |
| Disconnected table + `SWITCH` measure | Full control in DAX; works in older tooling | One measure can have only one format string (without dynamic formats); dimensions not supported |
| Bookmarks with duplicated visuals | Anything goes | Maintenance multiplies with every variant |
| Calculation groups | Transform any measure (e.g. YTD) | Different purpose: they *modify* a measure rather than choose one |

Field parameters and calculation groups combine well: a Metric parameter picks *what*, a Time
Calc group picks *which period*.

## How It Actually Works

A field parameter is a **calculated table** with three columns: the display name, a hidden
column containing the field reference (created with `NAMEOF`, which returns the fully qualified
name as text), and a hidden order column. The table carries extended metadata marking it as a
parameter and linking each row to its field.

When a visual uses the parameter column, the report layer — not DAX — reads the slicer's
selection, looks up the referenced fields, and **rewrites the visual's query** to use those
fields directly. With *Total Units* selected, the chart sends exactly the query it would send if
you had dragged `[Total Units]` into the Y-axis yourself. That's why each measure keeps its own
format string, and why performance is the same as a normal visual: no giant `SWITCH` evaluating
branches at query time.

A `SWITCH`-based approach, by contrast, is one measure whose DAX branches on the selection. The
engine can usually short-circuit to the selected branch, but the query plan, format string and
sort behaviour belong to one measure — which is where the limitations come from.

## Common mistakes

- Dragging the individual measures from the parameter table instead of the parameter column.
- Renaming a measure and not updating the `NAMEOF` reference (the parameter row breaks).
- Letting users multi-select a *metric* parameter on a chart that becomes unreadable with three
  measures on different scales.
- Sorting: sort the visual by the parameter's value, not by a specific measure that may not be
  selected.

## Exercise

1. Build `Metric` and `Breakdown` and verify the Region table above and "Total Units by
   Category" (6, 4, 10).
2. Edit the `Metric` expression to add `[Orders]` (`COUNTROWS(FactSales)`) as the fourth option.
   Expected by Region: West 3, South 3, East 2.
3. Add the dynamic chart title and confirm it reads "Q1 2025 — Total Units by region" when Units
   is selected.
