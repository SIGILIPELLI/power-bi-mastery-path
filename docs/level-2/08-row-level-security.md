---
description: "Implement row-level security in Power BI: static roles, dynamic RLS with USERPRINCIPALNAME and a mapping table, testing with View as, and pitfalls."
---

# 08 · Row-Level Security

Row-level security (RLS) lets one semantic model serve many audiences: the West manager
sees West, the South manager sees South, the VP sees everything — from the same report.
You define **roles** with DAX filter expressions in Desktop, then assign people or groups to
those roles in the service.

RLS restricts **data**, not report objects. Visuals, pages and measure names remain visible;
the numbers inside them are filtered.

## Static roles

1. **Modeling → Manage roles** (newer releases open a role editor with a simple builder and
   a "Switch to DAX editor" option).
2. **New role** → name it `West`.
3. Select table `DimStore`, and enter the filter expression:

```dax
[Region] = "West"
```

4. Save. Repeat for `South` and `East`.
5. Test: **Modeling → View as → West**. A yellow banner says you're viewing as a role.

Expected with the Level 2 sample (2025 rows): Total Amount **670**; by Category: Camping
430, Apparel 240; Accessories disappears (no West accessories sales).

Static roles are simple but don't scale: 50 regions means 50 roles, and moving a manager
means editing the model.

## Dynamic RLS

Store *who sees what* as data, and write one role that looks up the current user.

### Step by step

1. Create a mapping table `SecurityRegion` (Enter data, or better, from a managed source
   such as a SharePoint list or HR system):

| Email | Region |
|-------|--------|
| maria@contoso.com | West |
| maria@contoso.com | South |
| dev@contoso.com | East |

(Addresses are illustrative; use real user principal names from your tenant.)

2. Don't relate this table to anything. Hide it from report view.
3. **Manage roles → New role** `Region Managers`, table `DimStore`, filter:

```dax
[Region]
    IN CALCULATETABLE (
        VALUES ( SecurityRegion[Region] ),
        SecurityRegion[Email] = USERPRINCIPALNAME ()
    )
```

4. Also add a filter on `SecurityRegion` itself so users can't browse other people's rows:

```dax
[Email] = USERPRINCIPALNAME ()
```

5. **View as → Other user** `maria@contoso.com` **and** role `Region Managers`.

Expected for Maria: West 670 + South 315 = **985**. By Category: Camping 430 + 215 = 645,
Apparel 240, Accessories 100 (row 2, Austin).

For `dev@contoso.com`: East only, **230**. For an email that isn't in the table: every
visual is empty, because `VALUES` returns an empty table and no region matches. That's the
safe default — no mapping, no data.

## Assigning users in the service

1. Publish.
2. Workspace → semantic model **…** → **Security**.
3. Select the role and add users or, better, **security groups** (Microsoft Entra ID groups).
4. Use **Test as role** in the service to check with a real account.

Important: RLS applies to users with **Viewer** permission (and to app audiences / people
the item is shared with). Workspace **Admins, Members and Contributors** have edit rights on
the model and **are not restricted by RLS**. Testing with your own admin account will always
show everything.

## Object-level security, briefly

RLS hides rows. **Object-level security (OLS)** hides entire tables or columns (for example a
`Salary` column) from certain roles. Power BI Desktop doesn't provide a UI for OLS; you set it
with external tools such as Tabular Editor. Visuals that reference a hidden object show an
error for users in that role, so OLS needs careful report design.

## How It Actually Works

When a user in a role opens a report, the service connects to the semantic model with that
user's **effective identity** and role membership. For every table that has a role filter,
the engine evaluates the filter expression **once per row of that table** (a row context,
like a calculated column), producing the set of visible rows. That set is then applied as
a filter *before* any query-level filter — the user's query can never remove it, not even
with `ALL` or `REMOVEFILTERS`, because the rows simply don't exist in their view of the model.

From there, the filter **propagates across relationships exactly like a slicer**: a filter on
`DimStore` restricts `FactSales` through the one-to-many relationship. It does **not** flow
from `FactSales` up into `DimProduct` unless the relationship is bidirectional *and* "Apply
security filter in both directions" is enabled. That's why Accessories disappears from
visuals for the West role only when those visuals are driven by fact data — a table of
`DimProduct[Product]` alone still lists all five products.

`USERPRINCIPALNAME()` returns the signed-in user's UPN (usually their email-style sign-in
name). In Desktop, without a sign-in or "View as," it returns your own account or a local
identity — which is why you test with **View as → Other user**. (`USERNAME()` returns a
`DOMAIN\user` form in some contexts; prefer `USERPRINCIPALNAME()` for cloud RLS.)

Because the role filter is computed per user session and cached, keep role expressions simple
and based on small dimension tables. A filter on a 100-million-row fact table would be
evaluated over every row.

## Common mistakes

- **Testing as an admin** and concluding RLS "doesn't work."
- **Filtering the fact table** instead of a dimension.
- **Bidirectional relationships** that unexpectedly spread (or fail to spread) security.
- **Case or format mismatches** between the mapping table and UPNs. Normalize to lowercase:
  `LOWER ( [Email] ) = LOWER ( USERPRINCIPALNAME () )`.
- **Relying on RLS for users with edit/Build access** to download or query the model without
  the role — RLS applies to viewers; design permissions accordingly.

## Exercise

1. Create static roles for West, South and East; verify 670, 315 and 230 with View as.
2. Build the dynamic role and verify Maria (985) and Dev (230), plus an unknown user (blank).
3. Add a `DimStore` row for a new store in the West (StoreKey 4, Seattle) with no sales; add
   a table of `DimStore[Store]` for Maria. Which stores are listed and why? (Denver, Seattle,
   Austin — the role filters the dimension itself, regardless of sales.)
