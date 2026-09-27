---
description: "Govern Power BI at scale: promoted and certified semantic models, workspace strategy, sensitivity labels, lineage and impact analysis, and admin settings."
---

# 03 · Governance & Endorsed Semantic Models

Governance is how an organization answers "which report can I trust?" and "who can see this
data?" without a committee meeting for every chart. In Power BI the practical tools are
**endorsement**, **workspace and permission design**, **sensitivity labels**, **lineage**, and **tenant
settings**. Good governance makes the trusted path the easy path.

## Endorsement: promoted and certified

Semantic models (and other items) can carry an endorsement badge:

| Level | Who can apply | Meaning |
|-------|---------------|---------|
| **Promoted** | Anyone with write permission on the item | "The owner thinks this is ready for others to use" |
| **Certified** | Only people/groups authorized by the tenant admin | "This meets the organization's quality standard" |
| **Master data** (in Fabric) | Authorized reviewers | Core reference data, the single source for entities |

Endorsed items are surfaced first in the data hub/catalog when people search for data to build on.
Apply it in the item's **Settings → Endorsement** section.

### A certification process that works

Certification should mean something testable. A practical checklist:

1. **Owner and support contact** named in the model description.
2. **Documented measures** — every visible measure has a description (shown as a tooltip in the
   Data pane).
3. **Reconciliation** — key measures tie to the system of record for a defined period (lesson 09).
4. **Security** — RLS roles tested; no personal data exposed without reason.
5. **Refresh** — scheduled, monitored, with failure notification to a group, not one person.
6. **Lifecycle** — lives in a pipeline or Git-managed workspace, not someone's personal workspace.
7. **Best-practice rules** pass (lesson 09).

Certification is reviewed periodically and revoked when the checklist no longer holds.

## Workspace strategy

- Workspaces are the **security and lifecycle boundary**. Organize them by subject area and
  stage (lesson 01), not by person.
- Assign roles to **groups**, not individuals.
- Readers consume through **apps**; they don't need workspace roles.
- Limit who can create workspaces if sprawl becomes a problem (a tenant setting).
- Use **domains** (Fabric) to group workspaces by business area for delegated administration.

## Sensitivity labels

With Microsoft Purview Information Protection configured, Power BI items can carry sensitivity labels
(e.g. *Public, General, Confidential, Highly Confidential*). Useful properties:

- Labels can be **inherited** downstream — a report built on a Confidential model inherits the label.
- When data is **exported** to Excel, PowerPoint or PDF, the label (and its protection, such as
  encryption) can travel with the file.
- Admins can require labels on content or set defaults.

Which of these behaviours are enabled depends on tenant configuration.

## Lineage and impact analysis

The **lineage view** of a workspace shows the chain: data sources → dataflows/lakehouses → semantic
models → reports/dashboards. On a semantic model, **impact analysis** lists every downstream report,
dashboard and workspace, with view counts — and lets you notify their owners.

### Worked example: a breaking change

You need to rename the measure `Revenue` to `Gross Revenue` in the certified Sales model because
Finance has introduced `Net Revenue`.

1. **Impact analysis** on the model shows, say, 14 reports in 6 workspaces.
2. Renaming a measure breaks visuals that reference it by name in thin reports. Options:
    - Keep `Revenue` as a hidden or deprecated alias (`Revenue = [Gross Revenue]`) with a description
      "Deprecated — use Gross Revenue," and remove it after a notice period.
    - Or rename and fix the 14 reports.
3. Use **Notify contacts** from impact analysis to announce the change and the date.
4. Deploy through the pipeline; check a sample of downstream reports in Test first.

The alias approach costs one hidden measure and saves 14 broken reports on Monday morning.

## Tenant settings worth knowing

Admins control these in the **Admin portal** (names vary by release):

- **Publish to web** — often disabled or restricted to a security group.
- **Export data** and **Export to Excel/CSV** — which users can export underlying or summarized data.
- **Share content with external users** (B2B guests).
- **Create workspaces**, **use semantic models across workspaces**, **certification** (who can certify).
- **Custom visuals** — allow only certified or organizational visuals.
- **Fabric item creation** — whether users can create non-Power BI Fabric items.

Governance teams review these periodically and document the reasoning for each setting.

## How It Actually Works

Endorsement and labels are **metadata on items** stored by the service. The data hub/catalog is a
search index over those items that ranks endorsed ones higher and shows their badges; nothing about
the model's contents changes when it's certified. That's why certification is only as good as the
process behind it.

Lineage is computed from the connections the service already knows about: each report stores the
ID of the semantic model it binds to, each model stores its data source definitions, and each
dataflow or Fabric item stores its inputs and outputs. The lineage view is a graph assembled from
those stored references — which is also why it can't show dependencies it doesn't know about, such as
an Excel workbook that queries the model through Analyze in Excel and is emailed around, or a
custom application calling the REST API.

Sensitivity labels are enforced by Microsoft Purview: when content is exported, the service applies
the label's protection policy to the file, so access control continues outside Power BI.

## Common mistakes

- Certifying models without a checklist — a badge nobody believes.
- Individual users in workspace roles, making access reviews impossible.
- Breaking changes in shared models without impact analysis.
- Leaving *Export data* wide open for sensitive models, then relying on RLS alone.

## Exercise

1. Write a certification checklist for your organization (adapt the seven points above) and apply it
   to your Level 2 project model. Which items fail?
2. Open lineage view for a workspace you own and list every downstream item of one semantic model.
3. Plan the `Revenue` → `Gross Revenue` rename for a model with 14 dependent reports: write the
   notification message, the alias measure, and the date you'd remove the alias.
