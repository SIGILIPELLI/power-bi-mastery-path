---
description: "Filter Power BI reports with visual, page and report filters, slicers, cross-filtering and cross-highlighting, and edit interactions."
---

# 06 · Filters & Slicers

Filtering is where Power BI stops being a chart tool and becomes an analysis tool. There
are four ways to filter, and they stack: a number on the screen is the result of *all*
filters that apply to that visual at that moment.

## The four layers

| Layer | Set in | Applies to | Visible to readers? |
|-------|--------|-----------|---------------------|
| Visual-level filter | Filters pane → "Filters on this visual" | One visual | Optional (can be hidden/locked) |
| Page-level filter | Filters pane → "Filters on this page" | All visuals on the page | Optional |
| Report-level filter | Filters pane → "Filters on all pages" | Every page | Optional |
| Slicer | A visual on the canvas | By default, every visual on the same page | Always |

On top of these, **clicking** a data point in one visual *cross-filters* or
*cross-highlights* the others. And in the service, readers can apply their own
filters in the pane if you let them; their changes can persist for them.

## Step by step: a region slicer and a category filter

Using the page from lesson 05 (column chart by region, card, line chart, table):

1. Click empty canvas, choose **Slicer** from the gallery, drag `Region` into **Field**.
2. In **Format → Slicer settings → Options**, choose a style: *Vertical list*, *Tile*
   (buttons), or *Dropdown*. Turn on **Select all** under Selection if you like.
3. Select **North** in the slicer. Expected:
    - Card: **445**
    - Table: Camping 2 units / 240, Accessories 5 / 125, Apparel 1 / 80, total 8 / 445
    - Line chart: Jan 365, Feb blank, Mar 80
4. Clear the slicer. Now open the **Filters** pane (View → Filters if hidden) and drag
   `Category` to **Filters on this page**. Choose **Basic filtering**, tick *Camping*.
   Card: **720**.
5. Combine: page filter Camping + slicer North. Card: **240** (only order 1001).

Hand check for step 3: North orders are 1001 (240, Jan), 1003 (125, Jan), 1007 (80, Mar).
January = 240 + 125 = 365. February has no North orders, so the line chart has no point
for it — it does not show zero.

## Filter types in the pane

- **Basic** — tick values.
- **Advanced** — conditions like *contains*, *is greater than*, *is blank*, combined with
  And/Or.
- **Top N** — keep the top/bottom N items of a field **by a value**, e.g. top 2
  products by Revenue.
- **Relative date** — "in the last 3 months," evaluated at query time. Useful for rolling
  dashboards; remember that "today" is determined when the query runs, and in the
  service it is based on UTC unless otherwise configured.

## Controlling interactions

Click a column (South) in the column chart. By default other visuals **cross-highlight**
(bar charts show South's share darkened against the full bar) or **cross-filter** (cards
and tables show only South's values).

To change this: select the column chart, then **Format → Edit interactions** in the ribbon.
Small icons appear above every other visual:

- **Filter** (funnel) — filter the target visual.
- **Highlight** (chart icon) — show the selected portion against the total.
- **None** (circle with a line) — ignore the selection.

A common design: the headline card should *not* react to clicks in a detail chart, so set
it to **None**.

**Sync slicers** (View → Sync slicers) lets one slicer drive several pages, and optionally
appear only on some of them.

## How It Actually Works

Every filter — slicer, pane filter, cross-filter click — becomes part of the **filter
context** of the DAX query that each visual sends. With North selected in the slicer, the
card's query is roughly:

```dax
DEFINE
    VAR __Filter = TREATAS({"North"}, 'Sales'[Region])
EVALUATE
SUMMARIZECOLUMNS(
    __Filter,
    "SumRevenue", CALCULATE(SUM('Sales'[Revenue]))
)
```

`TREATAS` builds a small table containing `"North"` and applies it as a filter on the
`Region` column. `SUMMARIZECOLUMNS` evaluates the expression with that filter in place.
Stack a page filter on Category and a second filter argument is added; the engine combines
them with a logical **AND**: rows must satisfy both.

Filters apply to **columns**, and through relationships a filter on one table reaches
related tables (the subject of Level 2). This is also why a slicer on a column filters
*every* visual that uses the same model — they all receive the same filter unless you have
turned the interaction off, in which case Power BI simply omits it from that visual's
query.

Cross-highlighting is different: it requires *two* queries — one for the full values and
one with the selection applied — so the visual can draw the highlighted portion over the
total.

## Common mistakes

- **Hidden filters nobody knows about.** A visual-level filter left over from testing
  makes one chart disagree with the rest. Check the Filters pane when numbers differ.
- **Too many slicers.** Each slicer is itself a query; ten slicers on a page slow it down
  and overwhelm readers. Consider the Filters pane for secondary filters.
- **Expecting blanks to show as zero.** A month with no data is absent, not 0. Decide
  whether that's what you want (Level 2 shows how to force zeros where it's meaningful).
- **Relative date filters and time zones** — "today" can differ between your laptop and
  the service.

## Exercise

1. Build the slicer and filter combinations above and confirm 445, 720 and 240.
2. Add a Top N visual-level filter to the table: top 2 **Category** by **Revenue**. Which
   categories remain, and what does the table total show? (Answer: Camping and Apparel;
   the total becomes 1,200 because the filter applies to the total row too.)
3. Set the card to ignore clicks on the column chart using Edit interactions. Then write
   one sentence explaining when a headline number *should* respond to clicks.
