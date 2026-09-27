---
description: "Use DAX iterators precisely — SUMX, AVERAGEX, MAXX, RANKX, TOPN, FILTER — and variables, with every iteration traced by hand on a five-row table."
---

# 01 · Iterators & Variables

An **iterator** is any DAX function that walks a table row by row, evaluates an expression
for each row, and combines the results. You met `SUMX` in Level 2. This lesson treats the
iterator family systematically — what table is iterated, what expression runs per row, how
results combine — and shows how variables make iterator-heavy measures readable and correct.

All examples use the five-row `Sales` table from the [Level 3 overview](index.md)
(Revenue total 114; A 30, B 60, C 24).

## Anatomy of an iterator

```dax
SUMX ( <table>, <expression> )
```

1. Evaluate `<table>` in the current filter context.
2. For each row, create a **row context** and evaluate `<expression>`.
3. Sum the results.

The key design question is always: **what table am I iterating?** Iterating `Sales` (5 rows)
and iterating `VALUES ( Sales[Product] )` (3 rows) answer different questions.

## Worked examples

### SUMX over the fact table

```dax
Revenue = SUMX ( Sales, Sales[Qty] * Sales[Price] )
```

Rows: 2×10 = 20, 1×10 = 10, 3×20 = 60, 5×4 = 20, 1×4 = 4 → **114**.

### Two different averages

```dax
Avg Line Revenue = AVERAGEX ( Sales, Sales[Qty] * Sales[Price] )
Avg Price per Unit = DIVIDE ( [Revenue], SUM ( Sales[Qty] ) )
```

- `Avg Line Revenue` = 114 ÷ 5 lines = **22.8**.
- `Avg Price per Unit` = 114 ÷ 12 units = **9.5**.

Both are "averages"; they answer different questions. Name measures so readers know which.

### Iterating a column of distinct values

```dax
Avg Revenue per Product = AVERAGEX ( VALUES ( Sales[Product] ), [Revenue] )
```

`VALUES ( Sales[Product] )` returns {A, B, C}. For each, `[Revenue]` is evaluated **for that
product** — 30, 60, 24 — and averaged: 114 ÷ 3 = **38**. Why does `[Revenue]` return the
product's value and not 114? Because referencing a measure inside an iterator triggers
**context transition**: the current row (Product = A) becomes a filter. Lesson 02 is all
about this.

```dax
Best Product Revenue = MAXX ( VALUES ( Sales[Product] ), [Revenue] )   -- 60
```

### RANKX

```dax
Product Rank = RANKX ( ALL ( Sales[Product] ), [Revenue] )
```

In a table by Product: `RANKX` iterates **all** products (ignoring the row's filter, thanks to
`ALL`), evaluates `[Revenue]` for each (30, 60, 24), then evaluates `[Revenue]` in the current
cell (e.g. A = 30) and returns its position in descending order: B **1**, A **2**, C **3**.

Without `ALL`, the table iterated would be only the current product, and every row would be
rank 1 — the most common RANKX bug. At the total row, `[Revenue]` = 114, which is higher than
any product, so the total shows **1**; hide it with
`IF ( ISINSCOPE ( Sales[Product] ), RANKX ( … ) )`.

Ties: `RANKX` defaults to `SKIP` (1, 2, 2, 4); pass `DENSE` as the fifth argument for
(1, 2, 2, 3).

### TOPN inside an iterator

```dax
Top 2 Product Revenue =
SUMX ( TOPN ( 2, VALUES ( Sales[Product] ), [Revenue] ), [Revenue] )
```

`TOPN` returns the rows for B (60) and A (30); `SUMX` adds them: **90**. `TOPN` may return more
than N rows when there are ties at the boundary.

### FILTER is an iterator too

```dax
Revenue from Big Lines =
CALCULATE ( [Revenue], FILTER ( Sales, Sales[Qty] * Sales[Price] > 15 ) )
```

