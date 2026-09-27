---
description: "Add Power BI navigation: drill-down vs drill-through pages, report page tooltips, bookmarks, buttons, and the selection pane for show/hide patterns."
---

# 07 · Drill-through, Tooltips & Bookmarks

A single page crammed with every detail is hard to read. Power BI gives you several ways to
keep the main page simple and let readers go deeper *on demand*: drill-down within a
visual, drill-through to a detail page, rich tooltips, and bookmarks for saved views. Each
works by passing filter context from one place to another — once you see that, they stop
feeling like magic.

## Drill-down (within a visual)

Put a hierarchy on an axis — `Date[Year]`, `Date[Quarter]`, `Date[Month]`, or
`DimProduct[Category]` then `DimProduct[Product]`. The visual header gains arrows:

- **Drill down** (single arrow, then click a data point) — go into that one member.
- **Go to next level** (double arrow) — show the next level for all members.
- **Expand all down one level** (forked arrow) — show both levels together (e.g. "Camping
  Trail Tent").

Drill-down stays on the same visual. It's the right tool for exploring a hierarchy.

## Drill-through (to another page)

A **drill-through page** is a detail page that receives a filter from wherever the user
right-clicked.

### Step by step: a Product Details page

1. Add a page named **Product Details**.
2. Select the page canvas (no visual selected). In the **Visualizations** pane, find the
   **Drill through** section and drag `DimProduct[Product]` into it.
3. Keep **Cross-report** off and **Keep all filters** on (the default — explained below).
4. Build the page: a card with `[Total Amount]`, a card with `[Total Units]`, a table of
   `DimStore[Store]`, `Date[Year Month]`, `[Total Amount]`.
5. Power BI adds a **Back** button automatically at the top left (a button with the "Back"
   action). Ctrl+click it in Desktop to use it.
6. Go to the main page. Right-click the Trail Tent bar in any visual that shows Product →
   **Drill through → Product Details**.

Expected on the detail page for Trail Tent (Level 2 sample, 2025 rows only): Total Amount
**360**, Units **3**; rows Denver 2025-01 240 and Austin 2025-02 120.

**Keep all filters** means other filters on the source visual (for example, a Region slicer
set to West) also travel. With West selected, the Trail Tent drill-through shows only
Denver: 240, 2 units.

You can drill through on a **measure** too: put a measure in the drill-through well and
choose "Used as category" or "Summarized" to control when the option appears.

## Report page tooltips

Instead of a tooltip listing field values, show a whole mini page:

1. New page **TT Region**. In **Format → Page information**, turn on **Allow use as
   tooltip**. In **Canvas settings**, choose Type **Tooltip** (a small canvas).
2. Put a small line chart (Year Month × Total Amount) and a card on it.
3. In the page's **Tooltip** field well (Visualizations pane, when the page is selected),
   add `DimStore[Region]`.
4. On the main page, select the region bar chart → **Format → Properties → Tooltips**, set
   Type **Report page**, Page **TT Region** (or leave it Auto, which matches by the field).

Hover the West bar: the tooltip page renders filtered to West — January 240, February 190,
March 240.

## Bookmarks

A **bookmark** captures a page's state: filters and slicers, sort order, drill location,
visibility of visuals, and which page is shown. Open **View → Bookmarks** and
**View → Selection**.

### Step by step: a chart/table toggle

1. On the main page, place a column chart (Amount by Category) and a table with the same
   data exactly on top of each other.
2. In the Selection pane, hide the table (eye icon). Add a bookmark **Show Chart**.
3. Hide the chart, show the table. Add bookmark **Show Table**.
4. For both bookmarks, open **…** and **untick Data** — so switching views doesn't reset
   the user's slicer selections. Keep **Display** ticked (visibility) and **Current page**.
5. Insert two buttons (**Insert → Buttons → Blank**), labelled *Chart* and *Table*. Set each
   button's **Action → Type: Bookmark**, and the right bookmark.
6. Alternatively, group both bookmarks into a **bookmark navigator** (Insert → Buttons →
   Navigator → Bookmark navigator), which highlights the active one.

The **Data / Display / Current page** checkboxes on each bookmark are the part most people
miss, and they explain most "my bookmark reset my filters" complaints.

## How It Actually Works

Drill-through builds a filter from the **data point the user right-clicked**. Internally, the
clicked cell's coordinates (Product = "Trail Tent", and with *Keep all filters*, every
filter in effect on that visual — slicers, page filters, the visual's own filters) are
serialized as a filter set, the target page is opened, and those filters are applied to it
as page-level filters, visible in the target page's Filters pane. Every visual on the
target page then sends its DAX query with that filter context. Nothing is precomputed; the
detail page is just a page with filters injected.

Report page tooltips work the same way, with the hovered data point's filters injected into
the tooltip page's queries. That's why tooltip pages must stay light: each hover fires fresh
queries for every visual on the tooltip page.

A bookmark is stored as JSON describing the state of each visual (visible/hidden, sort,
drill level, selected data points) and, if **Data** is ticked, the full filter state of the
page. Applying it replays that state. It does not store query results. Personal bookmarks
in the service work the same way but are saved per user.

## Common mistakes

- **Drill-through field not on the source visual.** The right-click option only appears if the
  source visual has the drill-through field (or a field that maps to it via the model).
- **Keep all filters surprises** — users drilling from a filtered page see "missing" data.
  Tell them in a subtitle which filters came through, e.g. a card with
  `SELECTEDVALUE(DimStore[Region], "All regions")`.
- **Bookmarks capturing Data** when they should only toggle visibility.
- **Heavy tooltip pages** that make hovering slow.

## Exercise

1. Build the Product Details drill-through and verify Trail Tent: 360 / 3 units, and with
   West selected, 240 / 2 units.
2. Build the region tooltip page and check East: January 80, February 150, March blank.
3. Build the chart/table toggle with bookmarks that do **not** capture Data. Test that a
   Category slicer selection survives switching views.
