---
description: "Control DAX filters precisely: ALL vs REMOVEFILTERS vs ALLEXCEPT vs ALLSELECTED, and KEEPFILTERS for intersecting instead of replacing, all traced by hand."
---

# 03 · ALL, REMOVEFILTERS & KEEPFILTERS

`CALCULATE` changes filters, and a small set of functions decides *how*: remove filters from a
column or table, keep some and remove others, respect only what the user selected, or
**intersect** a new filter with the existing one instead of replacing it. This lesson traces
each on the Level 3 `Sales` table, with the `Products` dimension (A, B → North; C → Summit)
related to `Sales[Product]`.

Reference values: Revenue by product A 30, B 60, C 24 (total 114); by color Red 84, Blue 30;
by brand North 90, Summit 24.

## ALL and REMOVEFILTERS

As a `CALCULATE` modifier, `REMOVEFILTERS ( X )` and `ALL ( X )` do the same thing: remove
filters from column(s) or table X. `REMOVEFILTERS` exists because it says what it does;
`ALL` is also a **table function** that returns all rows/values ignoring filters.

```dax
Revenue All Products = CALCULATE ( [Revenue], REMOVEFILTERS ( Sales[Product] ) )
Revenue All Sales    = CALCULATE ( [Revenue], REMOVEFILTERS ( Sales ) )
```

Matrix with Color on rows and Product on columns — cell (Red, A):

- `[Revenue]` = 20
- `Revenue All Products` = Red across all products = 84 (Color filter kept)
- `Revenue All Sales` = 114 (every filter on the Sales table removed)

Removing filters from a column vs a table is the most important choice you make here.

### What about filters on the dimension?

Put `Products[Brand]` on rows instead of Sales columns. Now compare:

| Brand | Revenue | `REMOVEFILTERS ( Sales[Product] )` | `REMOVEFILTERS ( Sales )` |
|-------|--------:|-----------------------------------:|--------------------------:|
| North | 90 | 90 | 114 |
| Summit | 24 | 24 | 114 |

Removing the filter from the *column* `Sales[Product]` does nothing here, because the filter
isn't on that column — it's on `Products[Brand]` and reaches `Sales` through the relationship.
Removing filters from the *table* `Sales` does clear it, because a table reference in
`REMOVEFILTERS`/`ALL` means the **expanded table**: `Sales` plus every column reachable through
its many-to-one relationships, which includes `Products[Brand]`. (The reverse is not true:
`REMOVEFILTERS ( Products )` removes the Brand filter but would not remove a filter placed
directly on a `Sales` column such as Color.) To say exactly what you mean, name the dimension:
`REMOVEFILTERS ( Products )`.

## ALLEXCEPT

"Remove all filters on this table except these columns":

```dax
Revenue Same Color = CALCULATE ( [Revenue], ALLEXCEPT ( Sales, Sales[Color] ) )
```

In the (Red, A) cell: Product filter removed, Color kept → **84**. Useful for "share within
group." Caveat: like `REMOVEFILTERS ( Sales )`, `ALLEXCEPT ( Sales, … )` works on the expanded
table, so a `Products[Brand]` slicer is removed as well — often not what people intend. The
more explicit alternative
`CALCULATE ( [Revenue], REMOVEFILTERS ( Sales ), VALUES ( Sales[Color] ) )` removes everything
on `Sales` and then restores the colors *currently visible*. They differ when other filters
limit which colors are visible: in a table by Product with no color filter, for product B
(sold only in Red) the `VALUES` version returns Red's **84**, while `ALLEXCEPT` has no color
filter to keep and returns **114**.

## ALLSELECTED

`ALLSELECTED` removes filters coming from the **visual's own rows and columns** while keeping
filters from outside the visual (slicers, page filters). It's the tool for "percent of the
visible total":

```dax
% of Visible Total =
DIVIDE ( [Revenue], CALCULATE ( [Revenue], ALLSELECTED ( Sales[Product] ) ) )
```

With a slicer selecting products **A and B**, a table by Product:

| Product | Revenue | % of Visible Total |
|---------|--------:|-------------------:|
| A | 30 | 33.3% (30 ÷ 90) |
| B | 60 | 66.7% |
| Total | 90 | 100% |

