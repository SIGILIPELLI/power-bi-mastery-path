---
description: "Where Power BI fits in Microsoft Fabric: OneLake, lakehouses, warehouses, Direct Lake semantic models, capacity, and what changes for BI teams."
---

# 02 · Power BI in Microsoft Fabric

Microsoft Fabric is Microsoft's unified analytics platform, and Power BI is one of its
workloads. For a Power BI developer, Fabric mostly means three things: data can live in a
single organizational lake (**OneLake**) in open **Delta** format; semantic models can read that
data through a mode called **Direct Lake**; and the capacity you buy is shared by all the
workloads.

!!! warning "A fast-moving platform"
    Fabric has changed substantially since it became generally available, with new item types,
    renamed features and shifting licensing details. This lesson focuses on the concepts that
    have been stable and names features as they're generally known at the time of writing.
    Verify specifics — especially licensing, limits, and preview/GA status — in Microsoft's
    current documentation before designing anything.

## The building blocks

| Fabric item | What it is | Relevance to Power BI |
|-------------|-----------|------------------------|
| **OneLake** | One logical data lake per tenant, organized by workspace; stores tables as Delta/Parquet | The common storage everything reads from |
| **Lakehouse** | Files + Delta tables, engineered with Spark/notebooks; comes with a SQL analytics endpoint (read-only T-SQL) | A source for semantic models |
| **Warehouse** | A T-SQL data warehouse storing its tables in OneLake | A source for semantic models; familiar to SQL teams |
| **Dataflow Gen2** | Power Query at scale, with output destinations such as a lakehouse | Moves Power Query logic upstream |
| **Data pipeline** | Orchestration (copy, schedule, dependencies) | Runs loads before model refresh |
| **Semantic model** | The same Power BI model you know | Import, DirectQuery, or **Direct Lake** |
| **Shortcuts** | References to data in other OneLake locations or external storage, without copying | Share tables across workspaces/domains |

## Direct Lake, conceptually

Import mode copies data into VertiPaq during refresh. DirectQuery leaves data in the source and
translates each query into SQL. **Direct Lake** is a third option for Delta tables in OneLake:

- The model's tables point at Delta tables.
- When a query needs a column, the engine loads that column's data **from the Parquet files** into
  memory in VertiPaq's format — no scheduled import of the full table, and no SQL translation.
- A "refresh" of a Direct Lake model is typically a lightweight **framing** operation: the model
  records which version of the Delta tables to use. New data becomes visible after framing, which can
  happen automatically when the underlying tables change (a setting).
- If a query can't be served in Direct Lake mode (for example, some features or limits are exceeded),
  models on the SQL endpoint have historically been able to **fall back** to DirectQuery; behaviour and
  configuration options here have evolved, so check the current rules.

### Worked example: choosing a mode for a 2-billion-row fact

| Option | What happens | Main risk |
|--------|--------------|-----------|
| Import | Nightly refresh copies 2 B rows into VertiPaq | Refresh time and memory; needs incremental refresh (lesson 06) |
| DirectQuery on the warehouse | Every visual → SQL | Query latency and warehouse load under concurrency |
| Direct Lake on the lakehouse tables | Columns paged into memory on demand from Delta | Guardrails on table size/row groups per capacity size; Delta table quality (file sizes, V-Order) matters |

A reasonable Fabric-era design: engineer the fact as a well-maintained Delta table (sensible file
sizes, optimized layout), model it in Direct Lake, keep small reference tables in the same lakehouse,
and test performance on the target capacity size before committing.

## What changes for Power BI teams

- **Transformations move upstream more easily** — Dataflow Gen2 or notebooks write to a lakehouse,
  and the model reads the result. Power Query inside the semantic model gets thinner.
- **Capacity is shared** — a Spark job and your report queries consume the same capacity units.
  Monitoring (the Fabric Capacity Metrics app) becomes a BI team concern.
- **Model authoring** — semantic models can be created and edited in the browser (web modeling),
  and Direct Lake models are edited there or with tools connected via the XMLA endpoint; Desktop
  support for Direct Lake editing has been growing. Check which your tenant supports.
- **Git integration and deployment pipelines** span more item types (lesson 07).
- **Governance** — domains, endorsement and lineage apply across Fabric items, not just reports.

## How It Actually Works

Delta tables are Parquet files plus a transaction log (`_delta_log`) that records which files make up
each table version. Parquet is itself columnar: each file stores row groups, and within each row group,
each column is stored and compressed separately. That columnar layout is the key to Direct Lake —
VertiPaq needs data column by column, and Parquet already provides it column by column.

When a Direct Lake query touches `Sales[Amount]`, the engine reads that column's chunks from the Parquet
files listed in the framed table version, **transcodes** them into VertiPaq's in-memory encoding
(dictionaries, bit-packed IDs), and keeps them resident while memory allows. Columns never queried
are never loaded. Under memory pressure, columns can be evicted and paged back later. This is why
Direct Lake performance depends on Delta *physical* quality — many tiny files or huge unoptimized row
groups make loading slower — and why Microsoft applies an optimization called V-Order when Fabric
engines write Parquet, which sorts and encodes data in a way that is friendlier to VertiPaq.

Framing is cheap because it's metadata: the model reads the Delta log, notes the current version and
its file list, and discards any cached column data that no longer matches. No rows move.

## Common mistakes

- Assuming Direct Lake is "Import without refresh" in every respect — it has its own guardrails and
  depends on the Delta layer's quality.
- Letting data engineering and BI teams build separate copies of the same tables in different
  workspaces instead of using shortcuts.
- Ignoring capacity monitoring after mixing Spark and BI workloads.
- Treating preview features as production-ready.

## Exercise

1. Using Microsoft's current Fabric documentation, write down for your region/tenant: whether Fabric is
   enabled, which capacity SKU you'd need for a trial project, and the current Direct Lake guardrails
   for that SKU. Note the date you checked.
2. Redraw the architecture diagram from lesson 01 as a Fabric design: which items (lakehouse,
   warehouse, Dataflow Gen2, pipeline, semantic model) would you use at each layer?
3. For the 2-billion-row example, write a half-page recommendation choosing Import, DirectQuery or
   Direct Lake, including what you would measure in a proof of concept.
