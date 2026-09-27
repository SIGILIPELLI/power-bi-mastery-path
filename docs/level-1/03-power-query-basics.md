---
description: "Learn the Power Query editor: applied steps, the formula bar, the M code behind every click, column profiling, and why step order matters."
---

# 03 · Power Query Basics

Power Query is the "get and transform" layer of Power BI (the same engine lives in Excel
and in Fabric dataflows). You describe a series of transformations; Power Query records
them as **applied steps** and replays the whole recipe from the source every refresh.
Nothing you do in the editor changes the source file.

## Opening the editor

- **Home → Transform data** opens the Power Query Editor window.
- Left: the **Queries** pane (one entry per table, plus parameters and functions).
- Centre: a preview of the current step's output (not necessarily every row).
- Right: **Query Settings**, with the query name and the list of **Applied Steps**.
- Top: the **formula bar** (turn it on with **View → Formula Bar** if hidden), which shows
  the M expression of the selected step.

## Steps are a recipe

With the `Sales` query from lesson 02 selected, you probably already see:

1. `Source`
2. `Promoted Headers`
3. `Changed Type`

Click any step and the preview shows the table *as of that step*. Add new steps with the
ribbon or the column right-click menu; they are appended after the selected step (Power
Query will warn you if you insert in the middle).

## Step by step: a first transformation

We want a `Revenue` column and only rows with at least one unit.

1. Select the `Units` column header, open its filter arrow, and choose
   **Number Filters → Greater Than Or Equal To…**, value `1`. A step named
   `Filtered Rows` appears.
2. **Add Column → Custom Column**. Name it `Revenue`, formula:
   `[Units] * [UnitPrice]`. Click OK.
3. The new column has type "any" (the `ABC123` icon). Click the icon in the header and
   choose **Decimal Number** (or **Fixed Decimal Number** for currency).
4. Right-click the `Filtered Rows` step → **Rename**, call it `Only rows with units`.
   Descriptive step names are documentation for the next person.
5. **Home → Close & Apply**.

For our sample, all 8 rows have Units ≥ 1, so nothing is filtered; `Revenue` should
contain 240, 240, 125, 120, 100, 160, 80, 360.

## Reading the M

Open **Home → Advanced Editor**. After the steps above, the query looks like this
(line breaks added for readability):

```powerquery
let
    Source = Csv.Document(File.Contents("C:\Data\trailhead_sales.csv"),
        [Delimiter = ",", Columns = 7, Encoding = 65001, QuoteStyle = QuoteStyle.None]),
    #"Promoted Headers" = Table.PromoteHeaders(Source, [PromoteAllScalars = true]),
    #"Changed Type" = Table.TransformColumnTypes(#"Promoted Headers", {
        {"OrderID", Int64.Type}, {"OrderDate", type date}, {"Region", type text},
        {"Product", type text}, {"Category", type text}, {"Units", Int64.Type},
        {"UnitPrice", type number}}),
    #"Only rows with units" = Table.SelectRows(#"Changed Type", each [Units] >= 1),
    #"Added Custom" = Table.AddColumn(#"Only rows with units", "Revenue",
        each [Units] * [UnitPrice]),
    #"Changed Type1" = Table.TransformColumnTypes(#"Added Custom",
        {{"Revenue", type number}})
in
    #"Changed Type1"
```

Things worth noticing:

- A query is one `let … in` expression. Each step is a named variable; each step
  usually takes the previous step's name as its first argument.
- Names with spaces are written `#"Like This"`.
- `each [Units] >= 1` is shorthand for a function `(_) => _[Units] >= 1`, called once per
  row. `[Units]` means "the Units field of the current row."
- The `in` clause names which step is the output — normally the last one.

You can add the column type directly in `Table.AddColumn` to save a step:
`Table.AddColumn(prev, "Revenue", each [Units] * [UnitPrice], type number)`.

## Column profiling

Under **View**, enable **Column quality**, **Column distribution** and **Column profile**.
You will see percentages of Valid / Error / Empty per column and a count of distinct
values. Note the status bar text: *"Column profiling based on top 1000 rows"*. Click it
to switch to **entire data set** when you need true counts — slower, but honest.

## How It Actually Works

M is a **functional, lazily evaluated** language. The `let` block is not executed top to
bottom like a script. When Power Query needs the output, it evaluates the expression named
after `in`, which demands the step it depends on, and so on back to `Source`. A step no
one references is never evaluated at all.

Because evaluation is lazy and the steps form an expression, Power Query can look at the
whole chain before running it. For foldable sources (databases, some OData feeds), it
tries to translate the chain into a single native query — `Table.SelectRows` becomes a
`WHERE`, `Table.AddColumn` with arithmetic becomes a computed column in `SELECT`. This is
**query folding**. Right-click a step and choose **View Native Query**: if it is enabled,
that step folded; if it is greyed out, it either didn't fold or the connector can't show
you. Newer releases also show *step folding indicators* next to each step.

For a CSV there is nothing to fold to, so Power Query reads the file and applies every
step itself, streaming rows through the chain where it can. Some operations (sorting,
grouping, pivoting) need to see all rows before producing output, so they buffer data in
memory — that's why a sort early in a big query is costly.

The preview you see in the editor is a *separate* evaluation limited to a sample of rows,
with results cached on disk. That is why a preview can look fine while the full refresh
fails on a bad row deep in the file.

## Common mistakes

- **Deleting a step in the middle** and breaking later steps that reference a column it
  created. Power Query shows an error on the later step; read it before panicking.
- **Letting auto "Changed Type" run twice** — once after headers, and again after you
  add columns — with hard-coded column lists. If a source column is renamed, the step
  fails. Keep one deliberate type step per stage.
- **Sorting in Power Query "for the report."** Sort order in Power Query does not control
  the order in visuals. Sort in the visual or with **Sort by column** in the model.
- **Trusting the 1,000-row profile** for "no errors" on a million-row table.

## Exercise

1. Recreate the steps above and compare your Advanced Editor text with the listing.
   Differences in auto-generated names are fine; differences in logic are not.
2. Add a step with **Add Column → Column From Examples**: type `Jan` in the first row next
   to a January date and let Power Query infer a month-name column. Open the formula bar
   and write down the M function it generated.
3. In the Advanced Editor, add `type number` as the fourth argument of `Table.AddColumn`
   and delete the separate `Changed Type1` step. Make sure the query still ends with a
   step that exists.
