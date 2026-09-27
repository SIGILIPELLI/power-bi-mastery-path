---
description: "Make Power BI fast: how VertiPaq compresses columns, why cardinality drives model size, storage vs formula engine, Performance Analyzer and DAX Studio."
---

# 04 · Performance: VertiPaq & Performance Analyzer

Slow Power BI reports usually have one of three causes: a **model that's bigger than it needs
to be**, **DAX that forces the formula engine to do row-by-row work**, or **too many visuals
firing too many queries**. You can't fix any of them by guessing. This lesson explains how
the VertiPaq engine stores data — so you can predict what's expensive — and how to measure
with Performance Analyzer and DAX Studio.

## How VertiPaq stores a column

Imported tables are stored **column by column**, and each column is compressed on its own in
roughly three stages.

### 1. Encoding: dictionary or value

**Dictionary (hash) encoding** — the engine builds a dictionary of the column's distinct
values and stores, for each row, the integer ID of its value. Take `Sales[Color]` from the
Level 3 sample:

| Row | Color | Stored ID |
|----:|-------|----------:|
| 1 | Red | 0 |
| 2 | Blue | 1 |
| 3 | Red | 0 |
| 4 | Blue | 1 |
| 5 | Red | 0 |

Dictionary: {0 → Red, 1 → Blue}. Two distinct values need only **1 bit** per row. In general
the number of bits per row is about log₂(distinct values), rounded up: 1,000 distinct values
→ 10 bits; 1 million → 20 bits. Text columns always use dictionary encoding.

**Value encoding** — for integer columns where arithmetic can shrink the range, the engine
stores `value − offset` (and possibly divided by a common factor) directly, with no dictionary.
A column of order numbers from 1,000,000 to 1,000,999 can be stored as 0–999, in 10 bits.

### 2. Run-length encoding (RLE)

If consecutive rows have the same ID, the engine stores runs: *(value, count)*. The Color IDs
above (0,1,0,1,0) have no runs — five entries. Sorted by Color they become (0,0,0,1,1) → two
runs: *(Red × 3), (Blue × 2)*. On a real fact table, a low-cardinality column sorted well can
compress from millions of entries to a handful of runs. The engine chooses a sort order at
processing time to maximize compression; you influence it only indirectly (low cardinality,
fewer columns).

### 3. Bit-packing

Whatever remains is packed into the minimal number of bits per entry.

### Segments

Tables are split into **segments** (row groups of on the order of a million rows or more,
depending on configuration and model format). Each segment is compressed independently, and
queries scan segments in parallel — one reason the storage engine is multi-threaded.

## Cardinality is the cost driver

Because bits-per-row ≈ log₂(distinct values) and the dictionary stores each distinct value,
**the number of distinct values** matters far more than the number of rows.

### Worked example: splitting a date-time column

A fact table of 10 million rows has `OrderDateTime` with second precision over 10 years.
Distinct values could approach the row count — say ~10 million.

| Design | Distinct values | Bits per row (≈ log₂) | Dictionary size |
|--------|----------------:|----------------------:|-----------------|
| One `OrderDateTime` column | ~10,000,000 | 24 | ~10 M entries |
| `OrderDate` (Date) | 3,653 | 12 | tiny |
| `OrderTime` (to the second) | 86,400 | 17 | small |
| `OrderTime` rounded to the minute | 1,440 | 11 | tiny |

Per-row storage drops from 24 bits to 12 + 11 = 23 bits in the minute case — similar — but
the **dictionary** shrinks from ~10 million entries to about 5,000, and the separate columns
compress far better with RLE because dates repeat in long runs. In practice this split is one
of the largest single wins in real models. (These numbers are illustrations of the arithmetic,
not measurements; measure your own model with VertiPaq Analyzer.)

Other cardinality fixes:

- Remove columns nobody uses (every column costs memory, even hidden ones).
- Round decimals to the precision you need.
- Avoid high-cardinality text on facts (GUIDs, free-text comments, URLs) unless essential.
- Prefer measures over calculated columns on large fact tables.
- Turn off **Auto date/time** (hidden date tables per date column).

## Storage engine vs formula engine

Every DAX query is split between:

