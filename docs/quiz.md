---
title: "Final Quiz"
description: "Test what you have learned across all four levels of Power BI Mastery Path with a 19-question final quiz."
---

# Final Quiz

19 questions drawn from all four levels of this course. You need **70% (14 out of 19)** to pass —
you can retake it as many times as you like. Your best result is saved in this browser.

<div id="mp-quiz-root"></div>

<script type="application/json" id="mp-quiz-data">
[
  {
    "question": "A sales table has Units and Price columns and no Amount column. Which measure gives revenue?",
    "options": [
      "SUM(Units) * SUM(Price)",
      "SUMX(Sales, Sales[Units] * Sales[Price])",
      "COUNT(Units)",
      "AVERAGE(Price)"
    ],
    "correctIndex": 1,
    "explanation": "Multiply per row first, then sum."
  },
  {
    "question": "Rows (units, price): (2,10),(3,5). What do SUMX(units*price) and SUM(units)*SUM(price) give?\n```python\nprint(2*10+3*5, (2+3)*(10+5))\n```\n",
    "options": [
      "35 35",
      "75 35",
      "35 75",
      "75 75"
    ],
    "correctIndex": 2,
    "explanation": "Sum of products is not the product of sums. Verified by running it."
  },
  {
    "question": "Slicer Category=Camping is applied and a measure uses CALCULATE([Sales], ALL(Product)). What does it return?",
    "options": [
      "An error",
      "Camping sales",
      "Total sales for all products (the Category filter is removed)",
      "Zero"
    ],
    "correctIndex": 2,
    "explanation": "Used for denominators."
  },
  {
    "question": "Why does a flat table hide a product with no sales while a star schema doesn't?",
    "options": [
      "The dimension table still has the product row even without fact rows",
      "Stars sort",
      "Flat is bigger",
      "Names"
    ],
    "correctIndex": 0,
    "explanation": "Dimensions list everything."
  },
  {
    "question": "A measure returns an overall rate by averaging a per-row percentage column. What's wrong?",
    "options": [
      "Rounding",
      "Nothing",
      "Types",
      "It ignores weights; compute DIVIDE(SUM(numerator), SUM(denominator))"
    ],
    "correctIndex": 3,
    "explanation": "Ratio of sums."
  },
  {
    "question": "TOTALYTD returns blanks for some dates. What do you check first?",
    "options": [
      "Gateway",
      "That a contiguous, marked date table exists and is related to the fact",
      "Colors",
      "Slicers"
    ],
    "correctIndex": 1,
    "explanation": "Time intelligence prerequisite."
  },
  {
    "question": "Sales this year 150 versus 120 last year. Which expression gives growth?",
    "options": [
      "SUM",
      "[Sales LY] - [Sales]",
      "[Sales] / 2",
      "DIVIDE([Sales] - [Sales LY], [Sales LY])"
    ],
    "correctIndex": 3,
    "explanation": "(150-120)/120 = 25%."
  },
  {
    "question": "A user can see data for all regions in the Service, but the Desktop test with role 'West' worked. What's missing?",
    "options": [
      "Colors",
      "Rebuild",
      "Users must be assigned to the role in the Service (and testing needs the Viewer role)",
      "Refresh"
    ],
    "correctIndex": 2,
    "explanation": "Assignments are separate."
  },
  {
    "question": "A scheduled refresh from an on-premises SQL Server fails after publishing. What is needed?",
    "options": [
      "Desktop",
      "A data gateway with valid credentials",
      "More RAM",
      "Slicers"
    ],
    "correctIndex": 1,
    "explanation": "On-prem access."
  },
  {
    "question": "Two relationships connect Sales to Date (OrderDate, ShipDate). How use the inactive one?",
    "options": [
      "ALL",
      "VALUES",
      "CALCULATE([Sales], USERELATIONSHIP(Sales[ShipDate], 'Date'[Date]))",
      "FILTER"
    ],
    "correctIndex": 2,
    "explanation": "Role-playing dimension."
  },
  {
    "question": "Which refresh strategy suits a 200-million-row fact table that only changes recently?",
    "options": [
      "Incremental refresh with RangeStart/RangeEnd and folded date filters",
      "DirectQuery only",
      "Manual",
      "Full refresh"
    ],
    "correctIndex": 0,
    "explanation": "Partitions."
  },
  {
    "question": "A report is slow. What is the first diagnostic step?",
    "options": [
      "Guess",
      "Performance Analyzer to see which visual/query is slow, then DAX Studio if needed",
      "Delete visuals",
      "Add slicers"
    ],
    "correctIndex": 1,
    "explanation": "Measure."
  },
  {
    "question": "Which change typically shrinks a model the most?",
    "options": [
      "Colors",
      "Add visuals",
      "Removing unused columns and reducing cardinality (e.g. split datetime into date and time)",
      "Rename"
    ],
    "correctIndex": 2,
    "explanation": "VertiPaq compression."
  },
  {
    "question": "Many measures need YTD, YoY and MTD variants. What reduces duplication?",
    "options": [
      "More tables",
      "Copy/paste",
      "Calculation groups",
      "Bookmarks"
    ],
    "correctIndex": 2,
    "explanation": "Reusable time calculations."
  },
  {
    "question": "Which approach keeps content consistent for hundreds of reports?",
    "options": [
      "Flat files",
      "Excel",
      "Copies",
      "Shared certified semantic models with thin reports"
    ],
    "correctIndex": 3,
    "explanation": "One version of truth."
  },
  {
    "question": "What lets external customers view embedded reports without Power BI licenses?",
    "options": [
      "PDF",
      "Share link",
      "Embed for your customers (app owns data) with an embed token",
      "Excel"
    ],
    "correctIndex": 2,
    "explanation": "Capacity required."
  },
  {
    "question": "Why store a Power BI project as PBIP in Git?",
    "options": [
      "Law",
      "Text-based files give reviewable diffs and branching",
      "Faster",
      "Smaller"
    ],
    "correctIndex": 1,
    "explanation": "CI/CD."
  },
  {
    "question": "Which practice most improves accessibility?",
    "options": [
      "Alt text, logical tab order, sufficient contrast and not relying on color alone",
      "Colors",
      "Animation",
      "3D"
    ],
    "correctIndex": 0,
    "explanation": "WCAG."
  },
  {
    "question": "How do you confirm a model change didn't break numbers?",
    "options": [
      "Compare measure outputs before and after against known values (regression tests)",
      "Eyeball",
      "Trust",
      "Skip"
    ],
    "correctIndex": 0,
    "explanation": "Automate in CI."
  }
]
</script>

