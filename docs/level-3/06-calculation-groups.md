---
description: "Use Power BI calculation groups to apply time intelligence to any measure with SELECTEDMEASURE, including precedence, format strings, and pitfalls."
---

# 06 · Calculation Groups

Level 2 wrote `Amount YTD`, `Amount PY`, `Amount YoY %`. Now imagine ten base measures —
Sales, Cost, Margin, Units, Orders… — each needing YTD, PY, YoY, and rolling 3M. That's forty
nearly identical measures, and forty places to fix a bug. A **calculation group** defines the
time logic **once** and applies it to whichever measure is in the visual.

## The idea

A calculation group is a special table with one column. Each row is a **calculation item**
containing a DAX expression that refers to `SELECTEDMEASURE()` — "whatever measure is being
evaluated." Put the calculation group's column on a visual (or in a slicer) and each item
transforms the measure.

## Creating one

Power BI Desktop supports authoring calculation groups directly in **Model view** (newer
releases: **Home → Calculation group**, or right-click in the Data pane in Model view). Tabular
Editor can also create them. When you create the first one, Desktop turns on **Discourage
implicit measures**, because calculation groups only apply to explicit measures.

### Step by step: a time-intelligence group

Using the Level 2 model with the marked `Date` table (sales Jan–Mar 2024 380/400/290 and
Jan–Mar 2025 420/460/335):

1. Create a calculation group named `Time Calc`; rename its column to `Period`.
2. Add calculation items:

```dax
-- Current
SELECTEDMEASURE ()

-- YTD
CALCULATE ( SELECTEDMEASURE (), DATESYTD ( 'Date'[Date] ) )

-- PY
CALCULATE ( SELECTEDMEASURE (), SAMEPERIODLASTYEAR ( 'Date'[Date] ) )

-- YoY %
VAR Cur = SELECTEDMEASURE ()
VAR Prev = CALCULATE ( SELECTEDMEASURE (), SAMEPERIODLASTYEAR ( 'Date'[Date] ) )
RETURN DIVIDE ( Cur - Prev, Prev )
```

3. Set **Ordinal** on items (0, 1, 2, 3) so they appear in that order.
4. For **YoY %**, set a **dynamic format string expression**: `"0.0%"`. Other items keep the
   measure's own format (they return `SELECTEDMEASUREFORMATSTRING()` by default).

### Worked example

Matrix: `Date[Year Month]` (2025 months) on rows, `Time Calc[Period]` on columns, values
`[Total Amount]`:

| Year Month | Current | YTD | PY | YoY % |
|------------|--------:|----:|---:|------:|
| 2025-01 | 420 | 420 | 380 | 10.5% |
| 2025-02 | 460 | 880 | 400 | 15.0% |
| 2025-03 | 335 | 1,215 | 290 | 15.5% |

Swap the value to `[Total Units]` and the same four columns now compute units — no new
measures. With the Level 2 units (2025: Jan 7, Feb 9, Mar 4; 2024 rows added in lesson 05:
3, 5, 12), expected:

| Year Month | Current | YTD | PY | YoY % |
|------------|--------:|----:|---:|------:|
| 2025-01 | 7 | 7 | 3 | 133.3% |
| 2025-02 | 9 | 16 | 5 | 80.0% |
| 2025-03 | 4 | 20 | 12 | −66.7% |

(Jan 2025 units: rows 1–3 → 2 + 4 + 1 = 7; Feb: 1 + 2 + 6 = 9; Mar: 3 + 1 = 4.)

## Using a calculation item inside a measure

You can apply an item from DAX by filtering the calculation group column:

```dax
Sales PY = CALCULATE ( [Total Amount], 'Time Calc'[Period] = "PY" )
```

This lets you keep a few named measures for convenience while the logic lives in one place.

## Precedence

If a model has two calculation groups — say `Time Calc` and `Currency Conversion` — and both
are applied, which wraps which? Each group has a **Precedence** number; the **higher**
precedence group is applied first, i.e. it's the outer transformation, wrapping the lower one.
For "YoY % of converted amounts" you want conversion applied to the base measure *inside* the
YoY logic. Test with a two-row hand example before trusting any combination.

## How It Actually Works

A calculation group is stored as a table whose column is a filter target like any other. When
a filter context contains **exactly one** calculation item, the engine performs **calculation
item application**: when a *measure reference* is evaluated in that context, it replaces
the measure with the item's expression, substituting `SELECTEDMEASURE()` with the original
measure. `CALCULATE ( [Total Amount] )` under the "YTD" item becomes
`CALCULATE ( CALCULATE ( [Total Amount], DATESYTD ( 'Date'[Date] ) ) )`.

Three consequences of it being a *replacement of measure references*:

1. **Only explicit measures are affected.** Implicit "Sum of Amount" fields have no measure
   reference to replace — hence "Discourage implicit measures."
2. **The item wraps the measure the visual asked for.** `SELECTEDMEASURE()` is that measure,
   and it's evaluated inside the item's modified filter context — so the measures *it* calls
   (`[Margin]` and `[Sales]` inside `Margin %`) naturally see the YTD or PY dates, just as they
   would inside any `CALCULATE`. Where a model gets into trouble is when a measure *itself*
   references a calculation item (for example, a `Sales PY` measure built with the calculation
   group, then used under another item of the same group) — the rules for this "sideways"
   interaction are subtle. Keep calculation-group logic in the group, use `ISSELECTEDMEASURE` or
   `SELECTEDMEASURENAME` to special-case measures that need different treatment, and verify
   combinations against hand-computed numbers, as in the tables above.
3. **Multiple items selected** (no single item in the filter) means no application; the
   measure is evaluated as is. Totals across the Period column therefore show the raw measure,
   which is rarely meaningful — turn off column totals.

Dynamic format strings are evaluated in the same context and returned alongside the value, so
the visual can show "15.0%" for one column and "1,215" for another from the same measure.

## Common mistakes

- Expecting calculation groups to work on implicit measures.
- Forgetting ordinals, so items appear alphabetically (Current, PY, YoY %, YTD).
- Using the Period column in a slicer with multi-select allowed — multiple items means no
  transformation. Force single select.
- Ignoring precedence when adding a second calculation group.

## Exercise

1. Build `Time Calc` and reproduce both tables.
2. Add a `Rolling 3M` item using `DATESINPERIOD` and verify Mar 2025 = 1,215 for amount and 20
   for units.
3. Add a measure `Avg Price = DIVIDE ( [Total Amount], [Total Units] )` and show it with `YTD`.
   Compute by hand for Feb 2025: 880 ÷ 16 = **55.00**. Then explain why the YTD item gives a *ratio of
   year-to-date totals* (880 ÷ 16) rather than an average of monthly ratios (420 ÷ 7 = 60.00 and
   460 ÷ 9 = 51.11): the item changes the filter context once, and both inner measures are then
   evaluated over January–February together.
