---
description: "Choose Power BI storage modes: Import, DirectQuery, Dual and composite models, user-defined aggregations, and the limits and trade-offs of each design."
---

# 05 · Import, DirectQuery & Composite Models

Level 1 said "start with Import." This lesson is about when not to, and what happens when you
mix modes in one model. Every table in a Power BI model has a **storage mode**; the mix you
choose decides freshness, speed, which DAX runs where, and how much load you put on the source.

## The storage modes

| Mode | Data lives | Queries answered by | Typical use |
|------|-----------|---------------------|-------------|
| **Import** | In the model (VertiPaq) | Local storage engine | Most tables; fastest |
| **DirectQuery** | In the source | The source database, via generated SQL | Very large or near-real-time facts |
| **Dual** | Both | Import cache *or* source, whichever the query needs | Dimensions shared by Import and DirectQuery facts |
| **Direct Lake** | Delta tables in OneLake (Fabric) | VertiPaq, loading column data on demand from Delta files | Fabric lakehouse/warehouse models (Level 4) |

Set a table's mode in **Model view → select table → Properties → Advanced → Storage mode**.
Changing from Import to DirectQuery generally isn't allowed once imported; changing DirectQuery
to Import or Dual is, and is one-way in practice.

A model that combines tables in different modes — or combines DirectQuery connections to
several sources, including other Power BI semantic models — is a **composite model**.

## DirectQuery: what changes

With DirectQuery, every visual's DAX query is translated into one or more **native queries**
(SQL for relational sources) and sent to the source. Consequences:

- **Freshness**: data is as current as the source (subject to caching and visual refresh).
- **Performance** depends on the source's ability to answer aggregate queries quickly: indexes,
  columnstore, and warehouse sizing matter more than anything you do in Power BI.
- **Load**: a page with 12 visuals and 20 users can mean hundreds of source queries per minute.
- **Limits**: some Power Query transformations can't be used (everything must fold); some DAX
  functions are unsupported or slow in calculated columns; there's a cap on rows returned by an
  intermediate query (commonly cited as 1 million rows — check current documentation), which
  complex measures can hit.
- **Single sign-on** can pass the report reader's identity to the source for supported sources,
  so database-level security applies.

## Worked example: designing a composite model

Scenario: a `FactClicks` table with billions of rows in a cloud warehouse, updated
continuously; `DimProduct` (5,000 rows) and `Date` (3,653 rows); a monthly `FactBudget` from
Excel (1,000 rows).

| Table | Mode | Why |
|-------|------|-----|
| FactClicks | DirectQuery | Too large to import; must be near real time |
| DimProduct | **Dual** | Filters both FactClicks (DQ) and FactBudget (Import) |
| Date | **Dual** | Same reason |
| FactBudget | Import | Small; the source (Excel) can't be DirectQueried anyway |

Why Dual and not Import for the dimensions? If `DimProduct` were Import and `FactClicks` were
DirectQuery, a visual of clicks by `Category` would need to join an imported table to a
remote one. The engine can do it, but by sending lists of keys in the SQL (`WHERE ProductKey
IN (…)`) — a **limited relationship** across source groups, slower and with some semantic
restrictions. With Dual, the engine can send the whole query, including the join to
`DimProduct`, to the warehouse, while budget visuals use the imported copy.

## Aggregations

A **user-defined aggregation** table stores a pre-summarized copy of a DirectQuery fact:

```text
AggClicksByDayProduct (Import)
  DateKey, ProductKey, Clicks (sum), Sessions (sum)   -- e.g. a few million rows
```

In **Manage aggregations** on the agg table, map each column: `Clicks` → Sum of
`FactClicks[Clicks]`, `DateKey` → GroupBy `FactClicks[DateKey]`, and so on. Hide the agg table.

Now a visual of clicks by month and category is answered from the imported aggregation
(milliseconds); a drill to individual click rows falls through to DirectQuery. Users see one
`FactClicks` — the redirection is invisible. Automatic aggregations (machine-learned) exist
for capacity workspaces; check their current availability and requirements.

## Live connection vs DirectQuery to a semantic model

- **Live connection**: a report connected to a published semantic model; no local model; you
  can only add report-level measures.
- **DirectQuery to a Power BI semantic model** (composite): you "Make changes to this model,"
  which creates a local model that references the remote one and lets you add tables (for
  example, an Excel mapping) and relationships. Useful, but it creates a chain of dependencies
  and has security and performance implications — use sparingly in enterprise settings.

## How It Actually Works

The formula engine is the same in every mode; what changes is who the storage-engine requests
go to. For Import tables, the FE sends xmSQL to VertiPaq. For DirectQuery tables, it sends
requests to a DirectQuery **provider** that translates them into the source's query language.
Tables that live in the same source form a **source group**; relationships inside one source
group are *regular* and can be pushed into the native query as joins. Relationships that cross
source groups are **limited relationships**: the engine can't ask one source to join to data it
doesn't have, so it gets the grouping keys from one side and injects them as literal filters
(or joins intermediate results in the FE). That's why cross-source visuals can be slow with
high-cardinality keys.

**Dual** tables resolve this at query time: the engine checks which other tables a query
touches. If it touches only Import tables, Dual behaves as Import. If it touches a DirectQuery
table in the same source group, Dual behaves as DirectQuery so the whole query is pushed down.

The aggregation feature is a rewrite step in the same planner: before sending a request to the
DirectQuery source, the engine checks whether an aggregation table covers the requested columns
and aggregates at the requested grain; if so, it rewrites the request to scan the imported table
instead.

## Common mistakes

- Choosing DirectQuery "for freshness" on a source that can't answer aggregates quickly —
  every slicer click becomes a slow table scan.
- Import dimensions with DirectQuery facts (limited relationships everywhere) instead of Dual.
- Heavy DAX (iterators with context transition) on DirectQuery facts — each becomes complex
  SQL.
- Assuming the source can handle report concurrency; talk to the DBA before launch.

## Exercise

1. For the scenario above, write down the storage mode of each table and the reason, then
   add a fifth table — `DimCustomer` with 30 million rows — and decide its mode.
   (DirectQuery or Dual are both defensible; importing 30 M customers just to filter clicks
   usually isn't. Justify your choice by the queries it will serve.)
2. If you have access to a SQL database, create a DirectQuery table and use Performance
   Analyzer's copied query plus the database's query history to see the SQL generated for a
   bar chart.
3. Design an aggregation table for "clicks by day by product" and list every column mapping
   you'd define.
