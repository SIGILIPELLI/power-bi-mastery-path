---
description: "Understand DAX context transition: how CALCULATE and measure references turn a row context into a filter context, the duplicate-row trap, and its cost."
---

# 02 · Context Transition

Context transition is the single DAX mechanism that most often explains "the same formula
gives different answers in different places." It's simple to state:

> When `CALCULATE` (or `CALCULATETABLE`) is evaluated inside a **row context**, it converts
> that row context into an equivalent **filter context** — one filter per column of the
> current row — before evaluating its expression.

And the part people miss: **every measure reference is implicitly wrapped in `CALCULATE`.**
Writing `[Revenue]` inside an iterator is the same as writing `CALCULATE ( <Revenue's
formula> )`, so it triggers context transition too.

Examples use the Level 3 `Sales` table (A/Red 2×10, A/Blue 1×10, B/Red 3×20, C/Blue 5×4,
C/Red 1×4; Revenue 114).

## Example 1: a calculated column, with and without CALCULATE

Add two calculated columns to `Sales`:

```dax
Qty Plain = SUM ( Sales[Qty] )
Qty Transitioned = CALCULATE ( SUM ( Sales[Qty] ) )
```

| Product | Color | Qty | Price | Qty Plain | Qty Transitioned |
|---------|-------|----:|------:|----------:|-----------------:|
| A | Red | 2 | 10 | 12 | 2 |
| A | Blue | 1 | 10 | 12 | 1 |
| B | Red | 3 | 20 | 12 | 3 |
| C | Blue | 5 | 4 | 12 | 5 |
| C | Red | 1 | 4 | 12 | 1 |

- `Qty Plain`: a row context doesn't filter, so `SUM` sees all rows → 12 everywhere.
- `Qty Transitioned`: `CALCULATE` turns the row into filters `Product = "A", Color = "Red",
  Qty = 2, Price = 10`; only that row survives; the sum is 2.

## Example 2: measures inside iterators

```dax
Revenue = SUMX ( Sales, Sales[Qty] * Sales[Price] )

Avg Revenue per Product = AVERAGEX ( VALUES ( Sales[Product] ), [Revenue] )
```

For each product in `VALUES ( Sales[Product] )`, `[Revenue]` triggers context transition: the
row {Product = A} becomes the filter `Sales[Product] = "A"`, and `Revenue` returns 30. Results
30, 60, 24 → average **38**.

Now remove the transition by inlining the formula:

```dax
Avg Revenue per Product (inline) =
AVERAGEX ( VALUES ( Sales[Product] ), SUMX ( Sales, Sales[Qty] * Sales[Price] ) )
```

Inside the outer iterator there's a row context on Product, but the inner `SUMX ( Sales, … )`
isn't affected by it — no `CALCULATE`, no transition. Each of the three iterations returns
114; the average is **114**. Same arithmetic, different answer, purely because of the implicit
`CALCULATE`.

## Example 3: filtering by a measure

"How many products had revenue above 25?"

```dax
Products Over 25 =
COUNTROWS ( FILTER ( VALUES ( Sales[Product] ), [Revenue] > 25 ) )
```

Per product: A 30 ✓, B 60 ✓, C 24 ✗ → **2**. Written without the measure reference:

```dax
Products Over 25 (wrong) =
COUNTROWS (
    FILTER ( VALUES ( Sales[Product] ), SUMX ( Sales, Sales[Qty] * Sales[Price] ) > 25 )
)
```

Every product's condition sees 114 > 25 → **3**. Fix it by wrapping the inner expression in
`CALCULATE ( … )` — which is exactly what the measure reference did for you.

## Example 4: the duplicate-row trap

Add a sixth row identical to the first: A / Red / 2 / 10. Now:

```dax
Revenue via Measure Per Row = SUMX ( Sales, [Revenue] )
```

You might expect it to equal `[Revenue]` (now 134). Trace it:

| Row | Product | Color | Qty | Price | Filter from context transition | [Revenue] under that filter |
|----:|---------|-------|----:|------:|--------------------------------|----------------------------:|
| 1 | A | Red | 2 | 10 | A, Red, 2, 10 → matches rows 1 **and 6** | 40 |
| 2 | A | Blue | 1 | 10 | matches row 2 | 10 |
| 3 | B | Red | 3 | 20 | matches row 3 | 60 |
| 4 | C | Blue | 5 | 4 | matches row 4 | 20 |
| 5 | C | Red | 1 | 4 | matches row 5 | 4 |
| 6 | A | Red | 2 | 10 | matches rows 1 **and 6** | 40 |

Sum: 40 + 10 + 60 + 20 + 4 + 40 = **174**, not 134. Context transition filters by **values**,
not by row identity, so identical rows can't be told apart. Fact tables without a unique key
column are exposed to this. The fixes: don't call measures while iterating a fact table (use
the column expression directly), or include a unique key column in the table.

## Where context transition happens

- Any `CALCULATE` / `CALCULATETABLE` inside a row context.
- Any measure reference inside a row context: iterators (`SUMX`, `FILTER`, `ADDCOLUMNS`,
  `RANKX`…) and calculated columns.
- **Not** in a measure evaluated directly by a visual — there's no row context there, only a
  filter context.

## How It Actually Works

A row context is just "which row am I on" for one table; it has no effect on filters. When
`CALCULATE` begins, it checks for active row contexts. For each one, it takes the current
row and builds a filter that, for **every column of that table** (including columns reachable
via the **expanded table** — the many-to-one related columns), restricts the column to the
row's value. It then combines those filters with the existing filter context (overwriting
filters on the same columns) and evaluates its expression.

Two consequences follow directly:

1. **Correctness depends on uniqueness.** If the combination of column values isn't unique,
   the "one row" filter selects several rows (Example 4).
2. **Cost scales with rows × columns.** Transition on a wide fact table creates a filter on
   every column, for every row iterated. The engine optimizes common cases, but iterating a
   large fact table and calling a measure per row is one of the classic causes of slow DAX,
   often showing up as many small storage-engine queries or a huge datacache in DAX Studio.
   Iterating a small dimension (or `VALUES` of a column) and calling a measure is the
   efficient pattern: few rows, and filters on the dimension key propagate to the fact
   through the relationship.

## Common mistakes

- Inlining a measure's formula into an iterator and "losing" the per-row filter.
- Calling measures inside `SUMX ( FactTable, … )` over millions of rows.
- Assuming a calculated column `= [Some Measure]` returns the grand total — it returns the
  per-row value, by transition.
- Tables without a key, iterated with measure calls → silent double counting.

## Exercise

1. Build the two calculated columns in Example 1 and confirm the table.
2. Reproduce Examples 2 and 3 (38 vs 114; 2 vs 3).
3. Add the duplicate row and confirm `SUMX ( Sales, [Revenue] )` = 174. Then add a
   `LineID` column (1–6) and explain why the result becomes 134.
