---
description: "Go deeper in Power Query M: merge (join) kinds, append queries, custom columns with conditional logic, parameters, and turning a query into a function."
---

# 06 · Power Query M: Merges, Appends & Parameters

Level 1 used Power Query through the ribbon. Level 2 needs you to read and write M
comfortably, because real models combine several sources, parameterize connection
details, and reuse logic. This lesson covers the four tools you'll use constantly: merges,
appends, custom columns, and parameters (plus custom functions).

## Merges are joins

**Home → Merge Queries** joins two queries on one or more key columns and adds a column of
nested tables that you then **expand**. The **Join Kind** dropdown maps directly to SQL:

| Join kind | Keeps |
|-----------|-------|
| Left Outer | All rows from the first table, matches from the second |
| Right Outer | All rows from the second, matches from the first |
| Full Outer | All rows from both |
| Inner | Only matching rows |
| Left Anti | Rows in the first **with no match** in the second |
| Right Anti | Rows in the second with no match in the first |

### Worked example: finding orphaned keys

Take the Level 2 `FactSales` and a `DimProduct` that is missing Headlamp (ProductKey 3) —
a realistic "the dimension extract is late" situation.

1. Select `FactSales` → **Merge Queries as New**.
2. Second table `DimProduct`; select `ProductKey` in both; Join Kind **Left Anti**.
3. Result: sales rows whose product isn't in the dimension — rows 2 (Amount 100) and 6
   (Amount 150). Two rows, **250** total.

The M:

```powerquery
Orphans = Table.NestedJoin(
    FactSales, {"ProductKey"},
    DimProduct, {"ProductKey"},
    "DimProduct", JoinKind.LeftAnti
)
```

Turn this into a data-quality query that loads into the model (or a separate report page)
and you'll know when orphans appear instead of discovering a "(Blank)" category later.

### Worked example: enriching with a lookup

Merge `FactSales` (Left Outer) with `DimStore` on `StoreKey`, expand only `Region`:

```powerquery
Merged = Table.NestedJoin(FactSales, {"StoreKey"}, DimStore, {"StoreKey"},
    "Store", JoinKind.LeftOuter),
Expanded = Table.ExpandTableColumn(Merged, "Store", {"Region"}, {"Region"})
```

In a star schema you normally *don't* need this (the relationship handles it), but it's the
right tool when flattening a snowflake or building a dimension from several sources.

**Fuzzy merge** (tick *Use fuzzy matching*) matches on similarity instead of equality — useful
for messy customer names, dangerous for keys. Always review a sample of what it matched.

## Appends stack tables

**Home → Append Queries** puts rows of one table under another. Columns are matched **by
name**; a column present in only one table is filled with `null` in the other's rows.

Suppose the 2024 rows arrive in a separate file with slightly different headers:

| SalesKey | OrderDate | ProductKey | StoreKey | Qty | Amount |
|---------:|-----------|-----------:|---------:|----:|-------:|
| 9 | 2024-01-15 | 1 | 1 | 3 | 380 |

Append it to `FactSales` directly and you get both `Units` and `Qty` columns, each half
null. Rename `Qty` → `Units` in the 2024 query first, then append:

```powerquery
Combined = Table.Combine({FactSales2025, FactSales2024})
```

`Table.Combine` accepts a list, so appending three or twelve tables is the same call.

## Custom columns with logic

**Add Column → Custom Column** accepts any M expression. Conditional logic uses
`if … then … else`, and `null` handling is explicit:

```powerquery
#"Added Size Band" = Table.AddColumn(Source, "SizeBand", each
    if [Amount] = null then "Unknown"
    else if [Amount] >= 200 then "Large"
    else if [Amount] >= 100 then "Medium"
    else "Small",
    type text)
```

