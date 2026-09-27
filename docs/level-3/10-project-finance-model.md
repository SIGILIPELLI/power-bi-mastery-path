---
description: "Level 3 project: a Power BI profit-and-loss model with sign-aware DAX, a time calculation group, field parameters, and a measured performance fix."
---

# 10 · Project — An Optimized Finance Model

Finance models stress DAX in ways sales models don't: amounts with signs, subtotals that are
formulas rather than sums, and the same five time comparisons on every line. This project builds
a small profit-and-loss (P&L) model and applies the Level 3 toolkit to it: iterators and
variables, calculation groups, field parameters, and a before/after performance check.

## Data

**DimAccount**

| AccountKey | Account | Line | Sign | LineOrder |
|-----------:|---------|------|-----:|----------:|
| 1 | Product Sales | Revenue | 1 | 1 |
| 2 | Service Revenue | Revenue | 1 | 1 |
| 3 | Materials | COGS | −1 | 2 |
| 4 | Salaries | Opex | −1 | 3 |
| 5 | Rent | Opex | −1 | 3 |

**FactGL** — one row per account per month, `Amount` always positive as booked
(columns: `MonthStart`, `AccountKey`, `Amount`):

| Month | Acct 1 | Acct 2 | Acct 3 | Acct 4 | Acct 5 |
|-------|-------:|-------:|-------:|-------:|-------:|
| 2024-01 | 900 | 150 | 360 | 280 | 100 |
| 2024-02 | 950 | 200 | 400 | 280 | 100 |
| 2024-03 | 1,000 | 200 | 420 | 300 | 100 |
| 2025-01 | 1,000 | 200 | 400 | 300 | 100 |
| 2025-02 | 1,100 | 250 | 450 | 300 | 100 |
| 2025-03 | 900 | 300 | 380 | 320 | 100 |

That's 30 fact rows (6 months × 5 accounts). Load it in long form — unpivot the account columns
in Power Query as in Level 1, lesson 04 — then relate `DimAccount[AccountKey]` → `FactGL[AccountKey]`
and a marked `Date` table (2024–2025) → `FactGL[MonthStart]`. Sort `Line` by `LineOrder`.

## Step 1 — Base measures

```dax
Amount = SUM ( FactGL[Amount] )

Revenue = CALCULATE ( [Amount], DimAccount[Line] = "Revenue" )
COGS    = CALCULATE ( [Amount], DimAccount[Line] = "COGS" )
Opex    = CALCULATE ( [Amount], DimAccount[Line] = "Opex" )

Gross Margin     = [Revenue] - [COGS]
Operating Profit = [Gross Margin] - [Opex]
Gross Margin %   = DIVIDE ( [Gross Margin], [Revenue] )

Signed Amount =
SUMX (
    VALUES ( DimAccount[Sign] ),
    DimAccount[Sign] * [Amount]
)
```

`Signed Amount` iterates the (at most two) distinct signs and uses context transition on `[Amount]`
for each — a cheap iterator over a tiny table, giving revenue minus costs at any level. At the grand
total it equals Operating Profit.

## Step 2 — Expected results

| Period | Revenue | COGS | Gross Margin | Opex | Operating Profit | GM % |
|--------|--------:|-----:|-------------:|-----:|-----------------:|-----:|
| 2025-01 | 1,200 | 400 | 800 | 400 | 400 | 66.7% |
| 2025-02 | 1,350 | 450 | 900 | 400 | 500 | 66.7% |
| 2025-03 | 1,200 | 380 | 820 | 420 | 400 | 68.3% |
| **Q1 2025** | **3,750** | **1,230** | **2,520** | **1,220** | **1,300** | **67.2%** |
| Q1 2024 | 3,400 | 1,180 | 2,220 | 1,160 | 1,060 | 65.3% |

2024 by month: Operating Profit 310, 370, 380.

Hand checks: Q1 2025 Revenue = (1,000 + 200) + (1,100 + 250) + (900 + 300) = 3,750. Opex = (300 +
100) + (300 + 100) + (320 + 100) = 1,220. Operating Profit = 3,750 − 1,230 − 1,220 = 1,300.
GM % = 2,520 ÷ 3,750 = 0.672.

`Signed Amount` for Q1 2025 with no Line filter: 3,750 − 1,230 − 1,220 = **1,300** ✓.

## Step 3 — A P&L layout with a calculation group

Finance wants rows: Revenue, COGS, Gross Margin, Opex, Operating Profit, GM % — and columns:
Actual, PY, YoY %, YTD. Two tools:

**Field parameter `P&L Line`** with the six measures in that order, placed on a matrix's **rows**
(a field parameter with measures on rows gives a measure-per-row layout).

**Calculation group `Time Calc`** (lesson 06) with items `Actual`, `PY`, `YoY %`, `YTD`, placed on
**columns**. Give `YoY %` the format string expression:

```dax
IF (
    ISSELECTEDMEASURE ( [Gross Margin %] ),
    "+0.0 pp;-0.0 pp",      -- percentage-point change for a ratio
    "0.0%"
)
```