With `REMOVEFILTERS ( Sales[Product] )` instead, the denominator would be 114 and the
percentages 26.3% and 52.6%, not adding to 100% of what's shown.

## KEEPFILTERS

By default a `CALCULATE` filter argument **replaces** the existing filter on that column.
`KEEPFILTERS` makes it **intersect** instead.

```dax
Red Revenue       = CALCULATE ( [Revenue], Sales[Color] = "Red" )
Red Revenue (KF)  = CALCULATE ( [Revenue], KEEPFILTERS ( Sales[Color] = "Red" ) )
```

Table with Color on rows:

| Color | Revenue | Red Revenue | Red Revenue (KF) |
|-------|--------:|------------:|-----------------:|
| Blue | 30 | 84 | *(blank)* |
| Red | 84 | 84 | 84 |
| Total | 114 | 84 | 84 |

On the Blue row, the replacing version throws away "Color = Blue" and shows Red's 84 — often
not what the reader expects. `KEEPFILTERS` intersects {Blue} with {Red} = empty → blank.

`KEEPFILTERS` also matters for iterators over filtered tables, e.g.
`SUMX ( KEEPFILTERS ( VALUES ( … ) ), … )`, and for multi-column filters where replacing would
drop a user's selection.

## Summary table

| Goal | Use |
|------|-----|
| Grand total ignoring a column's filter | `REMOVEFILTERS ( T[Col] )` |
| Grand total ignoring everything on a table | `REMOVEFILTERS ( T )` (and its dimensions if needed) |
| Keep only certain columns' filters | `ALLEXCEPT ( T, T[Col] )`, or REMOVEFILTERS + VALUES |
| Total of what the visual shows under current slicers | `ALLSELECTED ( T[Col] )` |
| Add a filter without overriding the user's | `KEEPFILTERS ( … )` |
| A list of all values (table function) | `ALL ( T[Col] )`, e.g. inside `RANKX` |

## How It Actually Works

Filter context is a set of filters, each on one or more columns. `CALCULATE` builds a new
context from it in this order: evaluate filter arguments in the *original* context, apply
context transition, apply modifiers (`REMOVEFILTERS`, `ALLSELECTED`, `USERELATIONSHIP`…),
then merge the filter arguments. During the merge, each argument is compared with existing
filters by **column**: without `KEEPFILTERS`, existing filters on the same columns are dropped
and the new one inserted; with `KEEPFILTERS`, both are kept and the engine intersects them.

`ALLSELECTED` is special: it relies on **shadow filter contexts**. When a visual is queried,
`SUMMARIZECOLUMNS` records the filters that existed *before* it started iterating the groups
(the slicer and page filters). `ALLSELECTED` restores those, removing only filters the visual's
own grouping introduced. That's why it behaves "visually" but can give surprising results when
used inside other iterators — the shadow context isn't always the one you picture.

Finally, remember **expanded tables**. Every table in DAX is logically extended with the
columns of the tables on the "one" side of its relationships: the expanded `Sales` table contains
`Products[Brand]`. Table filters (`FILTER ( Sales, … )`, context transition on `Sales`) and table
modifiers (`REMOVEFILTERS ( Sales )`, `ALLEXCEPT ( Sales, … )`) all operate on that expanded
table. A column modifier such as `REMOVEFILTERS ( Sales[Product] )` touches only that one
column. That is the whole explanation of the Brand example above.

## Common mistakes

- Using `ALL ( Fact )` for a "total" without realizing it also wipes every dimension filter
  (slicers on Brand, Date, Region) through the expanded table.
- `ALLSELECTED` in a card with no visual grouping — it behaves like "total under slicers,"
  fine, but people then expect the same inside complex iterators.
- Boolean filters overriding user selections where `KEEPFILTERS` was intended.
- Mixing up `ALL` as a modifier (removes filters) with `ALL` as a table (returns rows).

## Exercise

1. Build the (Red, A) matrix and confirm 20 / 84 / 114.
2. Put `Products[Brand]` on rows and reproduce the Brand table above. Then write a measure
   that removes the Brand filter but keeps a Color slicer (use `REMOVEFILTERS ( Products )`);
   with the slicer on Red it should return 84 on both brand rows.
3. Reproduce the KEEPFILTERS table, then add a Color slicer set to Blue and predict the card
   value of each measure. (Red Revenue: 84; Red Revenue (KF): blank.)
