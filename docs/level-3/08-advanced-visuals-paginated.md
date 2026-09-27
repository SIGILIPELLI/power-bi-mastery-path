---
description: "Advanced Power BI visuals: decomposition tree, key influencers, small multiples, measure-driven formatting, custom visuals, and paginated reports."
---

# 08 · Advanced Visuals, Custom Visuals & Paginated Reports

The default bar, line and table visuals cover most needs. This lesson covers the tools for
the rest: built-in analytical visuals, conditional formatting driven by measures, small
multiples, third-party custom visuals (and their governance), and paginated reports for
pixel-perfect, printable output.

## Conditional formatting driven by measures

Nearly every colour, icon and data-bar setting accepts **fx** (field value), which means a
measure can decide it. For the Level 2 project's matrix:

```dax
Achievement Color =
VAR A = [Achievement %]
RETURN
    SWITCH (
        TRUE (),
        ISBLANK ( A ), BLANK (),
        A >= 0.95, "#2E7D32",   -- green
        A >= 0.80, "#F9A825",   -- amber
        "#C62828"               -- red
    )
```

Apply: select the matrix → **Format → Cell elements → Background color → fx** → Format style
**Field value** → `[Achievement Color]`. With the Level 2 project numbers: Accessories 96.2% →
green, Apparel 91.4% → amber, Camping 75.9% → red; January 97.7% green, February 92.0% amber,
March 63.2% red.

Keeping colour rules in a measure means they're consistent across visuals and testable.
Always pair colour with a second cue (an icon or the number itself) for accessibility.

## Built-in analytical visuals

- **Decomposition tree** — the reader picks the next dimension to split a measure by,
  interactively, and can ask for "high value" / "low value" splits. Good for ad-hoc root-cause
  exploration of one measure.
- **Key influencers** — runs a statistical analysis to find which factors are associated with a
  metric being high/low or a category occurring. Treat results as **leads to investigate, not
  proof of causation**; with small data it can find spurious patterns.
- **Q&A** — a natural-language box; quality depends heavily on model naming and synonyms.
- **Smart narrative / narrative visuals** — generated text summaries. Check every generated
  sentence against the numbers before publishing a report built around one; Microsoft has also
  been folding Copilot features into these areas, which vary by license and region.

## Small multiples

Line, bar, column and area charts have a **Small multiples** well. Put `DimStore[Region]` there
and a line of monthly Sales Amount in each panel — one chart per region, shared axes. Shared axes
make comparisons honest; turn off shared Y only when magnitudes differ so much that small regions
become flat lines, and say so in the title.

## Custom visuals (AppSource and organizational)

**Visualizations pane → … → Get more visuals** opens AppSource. Custom visuals are JavaScript
packages (`.pbiviz`) that run in a sandboxed iframe inside the report.

Governance points:

- **Certified** visuals have passed Microsoft's code review and requirements (for example, not
  accessing external services). Only certified visuals support some features such as export to
  PowerPoint/PDF with the visual rendered — check current documentation for the list.
- Admins can restrict visuals to an **organizational visuals** list and block uncertified ones.
- A visual that sends data to an external service is a data-protection question, not a design
  question. Read its privacy terms.
- Each custom visual adds load time. Prefer built-in visuals when they can do the job.

Building your own uses the `powerbi-visuals-tools` Node.js package (`pbiviz new`, `pbiviz start`,
`pbiviz package`) and TypeScript/D3. That's a development project in its own right; this course
doesn't cover it.

## Paginated reports

Power BI reports are interactive canvases; **paginated reports** are the SQL Server Reporting
Services lineage: page-oriented layouts that expand to print every row across as many pages as
needed, with headers, footers, page numbers and exact positioning.

| Need | Power BI report | Paginated report |
|------|-----------------|------------------|
| Explore, slice, drill | ✔ | Limited (parameters) |
| Print/PDF a 40-page invoice list | Awkward | ✔ |
| Operational list with every row | Table visual (scroll) | ✔ |
| Pixel-exact regulatory form | ✗ | ✔ |
| Scheduled email of a PDF/Excel export | Subscriptions exist but render the canvas | ✔ core use case |

Authoring uses **Power BI Report Builder** (free, Windows). The workflow:

1. **Data source**: connect to a published Power BI semantic model (or SQL, etc.).
2. **Dataset**: a query — for a semantic model, built with the query designer or written as DAX:

```dax
EVALUATE
SUMMARIZECOLUMNS (
    DimStore[Region],
    DimProduct[Category],
    "Sales", [Sales Amount],
    "Target", [Target Amount]
)
ORDER BY DimStore[Region], DimProduct[Category]
```

3. **Parameters**: e.g. `@Region`, used to filter the dataset.
4. **Layout**: a tablix (table/matrix) with row groups, page breaks between groups, header and
   footer with `Globals!PageNumber` and `Globals!TotalPages`.
5. Publish to a workspace. Paginated reports historically required Premium/capacity; licensing has
   broadened over time, so check what your tenant supports.

## How It Actually Works

A Power BI report visual asks for **only the data it can draw**, with data-reduction limits, and
re-queries on every interaction. A paginated report runs its dataset queries **once per
rendering**, pulls back the full result set (every row), and a layout engine then flows that data
into pages: each tablix grows downward, and when content hits the page's body height, the renderer
starts a new page, repeating headers where configured. Rendering extensions produce PDF, Excel,
Word, CSV and so on from the same layout tree. That's why paginated reports are the right tool
for "all 12,000 rows, printed" and the wrong one for interactive exploration.

Custom visuals are isolated for security: the host passes them a **data view** (the query result
shaped for the visual) through an API, and the visual renders into its own sandboxed iframe with
restricted access to the page. The visual never talks to the semantic model directly — it can
only draw what the host's query returned, subject to the same RLS as any other visual.

## Common mistakes

- Colour as the only signal of good/bad.
- Treating key influencers' output as causal findings.
- Installing uncertified visuals in regulated environments without review.
- Trying to make a Power BI canvas behave like a printable multi-page document.

## Exercise

1. Build `Achievement Color` and apply it to the Level 2 project matrix; verify the colours listed
   above.
2. Create a small-multiples line chart of monthly sales by region and write one sentence on
   whether shared Y-axes help or hurt here.
3. If you have Report Builder, build a one-page paginated report of Sales and Target by Region and
   Category with a page break per region. Otherwise, sketch the layout and list the dataset columns
   and parameters.
