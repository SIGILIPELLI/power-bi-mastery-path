---
description: "Build your first Power BI visuals: choosing chart types, field wells, implicit aggregation, sorting, and the DAX query generated behind each visual."
---

# 05 · Building Your First Visuals

A visual in Power BI is a question sent to the model: "group by these fields, compute these
numbers." The chart type is only the way the answer is drawn. Getting the *question* right
— which fields go where, and how numbers aggregate — matters more than picking a pretty
chart.

## The Report view

- **Canvas** in the middle, with page tabs at the bottom.
- **Visualizations** pane: a gallery of visual types and, below it, the **field wells**
  (called "buckets" in some docs) for the selected visual — for example *X-axis*,
  *Y-axis*, *Legend*, *Tooltips*.
- **Data** pane (formerly "Fields"): your tables and columns.
- **Format** tab inside the Visualizations pane (paint-roller icon) for styling.

Pane names and icons have been reshuffled more than once across monthly releases; if a
pane isn't where this lesson says, look under the **View** ribbon to toggle it.

## Step by step: revenue by region

Assuming the `Sales` table with the `Revenue` column from lesson 03:

1. Click an empty spot on the canvas, then click **Clustered column chart** in the
   Visualizations gallery.
2. Drag `Region` to **X-axis** and `Revenue` to **Y-axis**.
3. The Y-axis well shows "Sum of Revenue" — Power BI applied **implicit aggregation**.
4. Hover a column. Expected values (from the sample):

| Region | Sum of Revenue |
|--------|---------------|
| South | 700 |
| North | 445 |
| East | 280 |

5. Columns are sorted by value descending by default. Use the **More options (…) → Sort
   axis** menu on the visual to switch to alphabetical, and back.

## Picking the right visual

| Question | Good default | Avoid |
|----------|--------------|-------|
| Compare categories | Bar/column chart | Pie with more than ~5 slices |
| Change over time | Line chart (dates on X) | Column chart with 60 months |
| One headline number | Card (or the newer card visual) | A gauge unless there is a real target |
| Exact values to look up | Table or matrix | Charts with data labels on every point |
| Part-to-whole, few parts | Stacked bar, or 100% stacked | 3-D anything |
| Two measures' relationship | Scatter chart | Dual-axis charts with unrelated scales |

Add these three to the page for practice:

- **Card**: `Revenue` → shows **1.43K** by default (display units "Auto"). Set Display
  units to *None* in Format → Callout value to show 1,425.
- **Line chart**: `OrderDate` on X-axis, `Revenue` on Y. Power BI may create a date
  hierarchy (Year → Quarter → Month → Day). Use the **drill** arrows at the top of the
  visual to expand to month level. Expected: Jan 605, Feb 380, Mar 440.
- **Table**: `Category`, `Units`, `Revenue`. Expected: Camping 6 units / 720,
  Apparel 6 / 480, Accessories 9 / 225, total 21 / 1,425.

## The implicit-aggregation trap

Drag `UnitPrice` into the table next to `Category`. Power BI shows "Sum of UnitPrice":
Camping = 120 + 120 + 120 = 360. That number is meaningless — you never sell a tent at
360. Change the aggregation (dropdown on the field in the well → **Average**) and you
get 120, but even that is an unweighted average of rows, not revenue ÷ units.

The fix is not to rely on implicit aggregation for anything that matters. In lesson 07 you
will write explicit measures such as `Avg Selling Price = DIVIDE([Total Revenue],
[Total Units])`. You can also stop the model from summing a column automatically: select
`UnitPrice` in the Data pane and set **Column tools → Summarization → Don't summarize**.
Do the same for `OrderID`, which is a number but should never be added up.

## How It Actually Works

When a visual renders, Power BI generates a **DAX query** and sends it to the local
Analysis Services engine. You can see it: **View → Performance Analyzer → Start
recording → Refresh visuals**, expand the column chart's entry, and click **Copy query**.
Stripped of some wrapping, it looks like this:

```dax
EVALUATE
TOPN(
    1002,
    SUMMARIZECOLUMNS(
        'Sales'[Region],
        "SumRevenue", CALCULATE(SUM('Sales'[Revenue]))
    ),
    [SumRevenue], 0,
    'Sales'[Region], 1
)
```

- `SUMMARIZECOLUMNS` groups by the fields in the axis/legend wells and computes each value
  in the value wells, for every group that has data.
- The implicit "Sum of Revenue" becomes `SUM('Sales'[Revenue])`. It is a hidden measure
  generated on the fly.
- `TOPN(1002, …)` limits the rows returned. Visuals use *data reduction* limits so a chart
  with millions of categories doesn't try to draw them all; tables fetch more rows as you
  scroll.

The engine returns three rows for three regions, and the visual draws three columns.
Totals in tables and matrices come from separate evaluation at the "grand total" level —
not from adding up the displayed rows — which is why a measure can legitimately show a
total that isn't the sum of its rows (an average, a distinct count). Lesson 07 builds on
that.

## Common mistakes

- **Summing columns that aren't additive**: prices, percentages, IDs, ages, balances.
- **Too many categories in a pie or donut.** If you can't read the labels, it's the wrong
  visual.
- **Using the automatic date hierarchy without knowing it.** Power BI creates a hidden
  date table per date column when *Auto date/time* is on (File → Options → Data Load).
  Fine for learning; Level 2 replaces it with a proper date table.
- **Formatting before the numbers are right.** Check totals against a known answer first.

## Exercise

1. Build the four visuals described and confirm every number against the tables above.
2. Use Performance Analyzer to copy the DAX query for your table visual. Identify the
   group-by columns and the aggregations in it.
3. Set `OrderID` and `UnitPrice` to *Don't summarize*, then try to drag `UnitPrice` into a
   card. Describe what the card shows and why.
