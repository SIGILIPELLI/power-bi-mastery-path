---
description: "Set Power Query data types correctly, handle locales, trim and split text, replace errors and nulls, and unpivot wide spreadsheets into tidy tables."
---

# 04 · Data Types & Cleaning

Wrong data types are the most common reason a Power BI total is wrong. A number stored as
text can't be summed; a date parsed as month/day instead of day/month silently moves
sales into the wrong month. This lesson is a checklist for turning a messy source into a
clean table.

## The types that matter

| Power Query type | Model type | Use for |
|------------------|-----------|---------|
| Whole Number (`Int64.Type`) | Whole number | Counts, IDs, quantities |
| Decimal Number (`type number`) | Decimal number | Measurements, ratios |
| Fixed Decimal Number (`Currency.Type`) | Fixed decimal (4 places) | Money; avoids binary floating-point artifacts |
| Date / Date/Time / Time | Same | Dates; prefer **Date** unless you really need time |
| Text (`type text`) | Text | Labels, codes (including ZIP codes and phone numbers) |
| True/False (`type logical`) | Boolean | Flags |

Two rules of thumb: keep **identifiers that look like numbers as text** when leading
zeros matter (`00742`), and **split date/time** into separate Date and Time columns if you
need both — a combined date-time column has far more distinct values and compresses
poorly (see Level 3).

## A messy sample

Save this as `messy_orders.csv`. It deliberately contains the usual problems:

```text
Order ID , Order Date ,Customer,Amount,Status
A-001,05/01/2025,  Asha Rao ,"1,200.50",shipped
A-002,17/01/2025,ben ortiz,300,Shipped
A-003,02/02/2025,Chen Li,n/a,SHIPPED
A-004,14/02/2025,  Asha Rao,450.25,cancelled
```

Problems: header names with spaces around them, dates in **day/month/year**, customer
names with stray spaces and inconsistent case, a thousands separator inside quotes, an
`n/a` in a numeric column, and inconsistent status casing.

## Step by step

1. **Get data → Text/CSV**, choose the file, **Transform Data**.
2. **Clean headers.** Double-click each header and retype it without spaces:
   `OrderID`, `OrderDate`, `Customer`, `Amount`, `Status`.
3. **Remove the auto type step** if it guessed wrong (click the ✕ next to `Changed Type`).
4. **Dates with a locale.** Right-click `OrderDate` → **Change Type → Using Locale…**,
   Data type **Date**, Locale **English (United Kingdom)**. Now `05/01/2025` becomes
   5 January 2025, not May 1.
5. **Trim and fix case.** Select `Customer` → **Transform → Format → Trim**, then
   **Format → Capitalize Each Word**. Do **Trim** and **lowercase** on `Status`.
6. **Numbers with separators.** Right-click `Amount` → **Change Type → Using Locale…**,
   **Fixed Decimal Number**, English (United States). `"1,200.50"` becomes 1200.5; `n/a`
   becomes an **Error**.
7. **Handle the error deliberately.** Right-click `Amount` → **Replace Errors**, value
   `null`. (Replacing with `0` would claim the order was worth nothing, which is a lie.)
8. Optionally add a flag: **Add Column → Conditional Column**, `AmountMissing` = `true`
   if `Amount` equals null, else `false`.

Expected result:

| OrderID | OrderDate | Customer | Amount | Status |
|---------|-----------|----------|--------|--------|
| A-001 | 2025-01-05 | Asha Rao | 1200.50 | shipped |
| A-002 | 2025-01-17 | Ben Ortiz | 300.00 | shipped |
| A-003 | 2025-02-02 | Chen Li | null | shipped |
| A-004 | 2025-02-14 | Asha Rao | 450.25 | cancelled |

Sum of Amount = 1200.50 + 300 + 450.25 = **1,950.75** (nulls are ignored by SUM). Distinct
customers = **3**; before trimming and case fixing there would have been 4 distinct
strings (`Asha Rao` with and without the leading spaces), which is why a
`DISTINCTCOUNT` on raw data overcounts.

The M for the key steps:

```powerquery
#"Typed Dates" = Table.TransformColumnTypes(Renamed, {{"OrderDate", type date}}, "en-GB"),
#"Cleaned Text" = Table.TransformColumns(#"Typed Dates", {
    {"Customer", each Text.Proper(Text.Trim(_)), type text},
    {"Status",   each Text.Lower(Text.Trim(_)), type text}}),
#"Typed Amount" = Table.TransformColumnTypes(#"Cleaned Text",
    {{"Amount", Currency.Type}}, "en-US"),
#"No Errors" = Table.ReplaceErrorValues(#"Typed Amount", {{"Amount", null}})
```

## Unpivoting wide data

Spreadsheets often arrive "wide":

| Product | Jan | Feb | Mar |
|---------|-----|-----|-----|
| Trail Tent | 240 | 120 | 360 |
| Headlamp | 125 | 100 | 0 |

Power BI wants it "long" — one row per product per month — so that `Month` is a column
you can filter and slice. Select `Product`, then **Transform → Unpivot Columns → Unpivot
Other Columns**. You get `Product`, `Attribute`, `Value`; rename them to `Product`,
`Month`, `Revenue`. Six rows result. Choose *Unpivot Other Columns* rather than selecting
the month columns, so an `Apr` column added next quarter is included automatically.

## How It Actually Works

A Power Query column type is more than a display format. `Table.TransformColumnTypes`
*converts every value* using the culture you supply (or the file's/system's default), and
any value that fails conversion becomes an **error value** stored in that cell — not a
failed query. Errors travel with the row until something reads them. That's why a single
bad cell can let the query "succeed" in the preview and then fail the load: when the data
is written to the model, an error cell either becomes a load error (reported in a
"View errors" link after Close & Apply) or blank, depending on the step.

The culture argument (`"en-GB"`) controls parsing rules: date component order, decimal
separator, thousands separator. Without it, Power Query uses the locale in **File →
Options → Current File → Regional Settings**, which differs between colleagues' machines —
a classic "works on my laptop" refresh failure.

Downstream, the model uses the type to choose storage. Text columns are dictionary-encoded
strings; whole numbers can use *value encoding* (stored as integers with an offset);
Fixed Decimal is stored as a scaled 64-bit integer (value × 10,000), which is why it has
exactly four decimal places and avoids rounding surprises in currency totals.

## Common mistakes

- **Letting type detection run on the first 200 rows** of a file whose later rows differ.
- **Replacing errors with 0** in measures of money or quantity, which silently changes
  averages.
- **Using Replace Values on "n/a" after typing the column** — by then it's already an
  error, not text. Order of steps matters: clean text first, then type.
- **Changing types in the Data view instead of Power Query.** The model type changes, but
  the source values are still parsed by Power Query's rules.

## Exercise

1. Clean `messy_orders.csv` as above and confirm the sum of `Amount` is 1,950.75 and the
   distinct count of `Customer` is 3.
2. Change the `OrderDate` locale to English (United States) and note which rows become
   errors and which silently get the wrong date. Explain why `17/01/2025` behaves
   differently from `05/01/2025`.
3. Build the wide product-by-month table with **Enter data**, unpivot it, and add an
   `Apr` column to the source to prove the unpivot picks it up.
