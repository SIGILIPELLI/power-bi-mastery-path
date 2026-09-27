---
description: "Design Power BI dashboards that communicate: audience and questions first, visual hierarchy, choosing honest charts, context with targets, and annotation."
---

# 04 · Data Storytelling & Dashboard Design

A technically perfect model can still produce a report nobody uses. The failure is almost never
the chart library; it's that the page doesn't answer the reader's question quickly, or it answers
ten questions at once. This lesson is a working method for designing pages that communicate, with
a before/after redesign using the Level 2 project numbers.

## Start with the reader, not the data

Before opening Power BI Desktop, write down:

1. **Who** reads this page? (Regional managers, weekly, on a laptop, five minutes.)
2. **What decision** does it support? (Where to focus sales effort next week.)
3. **What questions**, in priority order? (Are we on target? Where are we behind? Why?)
4. **What would they do** if a number is red? (Call the store manager — so the page needs store
   names.)

If you can't answer (2), the page will become a data dump.

## A layout that follows the questions

Readers of left-to-right languages scan a page roughly top-left first. Put the answer to question 1
there.

```text
+--------------------------------------------------------------+
| Title as a finding: "Q1 sales 17% below target; Camping lags" |
| [Sales YTD] [Target YTD] [Achievement YTD %]  [Data as of ...]|
|--------------------------------------------------------------|
| Achievement by month (bars vs target line)  | Achievement by  |
|                                             | category (bars) |
|--------------------------------------------------------------|
| Detail table: category x month, conditional colour, drill →  |
+--------------------------------------------------------------+
```

- **Level 1: headline** — 3–4 KPIs with comparison (target or prior period). A KPI without a
  comparison is a number, not information.
- **Level 2: where** — breakdowns that locate the problem.
- **Level 3: detail** — a table or drill-through for people who need rows.

## Choosing honest charts

| Principle | In practice |
|-----------|-------------|
| Bars start at zero | Power BI's column/bar axes default to zero; don't override the start for bars. Lines may use a non-zero axis if labelled. |
| Compare against something | Target lines, prior-year series, reference lines (Analytics pane → Constant line / Average line). |
| One idea per chart | If a chart needs a paragraph to explain, split it. |
| Sort by value, not alphabet | Unless the category has a natural order (months, sizes). |
| Colour means something | Grey for context, one accent colour for the thing that matters; red/green only for good/bad and paired with icons or labels. |
| Label directly | Data labels at line ends beat a legend the reader must match by colour. |
| Avoid dual axes with unrelated scales | They invite false correlations. |

## Worked example: redesigning a page

**Before** (a typical first draft): nine visuals — a pie of sales by category, a gauge for
achievement, a donut by region, a treemap by product, three cards without comparisons, a map, and a
table with twelve columns. Title: "Sales Dashboard."

Applying the method with the Level 2 project numbers (Q1 2025: sales 1,215 vs target 1,460;
achievement 83.2%; Camping 75.9%, Apparel 91.4%, Accessories 96.2%; January 97.7%, February 92.0%,
March 63.2%):

1. **Title as a finding**: "Q1 sales 17% below target — March and Camping drive the gap." Check the
   claim: 1 − 0.832 = 0.168 ≈ 17% ✓. March's gap is 530 − 335 = 195 of the total 245 gap ✓. Camping's
   gap is 850 − 645 = 205 ✓. (Gaps by category sum to 205 + 30 + 10 = 245 ✓.)
2. **KPI row**: Sales YTD 1,215 · Target YTD 1,460 · Achievement 83.2% · Data as of date.
3. **Where (by month)**: clustered columns Sales vs Target by month, with the achievement % as data
   labels. March stands out without any colour tricks.
4. **Where (by category)**: horizontal bars of *variance* (Sales − Target): Camping −205, Apparel −30,
   Accessories −10, sorted most negative first, all in one accent colour.
5. **Detail**: the category × month matrix with conditional colour (lesson 08 of Level 3) and
   drill-through to Category Details.
6. **Removed**: pie, donut, treemap, gauge, map (there are three stores — a map adds nothing).

The result has five visuals instead of nine, and the reader learns the main point from the title
alone.

## Annotation and narrative

- **Subtitles** for method notes: "Targets are company-wide and not split by region."
- **Dynamic titles** with measures: `"Achievement: " & FORMAT([Achievement YTD %], "0.0%")`.
- **Tooltips** for secondary detail instead of more visuals.
- A small **"How to read this page"** info button (bookmark-driven overlay) for new audiences.
- Avoid narrative text that will go stale — tie text to measures or remove it.

## How It Actually Works

Why do these guidelines work? Human visual perception decodes some encodings more accurately than
others. Research in graphical perception (notably Cleveland and McGill's experiments in the 1980s)
found that people judge **position along a common scale** most accurately, then length, then angle
and area, with colour saturation among the least accurate for quantities. A bar chart encodes value as
position on a shared axis; a pie encodes it as angle and area — so comparing 26% and 21% slices is
genuinely harder than comparing two bars.

**Pre-attentive attributes** — colour hue, size, orientation, position — are processed before
conscious attention. A single accent colour on a grey chart directs the eye in a fraction of a second;
ten colours compete and nothing stands out. That's the mechanism behind "grey for context, colour for
emphasis."

**Working memory** is small: a legend with six colours forces the reader to hold mappings in mind while
scanning, which is why direct labels and fewer series read faster. The layout hierarchy (headline → where
→ detail) matches how people search for information: confirm the overall state, then locate the anomaly,
then inspect it.

## Common mistakes

- Titles that describe the chart ("Sales by Month") instead of the finding.
- KPIs without a comparison.
- Filling every pixel; white space is how readers see groups.
- Designing on a large monitor for readers on laptops or phones (use the mobile layout for phone
  audiences: View → Mobile layout).

## Exercise

1. Write the reader / decision / questions / action list for a report you use.
2. Redesign one page from your Level 2 project using the layout above; count visuals before and after.
3. Write three candidate "finding" titles for the page and verify each claim against measured numbers
   before choosing one.