## Coding challenges

Try these on your own computer, then compare your output. Nothing you write is sent anywhere.

### A filter-context engine: CALCULATE, REMOVEFILTERS and DIVIDE

Power BI Desktop isn't available here, so model its core idea in Python: join facts to dimensions, filter rows by a context dictionary, then implement CALCULATE (override and remove filters), a safe DIVIDE and a percent-of-total measure. Also list products with no sales, which a star schema can answer but a flat table cannot.

**Starter code**

```python
# A tiny model of Power BI's filter context: dimensions filter a fact table through relationships.
FACT = [  # (date, product_key, store_key, units, amount)
    ("2026-01-05", 1, 1, 2, 200), ("2026-01-20", 2, 1, 1, 150), ("2026-02-03", 1, 2, 4, 400),
    ("2026-02-14", 3, 2, 2, 90),  ("2026-03-02", 2, 1, 3, 450), ("2026-03-15", 1, 2, 1, 100),
]
DIM_PRODUCT = {1: {"Product": "Tent", "Category": "Camping"}, 2: {"Product": "Boots", "Category": "Hiking"},
               3: {"Product": "Stove", "Category": "Camping"}, 4: {"Product": "Lamp", "Category": "Camping"}}
DIM_STORE = {1: {"Store": "Austin", "Region": "South"}, 2: {"Store": "Denver", "Region": "West"}}

def rows_in_context(filters):
    """filters = {"Category": "Camping", "Region": "West", "Month": "2026-02"}; no filters means all rows.
    Join each fact row to its product and store attributes (plus Month = first 7 chars of the date) and keep
    the rows where every filter column equals its value."""
    return []

def total_sales(filters):                      # SUM(Sales[Amount]) in the given context
    return 0

def divide(n, d, alt=None):                    # DIVIDE(n, d) returns BLANK (None) when d is 0
    return alt if not d else n / d

def calculate(measure, context, add=None, remove=None):
    """CALCULATE: copy the context, REMOVEFILTERS on the named columns, then set/override filters from `add`."""
    ctx = dict(context)
    for k in (remove or []): ctx.pop(k, None)
    ctx.update(add or {})
    return measure(ctx)

def pct_of_region(context):                    # DIVIDE([Sales], CALCULATE([Sales], REMOVEFILTERS(Product)))
    return divide(total_sales(context), calculate(total_sales, context, remove=["Category", "Product"]))

def products_without_sales():
    # product names that have no fact rows, sorted
    return []

print("All sales:", total_sales({}))
print("Camping:", total_sales({"Category": "Camping"}))
print("Camping, West:", total_sales({"Category": "Camping", "Region": "West"}))
print("Camping vs all categories (South):", total_sales({"Category": "Camping", "Region": "South"}), "/", calculate(total_sales, {"Category": "Camping", "Region": "South"}, remove=["Category"]))
print("Share of South sales that is Camping:", round(pct_of_region({"Category": "Camping", "Region": "South"}) * 100, 1), "%")
print("CALCULATE override (Hiking in a Camping context):", calculate(total_sales, {"Category": "Camping"}, add={"Category": "Hiking"}))
print("Divide by zero ->", divide(5, 0), "| empty context ->", total_sales({"Category": "Lamps"}))
print("Products with no sales:", products_without_sales())
```

