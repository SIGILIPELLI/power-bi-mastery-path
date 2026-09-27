---
description: "Format Power BI reports that read well: canvas size, themes and JSON theme files, number formats, titles, alignment, spacing, and a simple layout grid."
---

# 08 · Report Formatting & Layout

A correct report that nobody can read is not finished. Good formatting in Power BI is
mostly about consistency and restraint: one theme, consistent number formats, aligned
visuals, and titles that say what the reader is looking at. This lesson covers the
mechanics; Level 4 covers design principles and storytelling.

## Canvas settings

Select an empty part of the canvas, then open **Format** (paint roller) → **Canvas
settings**:

- **Type** — 16:9 (default, 1280 × 720 px), 4:3, Letter, or Custom.
- **Vertical alignment** — Top or Middle.

Pick one size for the whole report. Most screens and Teams embeds suit 16:9. Also under
page formatting: **Canvas background** (the page) and **Wallpaper** (the area outside the
page).

## Themes

**View → Themes** offers built-in themes. A theme sets the data colour palette, fonts,
background colours and default formatting of every visual. You can also **Customize
current theme** or import a JSON theme file. A small theme file looks like this:

```json
{
  "name": "Trailhead",
  "dataColors": ["#2F6B4F", "#C97B4A", "#E0B457", "#5B7DB1", "#8C8C8C"],
  "background": "#FFFFFF",
  "foreground": "#252423",
  "tableAccent": "#2F6B4F",
  "textClasses": {
    "title":  { "fontFace": "Segoe UI Semibold", "fontSize": 14, "color": "#252423" },
    "label":  { "fontFace": "Segoe UI", "fontSize": 10, "color": "#605E5C" }
  }
}
```

Save it as `trailhead-theme.json`, then **View → Themes → Browse for themes**. Every
visual that hasn't been manually overridden picks up the new colours. That last clause
matters: manual formatting on a visual beats the theme, so format through the theme
first, and override visuals only when you must.

## Number formats — set them on the measure

Select a measure in the Data pane and use **Measure tools**:

- **Format**: Currency, Decimal number, Whole number, Percentage, or a custom format
  string such as `#,0.00` or `$#,0;($#,0);-`.
- **Decimal places** and the **thousands separator** toggle.

Formatting the *measure* means every visual that uses it inherits the format. For the
Level 1 measures:

| Measure | Format | Displays as |
|---------|--------|-------------|
| Total Revenue | Currency, 0 decimals | $1,425 |
| Total Units | Whole number | 21 |
| Avg Selling Price | Currency, 2 decimals | $67.86 |

The currency symbol follows the format you choose, not the data; pick the one your
audience expects. In visuals, **Display units** (Auto, Thousands, Millions) on each value
can override how numbers are abbreviated — "Auto" turns 1,425 into 1.43K on cards, which
readers often misread.

## Titles, labels and tooltips

- Replace auto titles like "Sum of Revenue by Region" with a statement or question:
  **"Revenue by region, Q1 2025"**.
- Turn on data labels only when exact values matter and there are few points.
- Remove axis titles when the visual title already says what the axis is.
- Tooltips are free detail: add `[Total Units]` to the **Tooltips** well of the region
  chart so hovering shows units too.

## Alignment and a layout grid

1. **View → Gridlines** and **Snap to grid** on.
2. Use a simple grid: 16:9 canvas, 16 px outer margin, 16 px gutters.
3. Put KPI cards in a row along the top, the main chart below-left (largest), supporting
   visuals to the right, slicers in a consistent place (left column or top bar).
4. Select several visuals and use **Format → Align** (Align top, Distribute horizontally)
   so edges line up exactly.
5. Use the **Selection** pane (View → Selection) to name visuals ("Card – Revenue") and
   set tab order — this matters for keyboard users (Level 4).

A reference layout for a 1280 × 720 page:

```text
+----------------------------------------------------------------+
| Title                                         [Region slicer]  |
| [Card Revenue] [Card Units] [Card Avg Price] [Card Orders]      |
| +------------------------------+ +---------------------------+ |
| | Revenue by month (line)      | | Revenue by region (bar)   | |
| |                              | +---------------------------+ |
| |                              | | Category table            | |
| +------------------------------+ +---------------------------+ |
+----------------------------------------------------------------+
```

## How It Actually Works

A report's formatting is stored as JSON inside the `.pbix` (in the report *layout*
definition, separate from the model). Each visual has a `config` with its type, its field
bindings (the query), and an `objects` section holding only the properties you changed
from the default. At render time Power BI resolves each property in order:

1. Property explicitly set on the visual → wins.
2. Otherwise the property from the theme's visual-specific styles (`visualStyles` in the
   theme JSON), if any.
3. Otherwise the theme's global values (`dataColors`, `textClasses`).
4. Otherwise the built-in default.

That cascade is why "reset to default" on a visual makes it adopt the theme, and why a
theme import seems to "not work" on visuals you already hand-formatted.

Number formatting works differently: the **format string is a property of the measure or
column in the model**, and the engine returns it with query results (you'll see
`FORMAT_STRING` handling if you inspect queries later). The visual applies it at render.
This is why the same measure looks the same in every report built on the model — and why
in Level 3 calculation groups can change a measure's format string dynamically.

## Common mistakes

- **Rainbow charts**: a different colour per bar with no meaning. Use one colour, and
  colour only to highlight.
- **Formatting each visual manually** instead of the theme, then fighting inconsistency.
- **Auto display units** on cards showing 1.43K when readers expect exact dollars.
- **Misaligned visuals** a few pixels apart; readers notice even if they can't say why.

## Exercise

1. Create and import the theme JSON above; change one data colour and confirm charts
   update.
2. Format the three measures as in the table, then build the reference layout with four
   cards (the fourth: `Orders = COUNTROWS(Sales)`, expected 8).
3. Rename every visual in the Selection pane and set a sensible tab order.
