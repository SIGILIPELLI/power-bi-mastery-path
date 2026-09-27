---
description: "Promote Power BI content through development, test and production with deployment pipelines: stages, deployment rules, what is copied, and data refresh."
---

# 09 · Deployment Pipelines

Editing a report that a hundred people are using right now is how production incidents happen.
**Deployment pipelines** give Power BI a lifecycle: build in a development workspace, validate in
test, promote to production — with rules that swap data sources per stage.

!!! note "Licensing and naming"
    Deployment pipelines require workspaces on a Premium, PPU or Fabric capacity. In Fabric the
    feature covers more item types than Power BI content. Stage names, the number of stages
    (commonly three by default, configurable), and the UI evolve — treat the steps below as the
    shape of the process and check current documentation for exact labels.

## The model

```mermaid
flowchart LR
  D[Development workspace] -->|Deploy| T[Test workspace]
  T -->|Deploy| P[Production workspace]
  P --> A[App for readers]
```

Each stage is a normal workspace. The pipeline knows which items in one stage correspond to which
in the next (pairing), so deploying updates the paired item rather than creating duplicates.

## Step by step

1. In the service, **Workspaces → Deployment pipelines** (or from a workspace's menu) → **Create
   pipeline**. Name it `Trailhead Sales`.
2. Assign your existing workspace to the **Development** stage (or create stages and let the
   pipeline create workspaces for later stages when you first deploy).
3. Click **Deploy** to Test. Choose *all* or *selected* items. The first deployment creates the
   Test workspace content.
4. In the Test stage, open **Deployment settings** (lightning/gear icon) → **Rules**:
    - **Data source rules**: for the semantic model, map `dev-sql` → `test-sql`.
    - **Parameter rules**: set `ServerName` parameter (lesson 06 of Level 2) to the test value.
5. **Deploy** again so the rules take effect, then **refresh** the Test semantic model — rules
   change connection details; they don't load data.
6. Validate in Test (see below), then **Deploy** to Production. Configure production rules and
   refresh.
7. Publish or update the **app** from the Production workspace for readers.

## What is — and isn't — copied

| Copied | Not copied |
|--------|------------|
| Report definitions, semantic model **metadata** (tables, measures, relationships, roles) | The **data** in the model (each stage refreshes itself) |
| Dashboards, paginated reports, dataflows (depending on type support) | Workspace membership and permissions |
| | RLS **role membership** (who's in each role) |
| | Scheduled refresh settings and credentials (configured per stage) |
| | App configuration |

Understanding this table prevents most pipeline surprises — for instance, "production shows no
data after deployment" (it needs a refresh with production credentials).

## Worked example: validating a change

You change `Achievement %` to use `[Sales Amount] + 0` so February Apparel shows 0% instead of
blank (Level 2 project). The validation plan:

1. **Compare stages**: the pipeline shows items that differ between stages; the semantic model
   is flagged as changed. Newer UIs show a change review listing modified measures — check that
   only the intended measure changed.
2. **In Test, with test data**, open the matrix and confirm February Apparel and March Accessories
   (the two cells with a target but no sales) now show **0.0%** and every other cell is unchanged (Camping 75.9%, Apparel 91.4%, Accessories 96.2%; total 83.2%).
3. **Regression check**: row counts (`COUNTROWS(FactSales)` = 8 in the sample) and a few
   headline measures match expected values (lesson 09 of Level 4 automates this).
4. **Deploy to Production** and refresh; repeat the headline check in production.

## Backward deployment and hotfixes

Pipelines can deploy **backwards** (e.g. Production → Test) in some configurations, but only into
an empty or compatible stage. A cleaner hotfix process: fix in Development, fast-track through
Test, deploy. If you find yourself editing in production, the process is broken.

## How It Actually Works

A pipeline is metadata linking workspaces into ordered stages, plus a pairing table matching items
across stages (initially by name and type, then by stored IDs). **Deploy** takes the item
definitions from the source stage and overwrites the paired items in the target. For a semantic
model, that means applying the model definition — the same tabular metadata you'd see in a PBIP
folder or through the XMLA endpoint — to the target model: new measures added, changed ones
replaced, and data partitions left in place where the structure allows it. Where a schema change
invalidates stored data (a changed column type, a new table), the affected parts require a refresh
before they have data.

Deployment rules are applied as part of that write: the pipeline substitutes the target stage's
data source or parameter values into the definition before it's saved, which is why a deploy is
needed after changing a rule. Reports are rebound automatically to the paired semantic model in
the target stage, so a production report never points at the test model.

## Common mistakes

- Expecting data to travel with deployments.
- Forgetting to configure refresh and credentials in each stage.
- Giving readers access to Test or Development workspaces; readers should see only the production
  app.
- Making "just one quick fix" directly in production and then overwriting it with the next deploy.

## Exercise

1. If you have a capacity or PPU workspace: create a two- or three-stage pipeline, deploy your Level
   2 project, add a parameter rule, deploy, refresh, and confirm the rule's effect.
2. Without capacity access: write a one-page release checklist for the Trailhead model — what to
   compare, which numbers to verify in Test, who approves the production deploy, and how to roll
   back (redeploy the previous version from source control; see Level 4, lesson 07).
3. Fill in the "copied / not copied" table from memory, then check it against this lesson.