and change its expression so a ratio measure shows a *difference* rather than a growth rate:

```dax
VAR Cur  = SELECTEDMEASURE ()
VAR Prev = CALCULATE ( SELECTEDMEASURE (), SAMEPERIODLASTYEAR ( 'Date'[Date] ) )
RETURN
    IF (
        ISSELECTEDMEASURE ( [Gross Margin %] ),
        ( Cur - Prev ) * 100,
        DIVIDE ( Cur - Prev, Prev )
    )
```

Expected for Q1 2025 (filter the page to Q1 2025):

| Line | Actual | PY | YoY % |
|------|-------:|---:|------:|
| Revenue | 3,750 | 3,400 | 10.3% |
| COGS | 1,230 | 1,180 | 4.2% |
| Gross Margin | 2,520 | 2,220 | 13.5% |
| Opex | 1,220 | 1,160 | 5.2% |
| Operating Profit | 1,300 | 1,060 | 22.6% |
| GM % | 67.2% | 65.3% | +1.9 pp |

(350 ÷ 3,400 = 0.1029; 50 ÷ 1,180 = 0.0424; 300 ÷ 2,220 = 0.1351; 60 ÷ 1,160 = 0.0517;
240 ÷ 1,060 = 0.2264; 67.20 − 65.29 = 1.91 pp.)

## Step 4 — A performance before/after

Add a "variance commentary" flag that finance asked for: months where Operating Profit fell versus
the previous month. A first attempt:

```dax
Months With OP Decline (slow) =
COUNTROWS (
    FILTER (
        FactGL,
        CALCULATE ( [Operating Profit] )
            < CALCULATE ( [Operating Profit], PREVIOUSMONTH ( 'Date'[Date] ) )
    )
)
```

This iterates **fact rows** (30 here, millions in real ledgers), with two context transitions per
row, and it's also *wrong*: it counts fact rows, not months, and each row's context transition filters
to a single account. Iterate the grain you mean — months:

```dax
Months With OP Decline =
VAR MonthsInScope = VALUES ( 'Date'[Year Month] )
RETURN
    COUNTROWS (
        FILTER (
            MonthsInScope,
            VAR CurOP  = [Operating Profit]
            VAR PrevOP = CALCULATE ( [Operating Profit], PREVIOUSMONTH ( 'Date'[Date] ) )
            RETURN NOT ISBLANK ( PrevOP ) && CurOP < PrevOP
        )
    )
```

Hand check across 2024–2025 months with data: 2024-02 (370 vs 310) no; 2024-03 (380 vs 370) no;
2025-01 vs 2024-12 (no data, excluded); 2025-02 (500 vs 400) no; 2025-03 (400 vs 500) **yes**.
Result: **1**. (2024-01 has no prior month data either.)

Measure it: record Performance Analyzer for a card with each version, and if you have DAX Studio,
compare Server Timings (number of storage-engine queries, FE vs SE time) with the cache cleared. On
30 rows both will be instant; the point is to read the *shape*: the row-level version generates work
proportional to fact rows, the month-level version to months. Write down what you observed rather
than expected.

## Step 5 — Model hygiene

- Hide `FactGL[AccountKey]`, `FactGL[Amount]` (force use of measures), and `LineOrder`.
- Set **Discourage implicit measures** (calculation groups do this).
- Remove unused columns; check with VertiPaq Analyzer if available.
- Put measures in display folders: `P&L`, `Ratios`, `Diagnostics`.

## How It Actually Works

In the P&L matrix, each cell is the intersection of three things: a field-parameter row (which
measure the report layer puts into the query), a calculation-item column (which transformation the
engine applies to that measure reference), and the page's date filter. For the cell (Operating
Profit, PY), the engine takes the measure reference `[Operating Profit]`, replaces it with the PY
item's expression, and evaluates `SELECTEDMEASURE()` — Operating Profit — inside a `CALCULATE`
whose date filter has been shifted back a year. Operating Profit's own references (`[Gross
Margin]`, `[Opex]`, and through them `[Revenue]` and `[COGS]`) are then ordinary measure
evaluations in that shifted context, which is why the PY column shows 2024's numbers all the way
down. `ISSELECTEDMEASURE ( [Gross Margin %] )` inspects the measure the visual asked for — the row
of the field parameter — so the percentage-point branch fires only on the GM % row.

If a cell in your build disagrees with the hand-computed table, that disagreement is the most
useful information you have: it usually means an item is being applied in a context you didn't
intend (for example, a measure that already contains its own `SAMEPERIODLASTYEAR` being wrapped by
the PY item, shifting two years back and returning blank).

## Exercise

1. Build the model and reproduce the tables in Steps 2 and 3 and the result in Step 4.
2. Add a `Budget` fact at month × Line grain (your own numbers), a `vs Budget %` calculation item,
   and verify one cell by hand.
3. Write a short performance note: which measures iterate which tables, and how many rows each
   iteration touches at production scale (assume 5 million GL rows, 2,000 accounts, 36 months).
