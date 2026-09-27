---
description: "Level 3 of Power BI: iterators, context transition, filter modifiers, VertiPaq performance, storage modes, calculation groups, and pipelines."
---

# Level 3 · Advanced <span class="level-badge">Engine & Scale</span>

Goal: understand what the engine is doing with your DAX and your data well enough to
predict results before you run them, make slow reports fast, and use the advanced modeling
features — calculation groups, field parameters, composite models — that separate a
workable model from a maintainable one.

Level 2 taught `CALCULATE` as "change the filters." Level 3 is precise about *how*: when a row
becomes a filter (context transition), how filter arguments replace or intersect existing
filters (`KEEPFILTERS`), what exactly `ALL` removes, and what the storage engine and formula
engine each do with a query.

## Modules

1. [Iterators & Variables](01-iterators-variables.md) — SUMX, AVERAGEX, RANKX, TOPN, FILTER; variables for clarity and correctness
2. [Context Transition](02-context-transition.md) — why a measure inside an iterator behaves differently from the same expression inline
3. [ALL, REMOVEFILTERS & KEEPFILTERS](03-filter-modifiers.md) — removing, restoring and intersecting filters, ALLSELECTED and visual totals
4. [Performance: VertiPaq & Performance Analyzer](04-performance-vertipaq.md) — columnar compression, cardinality, storage vs formula engine, DAX Studio
5. [Import, DirectQuery & Composite Models](05-storage-modes.md) — storage modes, dual tables, aggregations, and their trade-offs
6. [Calculation Groups](06-calculation-groups.md) — one set of time-intelligence logic for every measure
7. [Field Parameters](07-field-parameters.md) — letting readers choose the measure or dimension a visual shows
8. [Advanced Visuals, Custom Visuals & Paginated Reports](08-advanced-visuals-paginated.md) — decomposition trees, small multiples, AppSource visuals, and when to go paginated
9. [Deployment Pipelines](09-deployment-pipelines.md) — dev/test/prod stages, deployment rules, and what gets copied
10. [Project — An Optimized Finance Model](10-project-finance-model.md) — a P&L model with a calculation group, field parameters, and a measured performance fix

## The sample data used in this level

Lessons 01–03 use a tiny table so every iteration can be traced by hand. Create it with
**Enter data** as `Sales`, related to nothing:

| Product | Color | Qty | Price |
|---------|-------|----:|------:|
| A | Red | 2 | 10 |
| A | Blue | 1 | 10 |
| B | Red | 3 | 20 |
| C | Blue | 5 | 4 |
| C | Red | 1 | 4 |

Line values (Qty × Price): 20, 10, 60, 20, 4 → total **114**. By product: A **30**, B **60**,
C **24**. By color: Red 20 + 60 + 4 = **84**, Blue 10 + 20 = **30**. Total Qty **12**.

Some lessons also add a small `Products` dimension:

| Product | Brand |
|---------|-------|
| A | North |
| B | North |
| C | Summit |

related one-to-many to `Sales[Product]`.

## What you need before starting

- Level 2 complete — especially [CALCULATE, Filter Context & Row Context](../level-2/04-calculate-contexts.md).
- For lessons 04–06: **DAX Studio** and **Tabular Editor** are free community/third-party tools
  that connect to Power BI Desktop as *external tools*. They're Windows applications. The
  lessons explain what they show so you can follow the reasoning even without them.
- For lessons 05 and 09: some features need a Premium, PPU or Fabric capacity workspace; the
  lessons say so where it applies.
