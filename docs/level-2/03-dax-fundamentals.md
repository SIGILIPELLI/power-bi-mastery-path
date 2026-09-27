---
description: "DAX fundamentals for Power BI: syntax, aggregators vs iterators, COUNTROWS and DISTINCTCOUNT, BLANK handling, RELATED, variables, and formatting measures."
---

# 03 · DAX Fundamentals

DAX (Data Analysis Expressions) looks like Excel formulas, and that resemblance is both
helpful and misleading. Helpful because `SUM`, `IF` and `DIVIDE` behave as you'd expect.
Misleading because DAX doesn't operate on cells — it operates on **tables and columns
under a filter context**. This lesson covers the building blocks; the next one covers
context in depth.

## Syntax essentials

```dax
Total Amount = SUM ( FactSales[Amount] )
```

- `Total Amount` — the measure name. Reference measures in brackets without a table:
  `[Total Amount]`.
- `FactSales[Amount]` — a column, always written with its table: `'Table Name'[Column]`
  (quotes needed if the table name has spaces).
- Comments: `--` or `//` for a line, `/* … */` for blocks.
- Use a line break and indentation for anything longer than one function. DAX formatters
  (such as the one built into DAX Studio, or daxformatter.com) put each argument on its
  own line — readable DAX is debuggable DAX.

Convention used in this course: **always qualify columns with their table, never qualify
measures.** Then you can tell at a glance which is which.

## Aggregators

| Function | Returns | Sample result (Level 2 data) |
|----------|---------|-----|
| `SUM(FactSales[Amount])` | Sum of a column | 1,215 |
| `AVERAGE(FactSales[Amount])` | Mean of non-blank values | 1,215 ÷ 8 = 151.875 |
| `MIN` / `MAX(FactSales[Amount])` | Smallest / largest | 80 / 240 |
| `COUNTROWS(FactSales)` | Rows in a table | 8 |
| `DISTINCTCOUNT(FactSales[ProductKey])` | Distinct values (blank counts as a value) | 4 |
| `COUNTBLANK`, `COUNT`, `COUNTA` | Variants that treat blanks differently | — |

Prefer `COUNTROWS(Table)` to `COUNT(Table[Column])` for "how many rows" — it states intent
and doesn't depend on a column having no blanks.

## Iterators: the X functions

`SUMX`, `AVERAGEX`, `MINX`, `MAXX`, `COUNTX`, `RANKX` and friends take a **table** and an
**expression**, evaluate the expression once per row, then aggregate:

```dax
List Value = SUMX ( FactSales, FactSales[Units] * RELATED ( DimProduct[ListPrice] ) )
```

`RELATED` follows a many-to-one relationship from the current fact row to its dimension
row and returns a column value there. Row by row:

| SalesKey | Units | ListPrice | Units × ListPrice | Amount |
|---------:|------:|----------:|------------------:|-------:|
| 1 | 2 | 120 | 240 | 240 |
| 2 | 4 | 25 | 100 | 100 |
| 3 | 1 | 80 | 80 | 80 |
| 4 | 1 | 120 | 120 | 120 |
| 5 | 2 | 95 | 190 | 190 |
| 6 | 6 | 25 | 150 | 150 |
| 7 | 3 | 80 | 240 | 240 |
| 8 | 1 | 95 | 95 | 95 |

`List Value` = **1,215**, identical to Amount — in this sample, everything sold at list price.
A useful check measure:

```dax
Price Variance = [Total Amount] - [List Value]   -- 0 here; negative means discounting
```

Level 3 covers iterators in depth, including their cost.

## BLANK is not zero

DAX has a special value, **BLANK**. Aggregating no rows returns BLANK, not 0, and visuals
hide rows where every measure is blank. Camp Stove has no sales, so `[Total Amount]` for it
is BLANK and it disappears from a Product table.

BLANK's arithmetic rules:

- `BLANK() + 5 = 5`, `BLANK() * 5 = BLANK()`, `BLANK() - BLANK() = BLANK()`
- In comparisons, `BLANK() = 0` is **TRUE** (use `ISBLANK()` or `==` for strict checks)
- `DIVIDE ( x, BLANK() )` returns BLANK

If a business user needs to see Camp Stove with 0, be explicit and targeted:

```dax
Total Amount (zero-filled) =
VAR Amount = [Total Amount]
RETURN IF ( ISBLANK ( Amount ), 0, Amount )
```

Beware: this makes *every* combination non-blank, so a Product × Store matrix now shows all
15 combinations, most with 0 — sometimes what you want, often clutter, and slower on big
dimensions. `[Total Amount] + 0` is a common shorthand with the same effect.

## Variables

`VAR … RETURN` names intermediate results. Variables are evaluated once (where they are
defined, in the context at that point) and reused:

```dax
Avg Price per Unit =
VAR Amount = [Total Amount]
VAR Units  = SUM ( FactSales[Units] )
RETURN
    DIVIDE ( Amount, Units )
```

Whole model: 1,215 ÷ 20 = **60.75**. By category: Camping 645 ÷ 6 = **107.50**, Apparel
320 ÷ 4 = **80.00**, Accessories 250 ÷ 10 = **25.00**.

(Units by category: Camping rows 1, 4, 5, 8 → 2 + 1 + 2 + 1 = 6; Apparel rows 3, 7 → 4;
Accessories rows 2, 6 → 10.)

## Logic and text

```dax
Size Band = SWITCH ( TRUE (), [Total Amount] >= 500, "Big", [Total Amount] >= 250, "Mid", "Small" )
```

By Region: West 670 → Big, South 315 → Mid, East 230 → Small. (As a measure, this returns
a label for a visual cell; it still can't be used as an axis — see Level 1, lesson 07.)

## How It Actually Works

A DAX query is executed by two cooperating engines:

- The **storage engine** (VertiPaq for imported data) scans compressed columns and
  performs simple aggregations — `SUM`, `COUNT`, `MIN`, `MAX`, `DISTINCTCOUNT` — very fast
  and in parallel across segments of the table. It can also evaluate simple row-level
  arithmetic inside a scan, which is why `SUMX(FactSales, FactSales[Units] *
  FactSales[Price])` over columns of the same table is typically cheap.
- The **formula engine** handles everything else: logic, `IF`/`SWITCH`, combining
  results, and calls the storage engine cannot resolve. It is single-threaded.

`SUM(FactSales[Amount])` is actually shorthand for `SUMX(FactSales, FactSales[Amount])`;
the engine recognizes the pattern and pushes the whole thing into one storage engine scan.
When an iterator's expression includes something the storage engine can't handle — a
complex `IF`, a measure reference that needs its own context — the formula engine may
have to materialize intermediate rows and iterate them itself. That's the root of most DAX
performance problems, and Level 3 shows how to spot it.

Variables are evaluated lazily in current engines — only if used — but crucially **once**,
in the context where they're defined. That's why they both speed up and clarify measures,
and also why a variable defined outside a `CALCULATE` doesn't change when `CALCULATE`
changes the filters (next lesson).

## Common mistakes

- **Unqualified column references** and qualified measure references, making formulas
  hard to read.
- **`AVERAGE` of a column when you meant a ratio of sums** (weighted vs unweighted).
- **Zero-filling everything** and exploding visuals into thousands of empty rows.
- **Using `=` to test for blank** and accidentally matching zeros.

## Exercise

1. Create `Total Amount`, `Total Units`, `List Value`, `Price Variance` and `Avg Price per
   Unit`. Confirm the numbers above.
2. Change row 7's Amount to 210 (a discount). Before refreshing visuals, compute by hand
   the new `Price Variance` for Apparel and in total. (Answer: −30 in both.)
3. Build a Product table with `[Total Amount]`, then the zero-filled version. Explain why
   Camp Stove appears only with the second.
