---
description: "Build a proper date table in Power BI, mark it, and write year-to-date, prior-year, year-over-year and rolling-period DAX measures verified by hand."
---

# 05 · Time Intelligence & the Date Table

"Year to date," "same period last year," "rolling three months" — these are the questions
every business report asks, and DAX has functions for all of them. They only work reliably
with a **date table**: a dimension with one row per calendar day, no gaps, related to your
facts. This lesson builds one and then writes the standard measures.

## Why a date table

The automatic date hierarchy (Auto date/time) creates a hidden date table *per date
column*. That's convenient for a first report and a problem afterwards: you can't share one
calendar across fact tables, you can't add fiscal periods or holidays, and it adds hidden
tables that bloat the model. Turn it off for real models: **File → Options and settings →
Options → Current File → Data Load → Time intelligence → Auto date/time** (uncheck).

The time-intelligence functions need:

1. A column of type **Date** with **one row per day**, **no gaps**, covering **whole years**
   of the data's range.
2. That table related (1:*) to the fact's date column.
3. Ideally, the table **marked as a date table**.

## Sample data for this lesson

Add three prior-year rows to the Level 2 `FactSales` so there is something to compare
against (any product/store keys will do):

| SalesKey | OrderDate | ProductKey | StoreKey | Units | Amount |
|---------:|-----------|-----------:|---------:|------:|-------:|
| 9 | 2024-01-15 | 1 | 1 | 3 | 380 |
| 10 | 2024-02-15 | 2 | 2 | 5 | 400 |
| 11 | 2024-03-15 | 3 | 3 | 12 | 290 |

Monthly totals are now: Jan 2024 380, Feb 2024 400, Mar 2024 290, Jan 2025 420, Feb 2025
460, Mar 2025 335.

## Step by step: a DAX date table

**Modeling → New table**:

```dax
Date =
VAR FirstYear = YEAR ( MIN ( FactSales[OrderDate] ) )
VAR LastYear  = YEAR ( MAX ( FactSales[OrderDate] ) )
RETURN
    ADDCOLUMNS (
        CALENDAR ( DATE ( FirstYear, 1, 1 ), DATE ( LastYear, 12, 31 ) ),
        "Year", YEAR ( [Date] ),
        "Quarter", "Q" & QUARTER ( [Date] ),
        "Month Number", MONTH ( [Date] ),
        "Month", FORMAT ( [Date], "mmm" ),
        "Year Month", FORMAT ( [Date], "yyyy-mm" ),
        "Year Month Number", YEAR ( [Date] ) * 100 + MONTH ( [Date] )
    )
```

This yields 2024-01-01 to 2025-12-31: 366 + 365 = **731 rows** (2024 is a leap year).

Then:

1. Select the `Month` column → **Column tools → Sort by column → Month Number** (otherwise
   months sort alphabetically: Apr, Aug, Dec…).
2. **Table tools → Mark as date table**, choose the `Date` column. Power BI validates that
   it's unique and contiguous.
3. In Model view, relate `Date[Date]` (1) → `FactSales[OrderDate]` (*), single direction.
4. From now on, use `Date` columns — not `FactSales[OrderDate]` — on axes and slicers.

A Power Query date table is equally valid and is preferred by many teams because it can be
shared via a dataflow; the requirements are the same.

## The core measures

```dax
Total Amount = SUM ( FactSales[Amount] )

Amount YTD =
CALCULATE ( [Total Amount], DATESYTD ( 'Date'[Date] ) )

Amount PY =
CALCULATE ( [Total Amount], SAMEPERIODLASTYEAR ( 'Date'[Date] ) )

Amount YoY % =
DIVIDE ( [Total Amount] - [Amount PY], [Amount PY] )

Amount PY YTD =
CALCULATE ( [Amount YTD], SAMEPERIODLASTYEAR ( 'Date'[Date] ) )

Amount Rolling 3M =
CALCULATE (
    [Total Amount],
    DATESINPERIOD ( 'Date'[Date], MAX ( 'Date'[Date] ), -3, MONTH )
)
```

## Expected results

Matrix with `Date[Year Month]` on rows, filtered to 2025-01 … 2025-03:

| Year Month | Amount | YTD | PY | YoY % | PY YTD | Rolling 3M |
|------------|-------:|----:|---:|------:|-------:|-----------:|
| 2025-01 | 420 | 420 | 380 | 10.5% | 380 | 420 |
| 2025-02 | 460 | 880 | 400 | 15.0% | 780 | 880 |
| 2025-03 | 335 | 1,215 | 290 | 15.5% | 1,070 | 1,215 |