**Expected output**

```text
All sales: 1390
Camping: 790
Camping, West: 590
Camping vs all categories (South): 200 / 800
Share of South sales that is Camping: 25.0 %
CALCULATE override (Hiking in a Camping context): 600
Divide by zero -> None | empty context -> 0
Products with no sales: ['Lamp']
```

<details><summary>Show solution and explanation</summary>

<pre><code># A tiny model of Power BI&#x27;s filter context: dimensions filter a fact table through relationships.
FACT = [  # (date, product_key, store_key, units, amount)
    (&quot;2026-01-05&quot;, 1, 1, 2, 200), (&quot;2026-01-20&quot;, 2, 1, 1, 150), (&quot;2026-02-03&quot;, 1, 2, 4, 400),
    (&quot;2026-02-14&quot;, 3, 2, 2, 90),  (&quot;2026-03-02&quot;, 2, 1, 3, 450), (&quot;2026-03-15&quot;, 1, 2, 1, 100),
]
DIM_PRODUCT = {1: {&quot;Product&quot;: &quot;Tent&quot;, &quot;Category&quot;: &quot;Camping&quot;}, 2: {&quot;Product&quot;: &quot;Boots&quot;, &quot;Category&quot;: &quot;Hiking&quot;},
               3: {&quot;Product&quot;: &quot;Stove&quot;, &quot;Category&quot;: &quot;Camping&quot;}, 4: {&quot;Product&quot;: &quot;Lamp&quot;, &quot;Category&quot;: &quot;Camping&quot;}}
DIM_STORE = {1: {&quot;Store&quot;: &quot;Austin&quot;, &quot;Region&quot;: &quot;South&quot;}, 2: {&quot;Store&quot;: &quot;Denver&quot;, &quot;Region&quot;: &quot;West&quot;}}

def rows_in_context(filters):
    &quot;&quot;&quot;filters = {&quot;Category&quot;: &quot;Camping&quot;, &quot;Region&quot;: &quot;West&quot;, &quot;Month&quot;: &quot;2026-02&quot;}; no filters means all rows.&quot;&quot;&quot;
    out = []
    for d, p, s, u, a in FACT:
        attrs = {**DIM_PRODUCT[p], **DIM_STORE[s], &quot;Month&quot;: d[:7]}
        if all(attrs.get(k) == v for k, v in filters.items()):
            out.append((d, p, s, u, a))
    return out