`FILTER` evaluates the condition per row: 20 ✓, 10 ✗, 60 ✓, 20 ✓, 4 ✗. Remaining lines total
20 + 60 + 20 = **100**. By color: Red 80 (20 + 60), Blue 20.

## Variables

```dax
Revenue Share of Total =
VAR CurrentRevenue = [Revenue]
VAR TotalRevenue   = CALCULATE ( [Revenue], REMOVEFILTERS ( Sales ) )
RETURN
    DIVIDE ( CurrentRevenue, TotalRevenue )
```

By product: A 30 ÷ 114 = **26.3%**, B **52.6%**, C **21.1%**.

Variables give you:

- **Readability** — each step has a name.
- **Single evaluation** — `CurrentRevenue` is computed once even if used several times.
- **Debugging** — temporarily `RETURN TotalRevenue` to inspect an intermediate value.

### The variable trap

A variable is a **constant** once defined. This does *not* work:

```dax
Wrong Share =
VAR CurrentRevenue = [Revenue]
RETURN
    DIVIDE ( CurrentRevenue, CALCULATE ( CurrentRevenue, REMOVEFILTERS ( Sales ) ) )
```

`CALCULATE ( CurrentRevenue, … )` changes the filter context, but `CurrentRevenue` was already
computed (30 for A) — it doesn't re-evaluate. Result: 30 ÷ 30 = **100%** on every row. Put the
*expression* inside `CALCULATE`, not a variable holding its result.

### Variables inside iterators

Variables can be defined inside the per-row expression; they're then evaluated per row:

```dax
Revenue with Volume Discount =
SUMX (
    Sales,
    VAR LineValue = Sales[Qty] * Sales[Price]
    VAR Rate = IF ( Sales[Qty] >= 3, 0.10, 0 )
    RETURN LineValue * ( 1 - Rate )
)
```

Rows: 20, 10, 60 × 0.9 = 54, 20 × 0.9 = 18, 4 → **106**.

## How It Actually Works

The storage engine (VertiPaq) can execute an iterator **by itself** when the per-row expression
uses only columns of the iterated table and simple arithmetic. `SUMX ( Sales, Sales[Qty] *
Sales[Price] )` becomes a single storage-engine scan computing `SUM ( Qty * Price )` over the
compressed columns — multi-threaded and fast. You can see this in DAX Studio's *Server Timings*
as one xmSQL query with the expression inside it.

When the expression contains something the storage engine can't do — `IF` with complex
branches, a measure reference (context transition), `RELATED` across many tables in some
shapes — the formula engine takes over. It asks the storage engine for a **datacache** (an
uncompressed, in-memory intermediate table) with the needed columns, then iterates it itself,
single-threaded. The volume-discount measure above may compile into a storage-engine
expression with a callback (`CallbackDataID`) for the `IF`, meaning the storage engine calls
back into the formula engine per row — slower, and not cached as effectively.

So the cost of an iterator depends on **(a) the number of rows iterated** and **(b) whether the
expression can be pushed down**. Iterating `VALUES ( Sales[Product] )` (hundreds of rows) and
calling a measure is fine; iterating a 100-million-row fact table and calling a measure per row
is not, because each call means context transition over a large table.

## Common mistakes

- **RANKX without ALL** — every row ranks 1.
- **Averaging the wrong grain** — AVERAGEX over lines when you meant over products.
- **Using variables as if they were re-evaluated** inside a later `CALCULATE`.
- **FILTER over a whole fact table** when a column predicate would do.

## Exercise

1. Create every measure in this lesson and check each hand-computed number.
2. Write `Worst Product = MINX ( VALUES ( Sales[Product] ), [Revenue] )` (expected 24) and a
   measure returning the *name* of the best product, using `TOPN ( 1, … )` with `MAXX` or
   `CONCATENATEX`. Expected: `B`.
3. Change row 3's Qty from 3 to 2 and recompute by hand: Revenue, Product Rank, and
   Revenue with Volume Discount. (Answers: 94; B 1 with 40, A 2, C 3; volume discount total
   20 + 10 + 40 + 18 + 4 = 92.)
