---
description: "Level 4 of Power BI: enterprise architecture, Fabric, governance, storytelling, embedding, incremental refresh, PBIP and Git, and testing."
---

# Level 4 · Master <span class="level-badge">Enterprise</span>

Goal: design and run Power BI for an organization, not just a team — shared semantic models
that hundreds of reports trust, a lifecycle with source control and automated checks, refresh
that scales to years of history, reports that communicate clearly and work for everyone, and
a realistic understanding of where Power BI sits inside Microsoft Fabric.

Levels 1–3 were mostly about *building things correctly*. Level 4 is mostly about *decisions*:
where logic should live, who owns what, what to automate, and what to leave alone. Many of the
features involved depend on licensing, tenant settings, and a platform that Microsoft is still
actively changing. The lessons say plainly when something is conceptual, when it requires
capacity, and when details are likely to shift.

## Modules

1. [Enterprise BI Architecture](01-enterprise-architecture.md) — layers, shared semantic models, thin reports, and where each kind of logic belongs
2. [Power BI in Microsoft Fabric](02-microsoft-fabric.md) — OneLake, lakehouses and warehouses, Direct Lake, and what changes for Power BI teams
3. [Governance & Endorsed Semantic Models](03-governance.md) — endorsement, workspaces strategy, sensitivity labels, lineage, and tenant settings
4. [Data Storytelling & Dashboard Design](04-storytelling-design.md) — audience, questions, visual hierarchy, and honest charts
5. [Embedding Power BI](05-embedding.md) — secure embed, embed for your organization, embed for your customers, and tokens
6. [Incremental Refresh](06-incremental-refresh.md) — RangeStart/RangeEnd, partitions, folding requirements, and real-time options
7. [CI/CD with PBIP & Git Integration](07-cicd-pbip-git.md) — the project format, TMDL, workspace Git integration, and automated checks
8. [Accessible Reports](08-accessibility.md) — keyboard navigation, screen readers, colour, alt text, and testing
9. [Testing & Validating Semantic Models](09-testing-models.md) — reconciliation, DAX query tests, best-practice rules, and regression checks
10. [Capstone — An Enterprise Sales Analytics Solution](10-capstone.md) — design, build, secure, test and ship a complete solution

## What you need before starting

- Levels 1–3, especially star schemas, RLS, calculation groups and deployment pipelines.
- Access to a Power BI service tenant helps for lessons 03, 05, 06 and 07; several features need
  a Premium, PPU or Fabric capacity workspace. Where you don't have access, the lessons give a
  written design exercise instead of a hands-on one.
- Basic Git (commit, branch, pull request). The
  [GitHub Mastery Path](https://sigilipelli.github.io/github-mastery-path/) covers it.