def total_sales(filters):                      # SUM(Sales[Amount])
    return sum(r[4] for r in rows_in_context(filters))

def divide(n, d, alt=None):                    # DIVIDE(n, d) returns BLANK (None) when d is 0
    return alt if not d else n / d

def calculate(measure, context, add=None, remove=None):
    &quot;&quot;&quot;CALCULATE: copy the context, REMOVEFILTERS on the named columns, then set/override filters from `add`.&quot;&quot;&quot;
    ctx = dict(context)
    for k in (remove or []): ctx.pop(k, None)
    ctx.update(add or {})
    return measure(ctx)

def pct_of_region(context):                    # DIVIDE([Sales], CALCULATE([Sales], REMOVEFILTERS(Product)))
    return divide(total_sales(context), calculate(total_sales, context, remove=[&quot;Category&quot;, &quot;Product&quot;]))

def products_without_sales():                  # a star schema keeps dimension rows with no facts
    sold = {r[1] for r in FACT}
    return sorted(v[&quot;Product&quot;] for k, v in DIM_PRODUCT.items() if k not in sold)

print(&quot;All sales:&quot;, total_sales({}))
print(&quot;Camping:&quot;, total_sales({&quot;Category&quot;: &quot;Camping&quot;}))
print(&quot;Camping, West:&quot;, total_sales({&quot;Category&quot;: &quot;Camping&quot;, &quot;Region&quot;: &quot;West&quot;}))
print(&quot;Camping vs all categories (South):&quot;, total_sales({&quot;Category&quot;: &quot;Camping&quot;, &quot;Region&quot;: &quot;South&quot;}), &quot;/&quot;, calculate(total_sales, {&quot;Category&quot;: &quot;Camping&quot;, &quot;Region&quot;: &quot;South&quot;}, remove=[&quot;Category&quot;]))
print(&quot;Share of South sales that is Camping:&quot;, round(pct_of_region({&quot;Category&quot;: &quot;Camping&quot;, &quot;Region&quot;: &quot;South&quot;}) * 100, 1), &quot;%&quot;)
print(&quot;CALCULATE override (Hiking in a Camping context):&quot;, calculate(total_sales, {&quot;Category&quot;: &quot;Camping&quot;}, add={&quot;Category&quot;: &quot;Hiking&quot;}))
print(&quot;Divide by zero -&gt;&quot;, divide(5, 0), &quot;| empty context -&gt;&quot;, total_sales({&quot;Category&quot;: &quot;Lamps&quot;}))
print(&quot;Products with no sales:&quot;, products_without_sales())
</code></pre>

<p>A filter context is just the set of dimension filters applied to the fact table; CALCULATE copies the context, removes or overrides filters, then evaluates the measure. This is a Python model of the behaviour (not DAX itself), verified by running it.</p>
</details>

### Date tables, time intelligence and incremental refresh

Model a contiguous date table, TOTALYTD, SAMEPERIODLASTYEAR and a safe growth calculation, then work out which monthly partitions an incremental refresh would reload.

**Starter code**

```python
from datetime import date, timedelta

def date_table(start, end):
    # every date from start to end inclusive (contiguous - required for time intelligence)
    return []

SALES = {date(2025, 1, 15): 100, date(2025, 2, 10): 120, date(2025, 3, 5): 90,
         date(2026, 1, 12): 130, date(2026, 2, 8): 150, date(2026, 3, 20): 60}

def sales_on(days):                            # SUM over the dates in the filter context
    return sum(SALES.get(d, 0) for d in days)

def ytd(table, as_of):
    # TOTALYTD: sales from Jan 1 of as_of's year up to as_of
    return 0

def same_period_last_year(table, as_of):
    # the same window shifted back one year
    return 0

def growth(now, before):
    # DIVIDE(now - before, before) as a percentage rounded to 1 decimal; None when before is 0
    return None

def month_partitions(start, end):
    # list of "YYYY-MM" strings from start's month to end's month inclusive
    return []

def partitions_to_refresh(all_parts, today, refresh_months):
    # incremental refresh: only the most recent `refresh_months` months up to today's month
    return []

table = date_table(date(2025, 1, 1), date(2026, 12, 31))
print("date table days:", len(table), "| contiguous:", all((b - a).days == 1 for a, b in zip(table, table[1:])))
as_of = date(2026, 2, 28)
now, before = ytd(table, as_of), same_period_last_year(table, as_of)
print("YTD:", now, "| same period last year:", before, "| growth:", growth(now, before), "%")
print("growth with no prior year:", growth(50, 0))
parts = month_partitions(date(2025, 1, 1), date(2026, 3, 1))
print("partitions:", len(parts), parts[0], "..", parts[-1])
print("refresh last 2 months:", partitions_to_refresh(parts, date(2026, 3, 10), 2))
```

