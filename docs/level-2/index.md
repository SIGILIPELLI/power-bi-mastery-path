---
description: "Level 2 of Power BI: star schemas, relationships and filter direction, DAX and CALCULATE, time intelligence, M merges, drill-through, RLS, and refresh."
---

# Level 2 · Intermediate <span class="level-badge">Modeling & DAX</span>

Goal: stop building reports on one flat table and start building **models** — several
tables related in a star schema, with DAX measures that behave correctly at every level
of every visual, security that limits who sees what, and a refresh that runs without you.

This is the level where Power BI "clicks" for most people. The two ideas that do the most
work are **filter propagation** (a filter on a dimension table flows along relationships
into the fact table) and **evaluation context** (every DAX expression runs in a filter
context, and sometimes a row context, and `CALCULATE` is the tool that changes them).

## Modules

1. [Star Schema & Data Modeling](01-star-schema.md) — facts, dimensions, grain, and why one wide table is a trap
2. [Relationships, Cardinality & Filter Direction](02-relationships.md) — one-to-many, many-to-many, single vs both directions, inactive relationships
3. [DAX Fundamentals](03-dax-fundamentals.md) — syntax, aggregators, iterators at first sight, BLANK, and writing readable measures
4. [CALCULATE, Filter Context & Row Context](04-calculate-contexts.md) — the core of DAX, traced by hand
5. [Time Intelligence & the Date Table](05-time-intelligence.md) — building a date table, marking it, YTD, prior year, rolling periods
6. [Power Query M: Merges, Appends & Parameters](06-power-query-m.md) — joins, stacking tables, custom functions, and parameters
7. [Drill-through, Tooltips & Bookmarks](07-drillthrough-bookmarks.md) — navigation patterns that keep report pages simple
8. [Row-Level Security](08-row-level-security.md) — static and dynamic roles, `USERPRINCIPALNAME()`, and testing
9. [Refresh & Gateways](09-refresh-gateways.md) — how scheduled refresh works, credentials, and the on-premises gateway
10. [Project — A Multi-Table Sales Model](10-project-sales-model.md) — a verified star schema with time intelligence and dynamic RLS

## The sample model used in this level

Several lessons use the same small star schema. Create each table with **Home → Enter
data** (or as CSVs) so you can reproduce every hand-computed result.

**DimProduct**

| ProductKey | Product | Category | ListPrice |
|-----------:|---------|----------|----------:|
| 1 | Trail Tent | Camping | 120 |
| 2 | Rain Jacket | Apparel | 80 |
| 3 | Headlamp | Accessories | 25 |
| 4 | Sleeping Bag | Camping | 95 |
| 5 | Camp Stove | Camping | 60 |

**DimStore**

| StoreKey | Store | Region |
|---------:|-------|--------|
| 1 | Denver | West |
| 2 | Austin | South |
| 3 | Boston | East |

**FactSales**

| SalesKey | OrderDate | ProductKey | StoreKey | Units | Amount |
|---------:|-----------|-----------:|---------:|------:|-------:|
| 1 | 2025-01-03 | 1 | 1 | 2 | 240 |
| 2 | 2025-01-10 | 3 | 2 | 4 | 100 |
| 3 | 2025-01-21 | 2 | 3 | 1 | 80 |
| 4 | 2025-02-04 | 1 | 2 | 1 | 120 |
| 5 | 2025-02-12 | 4 | 1 | 2 | 190 |
| 6 | 2025-02-25 | 3 | 3 | 6 | 150 |
| 7 | 2025-03-08 | 2 | 1 | 3 | 240 |
| 8 | 2025-03-19 | 4 | 2 | 1 | 95 |

Reference totals you'll meet repeatedly:

- Total Amount **1,215**; total Units **20**; 8 sales rows.
- By Category: Camping **645**, Apparel **320**, Accessories **250**.
- By Region: West **670**, South **315**, East **230**.
- By month: January **420**, February **460**, March **335**.
- Camp Stove has **no sales** — useful for seeing how blanks behave.

## What you need before starting

- Level 1, especially [Measures vs Calculated Columns](../level-1/07-measures-vs-calculated-columns.md).
- Basic SQL joins help with lessons 01, 02 and 06, but aren't required.
- For lessons 08–09 you need access to the Power BI service; the gateway lesson is
  conceptual and walks through setup without requiring you to install one.