For the Level 2 fact: Large = rows 1 and 7 (240, 240); Medium = rows 2, 4, 5, 6 (100, 120,
190, 150); Small = rows 3 and 8 (80, 95). Row 8 is the one to watch: 95 is close to the
100 boundary, and it's easy to lump it into Medium when eyeballing. Boundary values
(exactly 100, exactly 200) land in the higher band because the tests use `>=`. Tracing a
small sample like this is how you catch off-by-one mistakes in banding logic.

Useful M patterns:

```powerquery
Text.Start([Product], 3)                 // first 3 characters
Date.StartOfMonth([OrderDate])           // month bucket
Number.Round([Amount] * 1.08, 2)         // rounding
try Number.From([Raw]) otherwise null    // safe conversion
List.Contains({"West","East"}, [Region]) // membership test
```

## Parameters

**Home → Manage Parameters → New Parameter** creates a named value you can use in any
query. The classic use: switching a source between development and production.

1. Create `ServerName` (Text, current value `dev-sql.contoso.local` — an example name) and
   `DatabaseName` (Text, `Sales`).
2. In the source step, replace literals with the parameters:

```powerquery
Source = Sql.Database(ServerName, DatabaseName)
```

3. After publishing, change parameter values in the service under the semantic model's
   **Settings → Parameters** without editing the file.

Parameters are also what incremental refresh uses (`RangeStart`/`RangeEnd`, Level 4), and
what deployment pipelines can swap per stage (Level 3).

## From query to function

When the same steps apply to many inputs, make a function. In a blank query
(**New Source → Blank Query**, then Advanced Editor):

```powerquery
(filePath as text) as table =>
let
    Source   = Csv.Document(File.Contents(filePath), [Delimiter = ",", Encoding = 65001]),
    Headers  = Table.PromoteHeaders(Source, [PromoteAllScalars = true]),
    Typed    = Table.TransformColumnTypes(Headers, {
                   {"OrderDate", type date}, {"Units", Int64.Type}, {"Amount", Currency.Type}})
in
    Typed
```

Name it `fnLoadSales`. Invoke it with **Add Column → Invoke Custom Function** on a table of
file paths, or call it directly: `fnLoadSales("C:\Data\sales_2025.csv")`. The Folder
connector from Level 1 generated exactly this kind of function for you.

## How It Actually Works

A merge in Power Query produces a **nested table per row** — the `DimProduct` column holds, for
each fact row, a table of matching dimension rows (zero, one or several). Expanding it
flattens those tables into columns, duplicating the fact row if there were several matches.
That's how a non-unique key on the "lookup" side multiplies rows in Power Query, just as it
would in SQL.

Whether this runs in the source or in the mashup engine depends on **query folding**. If
both queries come from the same SQL database with the same credentials, `Table.NestedJoin`
folds into a SQL `JOIN` and the database does the work. If one side is a CSV and the other
is SQL, nothing can fold across them; Power Query pulls both into its own engine and joins
there, often buffering one side entirely in memory. Merging two large, non-folding sources
is the classic slow refresh.

**Privacy levels** also affect this. When combining sources, Power Query's *firewall* stops
data from one source being sent to another (for instance, values from a private Excel file
being inserted into a SQL `WHERE`) unless the privacy levels allow it. The famous
*"Formula.Firewall: Query … references other queries or steps, so it may not directly access
a data source"* error is this protection: restructure so each query accesses one source,
then combine them in a separate query, or set privacy levels appropriately.

## Common mistakes

- **Expanding every column** from a merge "just in case," bloating the fact table.
- **Merging on keys with different types** (text vs number) — no rows match, silently.
- **Appending tables with different column names** and getting half-null columns.
- **Hard-coding server names** in 20 queries instead of one parameter.

## Exercise

1. Reproduce the Left Anti merge and confirm 2 orphan rows, 250 total.
2. Split `FactSales` into 2024 and 2025 queries with different column names (`Qty` vs
   `Units`), then fix and append them with `Table.Combine`.
3. Add the `SizeBand` custom column and confirm Large 2 / Medium 4 / Small 2 rows.
4. Create `fnLoadSales` and use it to load two CSV files.
