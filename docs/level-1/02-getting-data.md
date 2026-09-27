---
description: "Connect Power BI Desktop to CSV, Excel, folders and databases; use the Navigator; and understand Import vs DirectQuery before loading anything."
---

# 02 · Getting Data

Every Power BI project starts with **Get data**. The dialog lists well over a hundred
connectors, but they fall into a few families, and the choices you make at this moment —
which connector, which storage mode, "Load" or "Transform" — are harder to change later
than they look.

## Connector families

| Family | Examples | Notes |
|--------|----------|-------|
| Files | Text/CSV, Excel workbook, XML, JSON, Parquet, PDF | Path is stored in the query; moving the file breaks refresh. |
| Folders | Folder, SharePoint folder | Combine many files with the same shape into one table. |
| Databases | SQL Server, PostgreSQL, MySQL, Oracle, Snowflake, Databricks | Usually support **query folding** and often DirectQuery. |
| Online services | SharePoint Online list, Dataverse, Salesforce, Google Analytics | Authentication through an organizational or OAuth account. |
| Power Platform / Fabric | Semantic models, dataflows, lakehouses, warehouses | Reuse data other people already prepared. |
| Other | Web, OData, blank query | Blank query lets you type M directly. |

## Step by step: importing the sample CSV

Using `trailhead_sales.csv` from the [Level 1 overview](index.md):

1. In Power BI Desktop, go to **Home → Get data → Text/CSV** (on some releases the
   ribbon shows a "Get data" dropdown with CSV listed under "Common data sources").
2. Browse to the file and click **Open**.
3. A preview dialog appears. Check three settings at the top:
    - **File origin** — should be `65001: Unicode (UTF-8)` for a UTF-8 file. The wrong
      encoding shows up as garbled accented characters.
    - **Delimiter** — `Comma`.
    - **Data type detection** — "Based on first 200 rows" by default.
4. Choose **Transform Data** rather than **Load**. This opens Power Query so you can check
   types before anything reaches the model. (Load is fine for a file you already trust.)
5. In Power Query, confirm the query is named something meaningful — rename
   `trailhead_sales` to `Sales` in the **Query Settings** pane.
6. Click **Home → Close & Apply**.

The **Data** view (the table icon on the left rail) should now show 8 rows. In the
**Model** view you will see one table named `Sales` with seven columns.

## The Navigator (Excel and databases)

For sources that contain several objects — an Excel workbook with sheets and tables, or a
database with many tables — Power BI shows the **Navigator**. Tick the objects you want.

- In Excel, prefer a **formatted table** (the "Table" icon, created in Excel with
  Ctrl+T) over a **sheet**. A table has fixed headers and grows cleanly; a sheet includes
  every stray cell someone typed in column Z.
- In databases, prefer **views** or specific tables over "select everything," and use
  the advanced options only when you must write native SQL (which can block folding —
  see the next lesson).

## Import vs DirectQuery, first look

Some connectors ask for a **Data Connectivity mode**:

| | Import | DirectQuery |
|---|--------|-------------|
| Where data lives | Copied into the model (compressed, in memory) | Stays in the source; each visual sends a query to it |
| Freshness | As of last refresh | Near real time |
| Speed | Usually fastest | Depends on the source database |
| DAX and Power Query features | Full | Some transformations and functions are restricted |
| Model size | Limited by license/capacity | Not limited in the same way |

Start with **Import** unless you have a clear reason not to. Level 3 revisits this with
composite models.

## Worked example: combining a folder of monthly files

Suppose finance drops one CSV per month into a folder, each with identical columns:

```text
C:\Data\TrailheadMonthly\
    sales_2025_01.csv
    sales_2025_02.csv
    sales_2025_03.csv
```

1. **Get data → Folder**, pick the folder, then **Combine & Transform Data**.
2. Power Query picks a *sample file*, lets you set delimiter and encoding once, and builds
   a helper function (under a group called **Helper Queries**) that it applies to every
   file.
3. The result is a single table with a `Source.Name` column that tells you which file each
   row came from — useful for debugging.

Next month, a fourth file dropped into the folder appears automatically on refresh. If
the three files together contain the eight rows from the sample, your combined table
should have 8 rows and total revenue 1,425.

## How It Actually Works

Each connector is implemented as an **M function** in Power Query's library. When you pick
Text/CSV, Power Query writes something like this into the query (visible later in the
Advanced Editor):

```powerquery
let
    Source = Csv.Document(
        File.Contents("C:\Data\trailhead_sales.csv"),
        [Delimiter = ",", Columns = 7, Encoding = 65001, QuoteStyle = QuoteStyle.None]
    ),
    #"Promoted Headers" = Table.PromoteHeaders(Source, [PromoteAllScalars = true])
in
    #"Promoted Headers"
```

`File.Contents` returns raw bytes; `Csv.Document` parses them into a table using the
options you picked in the dialog; `Table.PromoteHeaders` turns the first row into column
names. The encoding number (65001) is the Windows code page for UTF-8.

**Credentials are not stored in the query.** Power BI keeps a separate list of *data source
settings* keyed by the source path or server name, with the authentication method and
privacy level for each (**File → Options and settings → Data source settings**). That is
why renaming a server in the M code prompts you for credentials again, and why the
service asks you to re-enter credentials after publishing: the cloud has its own
credential store.

For database connectors, the connector translates later steps into the source's query
language when it can (query folding). For file connectors there is no engine to delegate
to, so every step runs in the Power Query engine itself.

## Common mistakes

- **Pointing at a file in your Downloads folder or on a mapped drive letter.** The service
  cannot reach `C:\Users\you\Downloads`. For scheduled refresh, put files in SharePoint/
  OneDrive or on a share reachable by a gateway.
- **Clicking Load on messy data.** Types get auto-detected from the first 200 rows; a text
  value in row 5,000 becomes an error you won't see until later.
- **Importing an Excel *sheet* with notes and totals rows.** The totals row gets summed again.
- **Leaving `Column1`-style headers** because the first row wasn't promoted.

## Exercise

1. Import `trailhead_sales.csv` with **Transform Data**, rename the query to `Sales`, and
   load it. Confirm 8 rows in Data view.
2. Split the eight rows into three monthly files (January, February, March) in a folder
   and import them with the **Folder** connector. Confirm the combined table has 8 rows
   and that `Source.Name` shows three distinct values.
3. Open **Data source settings** and write down which authentication method and privacy
   level each of your two sources uses.
