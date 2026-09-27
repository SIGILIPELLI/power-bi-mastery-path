---
description: "Master CALCULATE in Power BI: filter context vs row context, how filter arguments replace or add filters, percent-of-total patterns, all traced by hand."
---

# 04 · CALCULATE, Filter Context & Row Context

Almost every "why is my DAX wrong?" question comes down to one of two things: not knowing
which **filters** are active when a measure runs, or not knowing whether there's a **current
row**. This lesson names both contexts, then introduces `CALCULATE` — the only function
that can change the filter context — and traces results by hand on the Level 2 sample.

## Filter context

The **filter context** is the set of filters active when an expression is evaluated. It
comes from:

- the row and column headers of the visual cell (Region = West),
- slicers and Filters-pane filters,
- cross-filtering from other visuals,
- RLS roles,
- and any `CALCULATE` modifications inside the formula.

In a table visual with Region on rows, `[Total Amount]` is evaluated four times: once with
Region = East, once South, once West, and once with no Region filter (the total).

## Row context

A **row context** exists when DAX is iterating a table: inside a calculated column, or
inside the expression argument of an iterator like `SUMX` or `FILTER`. It means "there is a
current row, and column references return its value."

Crucially, **a row context does not filter anything.** Inside a calculated column on
`FactSales`, `SUM ( FactSales[Amount] )` returns 1,215 on every row — the current row
doesn't restrict the SUM. Try it:

```dax
-- Calculated column on FactSales (don't keep it; it's a demonstration)
Wrong Share = FactSales[Amount] / SUM ( FactSales[Amount] )
```

Row 1: 240 ÷ 1,215 = 0.1975. Correct here, but only because the "total" happens to be the
unfiltered total — and it will never respond to a slicer, because it was computed at
refresh.

## CALCULATE

```dax
CALCULATE ( <expression>, <filter1>, <filter2>, … )
```

`CALCULATE` evaluates the expression in a **modified filter context**:

1. Start from the current filter context.
2. For each filter argument, **replace** any existing filter on the same column(s) with the
   new one (unless wrapped in `KEEPFILTERS`, Level 3).
3. Combine filters on different columns with AND.
4. Evaluate the expression.

### Worked example 1: a fixed filter

```dax
Camping Amount = CALCULATE ( [Total Amount], DimProduct[Category] = "Camping" )
```

In a table by Region:

| Region | Total Amount | Camping Amount | Hand check (Camping rows in region) |
|--------|-------------:|---------------:|-------------------------------------|
| East | 230 | *(blank)* | none |
| South | 315 | 215 | row 4 (120) + row 8 (95) |
| West | 670 | 430 | row 1 (240) + row 5 (190) |
| Total | 1,215 | 645 | |

Region filters stay (different column); the Category filter is added.

Now put **Category** on rows instead:

| Category | Total Amount | Camping Amount |
|----------|-------------:|---------------:|
| Accessories | 250 | 645 |
| Apparel | 320 | 645 |
| Camping | 645 | 645 |

Every row shows 645 because the filter argument **replaced** the Category filter coming
from the row header. This surprises everyone once; after that it's the most useful
behaviour in DAX.

### Worked example 2: percent of total

```dax
All Categories Amount = CALCULATE ( [Total Amount], REMOVEFILTERS ( DimProduct[Category] ) )
% of Category Total   = DIVIDE ( [Total Amount], [All Categories Amount] )
```

`REMOVEFILTERS` removes the Category filter; the denominator is 1,215 on every row:

| Category | Total Amount | % of Total |
|----------|-------------:|-----------:|
| Camping | 645 | 53.1% (645 ÷ 1,215) |
| Apparel | 320 | 26.3% |
| Accessories | 250 | 20.6% |

