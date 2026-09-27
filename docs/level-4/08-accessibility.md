---
description: "Build accessible Power BI reports: keyboard navigation and tab order, alt text, screen readers, contrast, colour-blind-safe palettes, and testing."
---

# 08 · Accessible Reports

Some of your readers use a screen reader, navigate with a keyboard, have low vision, or can't
distinguish red from green. In many organizations accessibility is also a legal or policy
requirement. Power BI has solid accessibility support — but most of it only works if the report
author sets it up. This lesson is a practical checklist, with the mechanism behind each item.

## Keyboard navigation

Readers can move through a report without a mouse:

- **Ctrl+F6** moves focus between the report canvas and the rest of the page (panes, header).
- **Tab** moves between visuals in the **tab order**.
- **Ctrl+Right arrow** (in many visuals) enters a visual to move between data points; **Esc** leaves it.
- **Alt+Shift+F10** opens the visual's context menu; **Alt+Shift+F11** opens the "show as table"
  view, which presents the visual's data as an accessible table.
- **?** or **Shift+?** in the service shows the keyboard shortcut list.

(Keyboard shortcuts vary by platform and release; the in-product shortcut list is authoritative.)

### Setting tab order

**View → Selection → Tab order**. Order it the way a sighted reader would read the page — title,
KPI cards left to right, main chart, supporting charts, table. Hide purely decorative items (shapes,
background images) from tab order with the eye-slash in the Tab order list, so keyboard users don't
land on them.

## Alt text and titles

Every visual has **Format → General → Alt text** (the location has moved between releases). Screen
readers announce the visual type, title and alt text. Static alt text goes stale; use **conditional
formatting (fx)** to drive it from a measure:

```dax
Alt Text Sales by Month =
"Clustered column chart of sales versus target for "
    & COUNTROWS ( VALUES ( 'Date'[Year Month] ) ) & " months. Total sales "
    & FORMAT ( [Sales Amount], "#,0" ) & " against target "
    & FORMAT ( [Target Amount], "#,0" ) & ", achievement "
    & FORMAT ( [Achievement %], "0.0%" ) & "."
```

For the Level 2 project's Q1 page this produces: "Clustered column chart of sales versus target for 3
months. Total sales 1,215 against target 1,460, achievement 83.2%." Alt text has a length limit; keep it
to the key message, not every value.

Give every visual a **meaningful title** too — screen readers use it — and don't rely on a title that's
only an image.

## Colour and contrast

- **Contrast**: WCAG 2.x asks for a contrast ratio of at least **4.5:1** for normal text and **3:1** for
  large text and for graphical objects needed to understand content.
- **Never colour alone**: pair colour with icons, labels, patterns or position. In the Level 3
  conditional-formatting example, add an icon set or show the percentage in the cell.
- **Colour-blind-safe palettes**: avoid red/green as the only distinction; blue/orange is a common safer
  pairing. Power BI includes an **Accessible** theme family (View → Themes) designed with contrast in
  mind.

### Worked example: checking a contrast ratio by hand

The WCAG contrast ratio is `(L1 + 0.05) / (L2 + 0.05)`, where L is relative luminance (0 for black,
1 for white). For black text (#000000, L = 0) on white (#FFFFFF, L = 1): (1 + 0.05) / (0 + 0.05) =
**21:1**, the maximum.

For the grey `#767676` on white: each channel is 118/255 = 0.4627; linearized with the sRGB formula
((0.4627 + 0.055) / 1.055)^2.4 ≈ 0.1812, so L ≈ 0.1812 (equal channels). Ratio = 1.05 / (0.1812 +
0.05) ≈ **4.54:1** — just passes 4.5:1 for normal text. A lighter grey such as `#999999` (L ≈ 0.318)
gives 1.05 / 0.368 ≈ **2.85:1**, which fails for text. Axis labels in pale grey are the most common
contrast failure in reports.

## Other checklist items

- **Sort order** of the table behind "show as table" should be meaningful.
- **Markers** on line charts (different shapes per series) help when colour can't.
- **Avoid auto-playing animations** and excessive motion.
- **Text size**: at least 12 pt for body text on the canvas; titles larger.
- **Data labels vs tooltips**: information only in tooltips is hard to reach by keyboard — put essential
  numbers on the page.
- **Mobile layout** has its own order and sizes; check it separately.

## Testing

1. Unplug the mouse (figuratively) and complete the page's main task with the keyboard only.
2. Turn on a screen reader — Narrator (Windows) or NVDA (free) — and listen to the page with your eyes
   closed. Do titles and alt text explain what each visual shows?
3. Check contrast with a contrast checker tool for text colours in your theme.
4. Use a colour-blindness simulator (several browser extensions and design tools offer one) on a
   screenshot of the page.
5. Use **Show as table** on each visual and check the result makes sense.

## How It Actually Works

The Power BI report canvas is rendered in the browser as HTML and SVG, with visuals exposing
information through **ARIA** (Accessible Rich Internet Applications) attributes — roles, labels and
live regions — that assistive technologies read. When a visual receives focus, its accessible name is
composed from the visual type, its title, and its alt text; when you move into a visual's data points,
the visual exposes each point's category and value as it gets focus.

"Show as table" works because every visual is backed by a query result (lesson 05 of Level 1): the
report host can present that result as a plain HTML table without any charting, which assistive
technology handles well. That's also why the visual's underlying query should be meaningful on its own
— sorted, with clear column names from well-named measures.

Dynamic alt text is evaluated like any measure in the visual's filter context, so it updates with
slicers, and it's recomputed whenever the visual is.

## Common mistakes

- Default tab order following creation order — often random.
- Decorative shapes that grab keyboard focus.
- Pale grey axis labels and light text on coloured backgrounds.
- Red/green conditional formatting with no icon or text cue.
- Alt text copied from the title, adding nothing.

## Exercise

1. Set tab order and hide decorative elements on your Level 2 project's overview page.
2. Add measure-driven alt text to two visuals and check it with Narrator or NVDA.
3. Compute by hand the contrast ratio of white text on your theme's primary data colour. (Convert each
   channel to 0–1, linearize with `((c + 0.055)/1.055)^2.4` for values above 0.04045, weight
   0.2126 R + 0.7152 G + 0.0722 B.) Does it pass 4.5:1 for data labels?