**Expected output**

```text
date table days: 730 | contiguous: True
YTD: 280 | same period last year: 220 | growth: 27.3 %
growth with no prior year: None
partitions: 15 2025-01 .. 2026-03
refresh last 2 months: ['2026-02', '2026-03']
```

<details><summary>Show solution and explanation</summary>

<pre><code>from datetime import date, timedelta

# 1. A contiguous date table (required for time intelligence).
def date_table(start, end):
    d, out = start, []
    while d &lt;= end:
        out.append(d); d += timedelta(days=1)
    return out

SALES = {date(2025, 1, 15): 100, date(2025, 2, 10): 120, date(2025, 3, 5): 90,
         date(2026, 1, 12): 130, date(2026, 2, 8): 150, date(2026, 3, 20): 60}

def sales_on(days):                            # SUM over the dates in the filter context
    return sum(SALES.get(d, 0) for d in days)

def ytd(table, as_of):                         # TOTALYTD: Jan 1 .. as_of
    return sales_on(d for d in table if date(as_of.year, 1, 1) &lt;= d &lt;= as_of)

def same_period_last_year(table, as_of):       # SAMEPERIODLASTYEAR: shift the YTD window back one year
    start, end = date(as_of.year - 1, 1, 1), as_of.replace(year=as_of.year - 1)
    return sales_on(d for d in table if start &lt;= d &lt;= end)

def growth(now, before):                       # DIVIDE(now - before, before)
    return None if not before else round((now - before) / before * 100, 1)

# 2. Incremental refresh: refresh only partitions inside [RangeStart, RangeEnd).
def month_partitions(start, end):
    parts, y, m = [], start.year, start.month
    while (y, m) &lt;= (end.year, end.month):
        parts.append(f&quot;{y}-{m:02d}&quot;); m += 1
        if m == 13: y, m = y + 1, 1
    return parts

def partitions_to_refresh(all_parts, today, refresh_months):
    y, m = today.year, today.month
    keep = []
    for _ in range(refresh_months):
        keep.append(f&quot;{y}-{m:02d}&quot;); m -= 1
        if m == 0: y, m = y - 1, 12
    return [p for p in all_parts if p in keep]

table = date_table(date(2025, 1, 1), date(2026, 12, 31))
print(&quot;date table days:&quot;, len(table), &quot;| contiguous:&quot;, all((b - a).days == 1 for a, b in zip(table, table[1:])))
as_of = date(2026, 2, 28)
now, before = ytd(table, as_of), same_period_last_year(table, as_of)
print(&quot;YTD:&quot;, now, &quot;| same period last year:&quot;, before, &quot;| growth:&quot;, growth(now, before), &quot;%&quot;)
print(&quot;growth with no prior year:&quot;, growth(50, 0))
parts = month_partitions(date(2025, 1, 1), date(2026, 3, 1))
print(&quot;partitions:&quot;, len(parts), parts[0], &quot;..&quot;, parts[-1])
print(&quot;refresh last 2 months:&quot;, partitions_to_refresh(parts, date(2026, 3, 10), 2))
</code></pre>

<p>Time intelligence needs a contiguous date table; YTD and last-year windows are date ranges over it; growth uses a safe divide; incremental refresh reloads only recent partitions. This is a Python model of the behaviour (not DAX or the Service), verified by running it.</p>
</details>