Hand checks:

- YoY for Jan: (420 − 380) ÷ 380 = 0.10526 → 10.5%. Mar: 45 ÷ 290 = 0.1552 → 15.5%.
- Rolling 3M at 2025-02: the anchor is 2025-02-28 and `DATESINPERIOD` counts back three
  months from it, giving 2024-11-29 … 2025-02-28. (Counting back from the 28th lands on
  the 29th, so the window picks up two November days — a known quirk at short month-ends.
  For strict calendar months, filter on a month-number column instead.) Nothing sold
  between late November and December 2024, so the result is 420 + 460 = 880. At 2025-03: Jan + Feb + Mar 2025 = 1,215.
- Rolling 3M for 2024-03 (if you include 2024 rows): 380 + 400 + 290 = 1,070.

A subtle one: at the **year total** row for 2025, `Amount YTD` = 1,215 (DATESYTD of the last
date in context, 2025-12-31, covers the whole year), and `Amount PY` = 1,070 — all of 2024.

## Fiscal years

For a fiscal year ending 30 June, pass the year-end date as text:

```dax
Amount FYTD = CALCULATE ( [Total Amount], DATESYTD ( 'Date'[Date], "06-30" ) )
```

Add matching `Fiscal Year` and `Fiscal Month Number` columns to the date table for axes.
Non-calendar patterns (4-4-5 retail calendars, ISO weeks) aren't handled by the built-in
functions; you write them with ordinary `CALCULATE` + `FILTER` over date-table columns.
Newer Power BI releases have introduced calendar-based time intelligence features that
help with custom calendars; check the current documentation for their status.

## Hiding future periods

Because the date table runs to 2025-12-31, `Amount YTD` shows 1,215 for April–December 2025
too (nothing new is added, but the running total persists). Suppress it:

```dax
Amount YTD (to last sale) =
VAR LastSaleDate = CALCULATE ( MAX ( FactSales[OrderDate] ), REMOVEFILTERS ( 'Date' ) )
RETURN
    IF ( MIN ( 'Date'[Date] ) <= LastSaleDate, [Amount YTD] )
```

`LastSaleDate` = 2025-03-19; for April onward, `MIN('Date'[Date])` is later, the `IF` has no
else branch, and the result is BLANK — so those rows disappear.

## How It Actually Works

Time-intelligence functions are **table functions that return a set of dates**, which
`CALCULATE` then uses as a filter on `'Date'[Date]`:

- `DATESYTD('Date'[Date])` looks at the dates in the current filter context, finds the
  **last** one, and returns every date from 1 January of that year up to it.
- `SAMEPERIODLASTYEAR('Date'[Date])` takes the current set of dates and shifts each back one
  year (with sensible handling for month ends and 29 February), returning that set.
- `DATESINPERIOD(col, anchor, -3, MONTH)` returns dates from three months before the anchor
  (exclusive) to the anchor.

Because they compute from the dates *present in the date column*, the column must be
complete: if 2024-12 were missing from the table, shifting February back a year or building
a rolling window would silently drop days. That's the real reason for "contiguous, whole
years."

**Marking as a date table** matters for another reason. When you filter `'Date'[Date]` inside
`CALCULATE`, other columns of the date table — like `Year Month` on your matrix rows — still
carry filters that would intersect with the new date set (row "2025-02" ∩ "dates of
2025-01-01…2025-02-28" = only February). The engine avoids that because, when a filter is
placed on the date column of a table marked as a date table (or on a column used in a
relationship of Date type), it automatically removes filters from the other columns of
that table — an implicit `REMOVEFILTERS('Date')`. Without it, some YTD patterns return
the wrong numbers when you slice by Year Month rather than by the date itself.

## Common mistakes

- Using `FactSales[OrderDate]` on axes after creating a date table — time intelligence then
  sees the wrong column's filters.
- A date table that starts at the first sale rather than 1 January, or with gaps.
- Forgetting **Sort by column** on month names.
- Date/time values in the fact table (`2025-01-03 14:22`) relating to a pure date table —
  they don't match `2025-01-03 00:00`. Convert to Date in Power Query.

## Exercise

1. Build the date table, mark it, relate it, and reproduce the expected-results table.
2. Add `Amount MTD` with `DATESMTD` and `Amount QTD` with `DATESQTD`. For a row showing the
   date 2025-02-12 in a day-level table, compute both by hand (MTD 310 = 120 + 190; QTD 730 =
   420 + 310).
3. Build `Amount YTD (to last sale)` and confirm April–December 2025 are blank.
