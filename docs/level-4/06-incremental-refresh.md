---
description: "Configure Power BI incremental refresh: RangeStart and RangeEnd parameters, refresh policies, partitions, query folding requirements, and hybrid tables."
---

# 06 · Incremental Refresh

A fact table with five years of history doesn't change for four and a half of them. Yet a normal
refresh reloads every row, every night. **Incremental refresh** splits the table into date-based
partitions, reloads only the recent ones, and keeps the rest — turning a two-hour refresh into
minutes and reducing load on the source.

## Requirements

- The table has a **date or date-time column** to partition by (usually the transaction date).
- The source supports **query folding** for the date filter (relational databases, most
  warehouses/lakehouse SQL endpoints). Against a non-folding source, the policy still "works" but
  every partition refresh reads the whole source and filters in the mashup engine — defeating the
  purpose, and Desktop warns you.
- Two Power Query parameters named exactly **`RangeStart`** and **`RangeEnd`**, of type
  **Date/Time**.
- Publishing to the service (Desktop doesn't create partitions; it only loads the parameter range).

Incremental refresh is available with Pro as well as capacity, with some advanced options
(real-time/hybrid tables, XMLA access to partitions) needing capacity or PPU. Check current
documentation for limits.

## Step by step

1. **Home → Transform data → Manage Parameters → New**:
    - `RangeStart`, type Date/Time, current value `2025-01-01 00:00:00`
    - `RangeEnd`, type Date/Time, current value `2025-02-01 00:00:00`
2. In the `FactSales` query, filter the date column **with the parameters**:

```powerquery
#"Filtered Rows" = Table.SelectRows(Source, each
    [OrderDateTime] >= RangeStart and [OrderDateTime] < RangeEnd)
```

   Use `>=` on one end and `<` on the other. If both were inclusive, a row stamped exactly at a
   partition boundary would load into two partitions and be counted twice.

   If the column is a **Date** (not Date/Time), convert the parameters in the filter:
   `[OrderDate] >= Date.From(RangeStart) and [OrderDate] < Date.From(RangeEnd)`. If the source stores
   dates as integer keys (20250101), convert parameters to that form in a helper so the filter still
   folds.

3. Right-click the step → **View Native Query** to confirm the filter appears in the SQL `WHERE`.
4. **Close & Apply.** Desktop loads only January 2025 — a small, fast development copy.
5. In the Data pane, right-click `FactSales` → **Incremental refresh**. Configure the policy:
    - **Archive data starting** `3` **Years** before refresh date.
    - **Incrementally refresh data starting** `3` **Months** before refresh date.
    - Optional: **Get the latest data in real time with DirectQuery** (hybrid table; capacity/PPU).
    - Optional: **Only refresh complete** days/months.
    - Optional: **Detect data changes** using a last-modified column.
6. Publish, set credentials, and run the first refresh (it loads all history, so it's the slow one).

## Worked example: what partitions exist

With the policy above and a refresh on **2025-09-27**, the service creates partitions roughly like
this (it chooses the granularity: whole years for older data, then quarters, then months):

| Partition | Covers | Refreshed on 2025-09-27? |
|-----------|--------|--------------------------|
| 2023 | Year 2023 | No (archive) |
| 2024 | Year 2024 | No |
| 2025 Q1 | Jan–Mar 2025 | No |
| 2025 Q2 | Apr–Jun 2025 | No |
| 2025-07 | July | **Yes** |
| 2025-08 | August | **Yes** |
| 2025-09 | September (to date) | **Yes** |

Each partition's query is the same Power Query definition with `RangeStart`/`RangeEnd` substituted by
that partition's bounds. The refresh window runs three queries — July, August, September — instead of
reading ~33 months.

On **2026-01-02**, the window moves: October–December 2025 and January 2026 become the refreshed
months as the window rolls; older monthly partitions are merged into quarters, quarters into years,
and the oldest year falls out of the three-year archive and is dropped. You don't manage any of this;
the policy does.

Rough arithmetic of the benefit: if the table grows evenly by 1 million rows a month, a full refresh
in September 2025 reads about 33 million rows (2023 through September 2025); the incremental refresh
reads about 3 million.

## Detect data changes

If old rows can change (late adjustments), enable **Detect data changes** with a column like
`LastModified`. For each partition in the *incremental* window, the service first asks the source for
`MAX(LastModified)`; if it hasn't changed since the last refresh, that partition is skipped. Changes to
rows in archived partitions are **not** picked up — you'd refresh those partitions explicitly (for
example via XMLA tools or the REST API) when you know history changed.

## How It Actually Works

When you publish a table with a refresh policy, the service converts it into a **partitioned table**
in the hosted Analysis Services model. Each partition has its own M query — your query with the two
parameters bound to that partition's range — and its own stored segments in VertiPaq. At refresh time
the service applies the policy: it calculates the partitions that should exist for "today," creates
missing ones, merges old ones into coarser ones, deletes ones outside the archive range, and processes
only those in the incremental window (or those whose change-detection value moved).

Because each partition's query includes `WHERE OrderDateTime >= @start AND OrderDateTime < @end`
(folded from your filter step), the source only returns that slice — ideally using an index or
partition elimination on its side. Processing a partition replaces its data and then recalculates
dependent structures (relationships, calculated columns, hierarchies), which is why calculated columns
on a huge incrementally refreshed table still cost time at every refresh: they're recalculated for the
whole table, not just the refreshed partitions, in many cases. Prefer computing such columns upstream.

Hybrid tables add one more partition in **DirectQuery** mode covering the period after the last
refresh, so queries for "today" go to the source while history comes from import partitions.

## Common mistakes

- Parameters of type Date instead of Date/Time, or misnamed (`rangeStart`).
- Inclusive bounds on both ends → duplicate rows at boundaries.
- A step before the filter that breaks folding (e.g. a custom column using a non-foldable function).
- Expecting archive partitions to pick up corrections automatically.
- Downloading the `.pbix` from the service after incremental refresh is configured — it isn't
  generally possible once the model has been refreshed with partitions; keep source files in Git
  (next lesson).

## Exercise

1. Add `RangeStart`/`RangeEnd` to a model built on a SQL source (a local SQL Server Express or
   PostgreSQL works), confirm the native query contains the date filter, and configure a policy.
2. For a policy of "archive 2 years, refresh 10 days" and a refresh on 2026-03-05, list the partitions
   you'd expect at the coarsest sensible granularity and which are refreshed.
3. Explain in two sentences why `>= RangeStart and < RangeEnd` prevents double counting, using a row
   stamped exactly 2025-08-01 00:00:00 as the example.