Add a Region slicer set to West: the denominator becomes 670 (Region filter still applies),
and Camping becomes 430 ÷ 670 = **64.2%**. That's "share within the selected region," which
is usually what people mean. To ignore the region too, use `REMOVEFILTERS ( DimProduct )`
plus `REMOVEFILTERS ( DimStore )`, or `ALL ( FactSales )` — Level 3 compares these.

### Worked example 3: table filters with FILTER

Boolean filters like `DimProduct[Category] = "Camping"` are shorthand for a table filter on
one column. To filter on a *measure* or on several columns with logic, use `FILTER`:

```dax
Big Sale Amount =
CALCULATE (
    [Total Amount],
    FILTER ( FactSales, FactSales[Amount] >= 150 )
)
```

Rows with Amount ≥ 150: 240, 190, 150, 240 → **820** in total. By Region: West 240 + 190 +
240 = 670, East 150, South none (blank).

Prefer filtering a **column** over a whole table when you can — `FILTER ( ALL (
FactSales[Amount] ), FactSales[Amount] >= 150 )` or simply the Boolean form
`FactSales[Amount] >= 150` — because filtering the whole fact table applies a filter on
every column of it, which interacts with other filters in ways that are easy to get wrong
and costs more. The Boolean form is the idiomatic choice here:

```dax
Big Sale Amount = CALCULATE ( [Total Amount], FactSales[Amount] >= 150 )
```

## Row context + CALCULATE = context transition (preview)

What happens when `CALCULATE` runs *inside* a row context — for example, when a calculated
column on `DimProduct` references a measure?

```dax
-- Calculated column on DimProduct
Product Amount = [Total Amount]
```

Result: Trail Tent 360 (240 + 120), Rain Jacket 320, Headlamp 250, Sleeping Bag 285
(190 + 95), Camp Stove blank. Each row got its *own* total, not 1,215. That's because a
measure reference is implicitly wrapped in `CALCULATE`, and `CALCULATE` turns the current
row into a filter. This is **context transition**, and Level 3 gives it a full lesson.

## How It Actually Works

Think of filter context as a set of **column filters**, each a list of allowed values. The
visual cell "Region = West" is the filter `{ DimStore[Region] : {"West"} }`.

`CALCULATE` works in a strict order:

1. It evaluates its **filter arguments first**, in the *original* filter context. Each
   Boolean filter is expanded into a table — `DimProduct[Category] = "Camping"` becomes
   `FILTER ( ALL ( DimProduct[Category] ), DimProduct[Category] = "Camping" )`. Note the
   `ALL`: that's precisely why it ignores the existing Category filter.
2. If there's a row context, it performs **context transition**: every column of the
   current row becomes a filter.
3. It applies modifiers (`REMOVEFILTERS`, `USERELATIONSHIP`, `CROSSFILTER`, `KEEPFILTERS`).
4. It merges the new filters into the context: for each column mentioned, the new filter
   **overwrites** the old one; untouched columns keep their filters.
5. It evaluates the expression in the resulting context, which then propagates across
   relationships to the fact table as in lesson 02.

Every number on a report is this machinery running once per cell.

## Common mistakes

- Expecting a filter argument to **intersect** with the visual's filter on the same column.
  It replaces it (use `KEEPFILTERS` to intersect).
- Using `FILTER ( FactSales, … )` for a simple column condition.
- Believing a row context filters aggregates inside a calculated column.
- Putting `CALCULATE` around everything "just in case." It's not free, and it can trigger
  context transition you didn't want.

## Exercise

1. Create `Camping Amount` and reproduce both tables (by Region, then by Category).
2. Create `% of Category Total`, then set the Region slicer to South and compute by hand the
   three percentages before checking. (South: Camping 215, Accessories 100, Apparel 0 of 315
   → 68.3%, 31.7%, blank.)
3. Create the `Product Amount` calculated column on `DimProduct` and verify Trail Tent 360.
   Then create `Wrong Product Amount = SUM ( FactSales[Amount] )` as another column and
   explain why every row shows 1,215.
