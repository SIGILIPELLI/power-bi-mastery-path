---
description: "Level 1 of Power BI: Desktop vs service, getting data, Power Query, data types, first visuals, slicers, measures vs columns, formatting, and publishing."
---

# Level 1 · Entry <span class="level-badge">Foundations</span>

Goal: go from a raw CSV file to a clean, published, filterable sales dashboard — and
understand each step well enough that you could explain why the numbers are right.

Most beginner material treats Power BI as a charting tool. This level treats it as a
pipeline with three stages you will keep returning to for the rest of the course:

1. **Get and transform** — Power Query pulls data from a source and applies a recorded
   list of cleaning steps.
2. **Model** — the cleaned tables are loaded into an in-memory model where you add
   relationships and DAX measures.
3. **Visualize and share** — report pages query the model; the service publishes,
   refreshes and shares them.

A mistake at stage 1 (a number stored as text, a date parsed in the wrong locale) shows
up as a wrong total at stage 3, so the early lessons are deliberately careful about
types and cleaning.

## Modules

1. [What Power BI Is: Desktop, Service & Mobile](01-what-is-power-bi.md) — the pieces of the platform, what each is for, and how licensing works in general terms
2. [Getting Data](02-getting-data.md) — connectors, CSV and Excel imports, the Navigator, and Import vs DirectQuery at a first glance
3. [Power Query Basics](03-power-query-basics.md) — the editor, applied steps, and the M code each click writes
4. [Data Types & Cleaning](04-data-types-cleaning.md) — types, locales, trimming, splitting, replacing errors, and unpivoting
5. [Building Your First Visuals](05-first-visuals.md) — choosing a visual, field wells, implicit aggregation, and the model query behind each chart
6. [Filters & Slicers](06-filters-slicers.md) — visual, page and report filters, slicers, and cross-filtering between visuals
7. [Measures vs Calculated Columns](07-measures-vs-calculated-columns.md) — the most important distinction in Power BI, with worked examples
8. [Report Formatting & Layout](08-report-formatting.md) — canvas settings, themes, number formats, alignment, and a layout grid
9. [Publishing & Sharing Basics](09-publishing-sharing.md) — workspaces, publishing, reports vs semantic models vs dashboards, and sharing options
10. [Project — A Sales Dashboard from CSV](10-project-sales-dashboard.md) — build, verify and publish a one-page dashboard from a sample file defined in the lesson

## What you need before starting

- A Windows PC (or Windows virtual machine) for **Power BI Desktop**, which is a free
  download from Microsoft. Desktop does not run natively on macOS or Linux.
- A work or school Microsoft account if you want to publish to the Power BI service.
  Personal consumer email addresses generally cannot sign up for the service; lesson 09
  explains the options.
- Comfort with spreadsheets: sorting, filtering, and SUM/AVERAGE-style formulas.

## The sample data used in this level

Most lessons use one small file, `trailhead_sales.csv`, for an imaginary outdoor-gear
shop. It is short enough to check every total by hand:

```text
OrderID,OrderDate,Region,Product,Category,Units,UnitPrice
1001,2025-01-05,North,Trail Tent,Camping,2,120.00
1002,2025-01-09,South,Rain Jacket,Apparel,3,80.00
1003,2025-01-17,North,Headlamp,Accessories,5,25.00
1004,2025-02-02,East,Trail Tent,Camping,1,120.00
1005,2025-02-14,South,Headlamp,Accessories,4,25.00
1006,2025-02-20,East,Rain Jacket,Apparel,2,80.00
1007,2025-03-03,North,Rain Jacket,Apparel,1,80.00
1008,2025-03-11,South,Trail Tent,Camping,3,120.00
```

Save it as a UTF-8 text file. Revenue (Units × UnitPrice) per row is 240, 240, 125, 120,
100, 160, 80 and 360, for a grand total of **1,425**. You will see that number many
times; if your report shows something else, something upstream is wrong.
