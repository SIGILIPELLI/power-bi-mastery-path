---
description: "Test Power BI semantic models: reconciliation with source systems, DAX query tests with expected values, RLS tests, and best-practice rules."
---

# 09 · Testing & Validating Semantic Models

Every lesson in this course has checked results against hand-computed numbers. At enterprise scale
that habit becomes a **test suite**: a set of queries with expected results that runs before every
deployment, so a changed measure or broken relationship is caught in Test rather than by the CFO.
This lesson shows what to test and how, using tools that range from a DAX query in Desktop to
automated runs against the XMLA endpoint.

## What to test

| Test type | Question | Example |
|-----------|----------|---------|
| **Reconciliation** | Does the model agree with the system of record? | Total sales for March equals the ERP's March sales report |
| **Measure unit tests** | Does each measure return the known answer on known data? | `Achievement %` for Camping in Q1 = 75.9% |
| **Structural tests** | Is the model shaped as intended? | Every fact row matches a dimension row (no "(Blank)" members); keys unique |
| **Security tests** | Does each role see exactly what it should? | Maria sees 985, Dev sees 230 |
| **Best-practice rules** | Does the model follow team standards? | No bidirectional relationships; all measures formatted |
| **Performance budgets** | Is the key page fast enough? | Overview page visuals under a set time on a cold cache |

## DAX query view and DAX queries

Power BI Desktop includes a **DAX query view** (the icon on the left rail, in recent releases) where
you can write and run `EVALUATE` queries against the open model; DAX Studio does the same as an
external tool. A test query that returns one row per test, with a pass/fail column:

```dax
EVALUATE
VAR Tests =
    UNION (
        ROW ( "Test", "Total sales Q1 2025", "Expected", 1215,
              "Actual", [Sales Amount] ),
        ROW ( "Test", "Camping sales", "Expected", 645,
              "Actual", CALCULATE ( [Sales Amount], DimProduct[Category] = "Camping" ) ),
        ROW ( "Test", "West sales", "Expected", 670,
              "Actual", CALCULATE ( [Sales Amount], DimStore[Region] = "West" ) ),
        ROW ( "Test", "Target total", "Expected", 1460,
              "Actual", [Target Amount] ),
        ROW ( "Test", "Fact rows", "Expected", 8,
              "Actual", COUNTROWS ( FactSales ) )
    )
RETURN
    ADDCOLUMNS ( Tests, "Pass", [Expected] = [Actual] )
```

Expected output for the Level 2 project model (all rows `Pass = TRUE`):

| Test | Expected | Actual | Pass |
|------|---------:|-------:|------|
| Total sales Q1 2025 | 1215 | 1215 | True |
| Camping sales | 645 | 645 | True |
| West sales | 670 | 670 | True |
| Target total | 1460 | 1460 | True |
| Fact rows | 8 | 8 | True |

For ratio measures, compare with a tolerance rather than equality:
`ABS ( [Actual] - [Expected] ) < 0.0005`.

### Structural tests as queries

```dax
-- Orphaned fact rows (should return an empty table)
EVALUATE
FILTER ( FactSales, ISBLANK ( RELATED ( DimProduct[ProductKey] ) ) )

-- Duplicate keys in a dimension (should return an empty table)
EVALUATE
FILTER (
    ADDCOLUMNS ( VALUES ( DimProduct[ProductKey] ), "Rows", CALCULATE ( COUNTROWS ( DimProduct ) ) ),
    [Rows] > 1
)
```

The second query relies on context transition (Level 3, lesson 02): for each key, `CALCULATE`
filters `DimProduct` to that key and counts rows.

## Reconciliation

Pick a small number of **anchor figures** that the business already trusts — monthly revenue from the
finance close, order counts from the order system — and compare them for closed periods:

1. Write the source query (SQL against the warehouse, or the ERP report) for March 2025 sales.
2. Write the DAX equivalent:
   `EVALUATE ROW ( "Mar", CALCULATE ( [Sales Amount], 'Date'[Year Month] = "2025-03" ) )`
   → 335 in the sample.
3. Record both, and the difference. Agree an acceptable tolerance (often zero for closed periods).
4. When they differ, trace by dimension: which category, which store, which day?

Reconciliation catches problems unit tests can't, such as a Power Query filter that silently excludes
cancelled orders the finance report includes.

## Security tests

- In Desktop: **Modeling → View as** with roles and "Other user" (Level 2, lesson 08).
- In DAX Studio and some other tools, you can connect with a role or effective user name specified in
  the connection properties and run the same test query — so the RLS test is: "run the test suite as
  Maria; expected Sales = 985."
- In the service: semantic model **Security → Test as role**.

Automate the ones you can; document the manual ones in the release checklist.

## Best-practice rules

Tabular Editor's **Best Practice Analyzer** evaluates rules (expressions over the model's metadata)
and lists violations; community-maintained rule sets cover performance, DAX style, naming, formatting
and error prevention. Microsoft Fabric also offers a Best Practice Analyzer through Semantic Link in
notebooks. Pick a small set of rules your team agrees to enforce, and run them in CI (lesson 07).

## Automating the suite

On a capacity or PPU workspace, the **XMLA endpoint** lets tools connect to the published model as if
it were Analysis Services. A CI job can:

1. Deploy the model to a Test workspace (pipeline or Git sync).
2. Refresh it (or use a fixed test data snapshot).
3. Run the test queries through the XMLA endpoint or the **Execute Queries** REST API.
4. Fail the release if any `Pass` is false.

In Fabric, notebooks with the Semantic Link library (`sempy`) can evaluate DAX against a semantic model
from Python, which makes a notebook a convenient test runner:

```python
import sempy.fabric as fabric

df = fabric.evaluate_dax(
    dataset="Trailhead Sales",
    dax_string="""
EVALUATE ROW ( "Sales", [Sales Amount], "Target", [Target Amount] )
""",
)
row = df.iloc[0]
assert round(row.iloc[0], 2) == 1215, row
assert round(row.iloc[1], 2) == 1460, row
```

Column names in the returned DataFrame follow the DAX output (e.g. `[Sales]`), which is why the snippet
reads by position. Check the current `sempy` documentation for exact signatures.

## How It Actually Works

A DAX query is exactly what a visual sends (Level 1, lesson 05): `EVALUATE` plus a table expression.
Test queries therefore exercise the **same engine path** as the report — the same measures, relationships,
filter propagation and, when run under a role, the same security filters. That's what makes them
meaningful: a test that recomputes a number in SQL only tests the SQL.

`ROW` creates a one-row table with named columns; `UNION` stacks tables with the same number of columns;
`ADDCOLUMNS` evaluates `[Expected] = [Actual]` in a row context for each test row. The measure references
inside each `ROW` are evaluated in the query's filter context — the whole model, unless the test adds
filters with `CALCULATE` — so each test states its own filters explicitly, which keeps tests independent
of each other.

## Common mistakes

- Tests that reuse the model's measures to compute the "expected" value — they'll always pass.
- Only testing grand totals; bugs often live in a specific filter combination.
- Reconciling against open periods whose source data is still changing.
- Skipping security tests because "RLS hasn't changed" — relationships changing can change what RLS
  reaches.

## Exercise

1. Paste the test query into DAX query view (or DAX Studio) against your Level 2 project and make every
   test pass. Then deliberately break a relationship and watch which tests fail.
2. Add five more tests, including one ratio with a tolerance and the two structural queries.
3. Write down three anchor figures for reconciliation in a real or imagined business, with the system of
   record for each.