- **Storage engine (SE)** — scans compressed columns, filters, groups, and computes simple
  aggregates. Multi-threaded; results cached.
- **Formula engine (FE)** — everything else: complex logic, iterating datacaches, combining
  results. Single-threaded.

A fast query does most of its work in a few SE scans. A slow one typically shows **many SE
queries** (for example, one per product because of context transition in a big iterator) or
**large datacaches** materialized for the FE, or **callbacks** from SE into FE.

## Measuring: Performance Analyzer

1. **View → Performance Analyzer → Start recording**.
2. Click **Refresh visuals** (or interact with a slicer).
3. Each visual lists durations for **DAX query**, **Visual display**, and **Other** (time waiting
   for other visuals, queueing and preparation).
4. **Copy query** on the slowest visual.

Reading it:

- High **DAX query** → model or measure problem. Take the query to DAX Studio.
- High **Visual display** → too many data points, or a heavy custom visual.
- High **Other** → too many visuals competing; reduce visual count or combine cards.

## Measuring: DAX Studio (external tool)

DAX Studio is a free, open-source Windows tool that connects to the model Desktop is running.
Conceptually:

- **Server Timings**: paste the copied query, enable Server Timings, run it. You'll see total
  time split into FE and SE, the number of SE queries, and each SE query's xmSQL text and
  rows returned. Look for many SE queries or huge row counts.
- **VertiPaq Analyzer** (Advanced → View Metrics): lists every table and column with
  cardinality, total size, dictionary size, and encoding. Sort by size; the top few columns are
  usually the whole story.
- **Clear cache** before each timed run so you measure cold-cache performance, which is what the
  first user of the morning experiences.

## Worked example: a slow measure and its fix

On a large fact table:

```dax
-- Slow: context transition per fact row
Revenue Big Customers Slow =
SUMX ( FILTER ( Sales, [Customer Revenue] > 1000 ), Sales[Amount] )
```

`[Customer Revenue]` is a measure, so every fact row triggers context transition. Iterate
customers instead:

```dax
Revenue Big Customers =
VAR BigCustomers =
    FILTER ( VALUES ( Customer[CustomerKey] ), [Customer Revenue] > 1000 )
RETURN
    CALCULATE ( SUM ( Sales[Amount] ), BigCustomers )
```

Now the transition happens once per customer (thousands, not millions), and the final
`SUM` is one SE scan filtered by the customer list. In Server Timings you'd expect far fewer SE
queries. The results are identical when `Customer Revenue` is defined per customer; confirm that
with a small test before swapping.

## How It Actually Works

When a visual query arrives, the formula engine builds a plan and issues **xmSQL** requests to
the storage engine — an internal, SQL-like language with `SELECT … FROM … WHERE`, simple
aggregates, and joins along relationships. For each request, the SE scans only the columns
named, segment by segment in parallel. Filters are evaluated against the **dictionary** first
(finding the IDs for "Red"), then the compressed column is scanned for those IDs, often without
decompressing RLE runs (a run of 1 million "Red" IDs is one comparison). Results come back as
small uncompressed datacaches, which the FE combines.

Compression, therefore, isn't only about memory: fewer bits and longer runs mean less data to
scan per query. That's why the same DAX over a well-shaped model (low-cardinality columns,
narrow fact tables, integer keys) is fast, and over a wide, high-cardinality table is slow.

## Common mistakes

- Optimizing DAX before checking model size and column cardinality.
- Timing with a warm cache and declaring victory.
- Dozens of cards, each a separate query — consider one multi-row card or a table.
- Keeping `SalesKey`/GUID columns nobody uses on a large fact table.

## Exercise

1. Using the Level 1 or Level 2 model, record Performance Analyzer for one page and list the
   three slowest visuals with their DAX-query and other times.
2. If you can run DAX Studio, open VertiPaq Analyzer and write down the three largest columns
   and their cardinality. If you can't, compute by hand the bits per row for columns with 2, 50,
   and 70,000 distinct values (1, 6, 17).
3. For the five-row `Sales[Product]` column (A, A, B, C, C), write the dictionary, the ID
   sequence, and the RLE runs. (Dictionary A→0, B→1, C→2; IDs 0,0,1,2,2; runs (0×2),(1×1),(2×2).)
