---
description: "Configure Power BI relationships: one-to-many vs many-to-many, single vs both filter direction, inactive relationships, and USERELATIONSHIP."
---

# 02 · Relationships, Cardinality & Filter Direction

A relationship in Power BI is not a join that runs when you refresh. It's a **filter
path**: a rule that says "a filter on this column can reach that table." Getting
cardinality and direction right is what makes totals correct; getting them wrong produces
numbers that look plausible and aren't.

## Creating and inspecting relationships

- **Model view** (the diagram icon on the left rail): drag `DimProduct[ProductKey]` onto
  `FactSales[ProductKey]`.
- Or **Home → Manage relationships → New**.
- Double-click a relationship line to open its properties: the two tables and columns,
  **Cardinality**, **Cross filter direction**, **Make this relationship active**, and
  (for one-to-one/both) **Apply security filter in both directions**.

Power BI's *Autodetect* may create relationships when you load tables with matching column
names. Review every one; turn autodetect off (**File → Options → Current File → Data
Load → Relationships**) in serious models.

## Cardinality

| Cardinality | Meaning | Typical use |
|-------------|---------|-------------|
| One-to-many (1:*) | Key unique on the "one" side | Dimension → fact. The default and the one you want 95% of the time. |
| One-to-one (1:1) | Unique on both sides | Usually a sign two tables should be merged. |
| Many-to-many (*:*) | Neither side unique | Facts at different grains sharing a dimension attribute; use deliberately. |

The diagram shows `1` and `*` at each end.

## Filter direction

The arrow on the line shows which way filters flow:

- **Single** — from the one side to the many side. `DimProduct` filters `FactSales`.
  `FactSales` does *not* filter `DimProduct`.
- **Both** (bidirectional) — filters flow both ways.

### Worked example: what "single" means in practice

Using the Level 2 sample with single-direction relationships from both dimensions to
`FactSales`, build a table with `DimStore[Region]` and `Product Count =
COUNTROWS(DimProduct)`:

| Region | Product Count |
|--------|--------------:|
| East | 5 |
| South | 5 |
| West | 5 |

Every region shows 5 — the whole product table — because the Region filter reaches
`FactSales` but can't travel "up" from `FactSales` into `DimProduct`.

Switch the `DimProduct`–`FactSales` relationship to **Both** and it changes:

| Region | Product Count | Products (by hand) |
|--------|--------------:|--------------------|
| East | 2 | Rain Jacket (row 3), Headlamp (row 6) |
| South | 3 | Headlamp (2), Trail Tent (4), Sleeping Bag (8) |
| West | 3 | Trail Tent (1), Sleeping Bag (5), Rain Jacket (7) |

That is "products sold in the region." Useful — but you can get it without changing the
model, using a measure that asks for it explicitly:

```dax
Products Sold =
CALCULATE (
    COUNTROWS ( DimProduct ),
    CROSSFILTER ( DimProduct[ProductKey], FactSales[ProductKey], BOTH )
)
```

…or more simply `DISTINCTCOUNT ( FactSales[ProductKey] )`, which gives the same 2/3/3.
Keep relationships single-direction by default and turn on bidirectional behaviour inside
the specific measures that need it.

### Why "Both" everywhere is dangerous

- **Ambiguity:** with several bidirectional relationships, there can be two paths between
  tables. Power BI refuses to create a relationship that would make paths ambiguous, or
  deactivates one, and the surviving path may not be the one you meant.
- **Performance:** each bidirectional hop adds filter work to every query.
- **Security:** RLS filters also propagate along relationships; bidirectional paths can
  expose or hide more than intended unless configured deliberately.

## Inactive relationships and USERELATIONSHIP

Suppose `FactSales` also had a `ShipDate`. Both `OrderDate` and `ShipDate` relate to
`DimDate[Date]`, but only one relationship between two tables can be **active** (solid
line). The other is **inactive** (dashed). Measures use the active one unless told
otherwise:

```dax
Amount by Ship Date =
CALCULATE ( [Total Amount], USERELATIONSHIP ( FactSales[ShipDate], DimDate[Date] ) )
```

`USERELATIONSHIP` activates the dashed relationship for this calculation only.

## Many-to-many, deliberately

Targets are set per **Category** per month, but `DimProduct` has one row per product —
Category isn't unique there. Options:

1. Best: create a `DimCategory` table (one row per category), relate it 1:* to both
   `DimProduct` and `FactTarget`.
2. Acceptable: relate `FactTarget[Category]` to `DimProduct[Category]` as many-to-many with
   single direction *from DimProduct to FactTarget*. Power BI shows a warning icon; the
   relationship works, but you must understand that a Product filter maps to its category's
   whole target.

## How It Actually Works

Internally, a filter on the "one" side becomes a **set of key values**; the engine applies
that set to the key column on the "many" side. In the storage engine's queries (visible in
DAX Studio as xmSQL, Level 3) this looks like a `LEFT OUTER JOIN` from the fact to the
dimension plus a `WHERE` on the dimension attribute — the join is between compressed
columns using the relationship's precomputed index, not a row-by-row lookup.

**Filter propagation is transitive but directional.** A filter on `DimStore[Region]`
restricts `FactSales`; it does not restrict `DimProduct`, because that would require going
from many to one. With bidirectional enabled, the engine additionally computes the set of
`ProductKey` values that remain in the filtered fact table and applies them back to
`DimProduct` — an extra step on every query.

**Blank rows for referential integrity.** If `FactSales` contains a `ProductKey` of 99 that
doesn't exist in `DimProduct`, the engine does not drop those rows. It adds a hidden
**blank member** to `DimProduct`, and the orphaned fact rows are attributed to it. You'll
see "(Blank)" appear as a Category. That's a data-quality signal: fix it at the source or in
Power Query, don't hide it.

**Relationship columns must have matching data types.** A text `"1"` and an integer `1` will
not relate; fix types in Power Query.

## Common mistakes

- Turning on **Both** to "make a visual work" without understanding which paths now exist.
- **Duplicate keys** on the one side making Power BI create many-to-many silently.
- Ignoring the **(Blank)** member instead of investigating orphaned keys.
- Relating two fact tables directly instead of through shared dimensions.

## Exercise

1. Reproduce both Product Count tables above (single, then both) and switch back to
   single.
2. Create the `Products Sold` measure and confirm East 2, South 3, West 3 with single
   direction.
3. Add a fact row with `ProductKey = 99`, Amount 50. Observe the (Blank) category and its
   total, then remove the row. Total Amount with the orphan should be 1,265.
