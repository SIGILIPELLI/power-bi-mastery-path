---
description: "Publish from Power BI Desktop to a workspace, understand reports vs semantic models vs dashboards, and compare sharing links, workspace roles, and apps."
---

# 09 · Publishing & Sharing Basics

Up to now everything has lived in a file on your PC. Publishing moves it into the Power BI
service, where it can refresh on a schedule and reach other people. This lesson covers the
first publish and the main ways to share, with enough mechanics to avoid the usual
surprises.

!!! note "What you need"
    Publishing requires signing in to Power BI Desktop with an organizational (work or
    school) account whose tenant has the Power BI service enabled. Sharing with others
    generally requires a paid per-user license or a capacity workspace (see lesson 01).
    Microsoft offers trials and developer programs from time to time; availability
    changes, so check the current options rather than relying on this page.

## Workspaces

In the service (app.powerbi.com), **Workspaces** in the left navigation lists:

- **My workspace** — personal. Fine for experiments; don't build shared content here.
- **Shared workspaces** — created with **New workspace**. Each has members with a role:

| Role | Can do |
|------|--------|
| Admin | Everything, including managing membership and deleting the workspace |
| Member | Publish, edit, share, add Contributors/Viewers |
| Contributor | Publish and edit content; cannot share items or manage access by default |
| Viewer | View and interact; cannot edit |

## Step by step: publishing

1. In Desktop, save the report (e.g. `Trailhead Sales.pbix`).
2. **Home → Publish**. Sign in if prompted.
3. Choose the destination workspace, click **Select**.
4. When it finishes, click the **Open in Power BI** link.
5. In the workspace list you'll now see two items with the same name: a **Report** and a
   **Semantic model**.
6. If the source is a local CSV, the semantic model's settings will show that refresh
   can't run without a gateway, or credentials need configuring. That's expected — lesson
   09 of Level 2 covers refresh. For now, the published data is a snapshot.

Publishing again with the same file name to the same workspace asks whether to **Replace**
the existing items. Replacing keeps the item IDs, so links, dashboards and shared access
keep working.

## Report vs semantic model vs dashboard

- Open the **report**: pages look like Desktop, with slicers and interactions. Readers can
  use **Edit** only if they have edit rights.
- Open the **semantic model** item: settings for refresh, credentials, security, and
  endorsement; also **Explore** and **Create report**, which build new reports on the same
  model.
- Build a **dashboard**: in the report, hover a visual → **Pin visual** (pin icon) →
  *New dashboard*, name it. Pin two or three visuals. The dashboard is a single page of
  tiles; clicking a tile goes back to the report page it came from.

## Ways to share

| Method | Best for | Notes |
|--------|----------|-------|
| **Share** button (links / direct access) | A few colleagues | Grants access to one item; can allow or block reshare and building on the model. |
| **Workspace roles** | The team that builds and maintains content | Everyone in the workspace sees everything in it. |
| **Apps** | A wider audience of readers | Bundle reports/dashboards; audiences can see different subsets; readers get a stable, curated experience while you keep editing in the workspace. |
| **Teams / SharePoint embedding** | Where people already work | Still respects permissions. |
| **Publish to web** | Genuinely public data only | Anyone with the link can see it, no sign-in. Often disabled by tenant admins. Never use for internal data. |

For the Level 1 project, share the report with one colleague by **Share → specific
people**, and untick **Allow recipients to build content with the data associated with
this report** unless you want them building on your model.

## How It Actually Works

A `.pbix` is a ZIP archive containing, among other parts, a **report layout** (JSON), the
**data model** (a serialized Analysis Services database — the `DataModel` part), and the
Power Query **mashup** definitions. On publish, Desktop uploads the file; the service
splits it:

- The model is loaded into the service's Analysis Services infrastructure as a
  **semantic model**. Imported data comes along as it was at the last refresh in Desktop.
- The report layout becomes a separate **report** item whose data binding points to that
  semantic model by ID — a *live connection*, in effect.

Because they're separate, you can later rebind a report to a different model, delete a
report without touching the model, or create many thin reports on one model.

Access checks happen at query time. When a reader opens a report, the service checks their
permission on the report, and every visual's DAX query executes under their identity
against the model — which is how row-level security (Level 2) filters data per user even
though everyone opens the same report. A shared link does not bypass that check: it grants
a permission; it doesn't hand over data.

## Common mistakes

- **Building everything in My workspace**, then leaving the company. Content in personal
  workspaces is awkward to recover.
- **Deleting and republishing** instead of replacing, which breaks dashboard tiles and
  shared links because IDs change.
- **Using Publish to web for internal reports.** It is public on the internet.
- **Granting workspace Member to readers.** Readers should get Viewer, or an app.

## Exercise

1. Publish your report to a new workspace called `Trailhead – Learning`.
2. Pin the revenue card and the region chart to a new dashboard. Click a tile and confirm
   it navigates to the report.
3. Write down, for a hypothetical 200-person sales team plus 3 analysts, which of the
   sharing methods above you would use for each group and why.
