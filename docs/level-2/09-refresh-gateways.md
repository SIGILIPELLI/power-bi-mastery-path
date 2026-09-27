---
description: "How Power BI refresh works: scheduled and on-demand refresh, credentials, privacy levels, the on-premises data gateway, and diagnosing refresh failures."
---

# 09 · Refresh & Gateways

An imported model is a snapshot. **Refresh** re-runs every Power Query query, reloads the
tables, recalculates calculated columns and tables, and swaps in the new version. In
Desktop you click **Refresh**; in the service you schedule it — and that's where most
teams first meet credentials errors, gateways and timeouts. This lesson is mostly
conceptual: you can follow the service steps in a browser, but installing a gateway needs
a Windows machine on your network and admin rights, so it's described rather than
demonstrated.

## What can refresh where

| Source location | Refresh in the service needs |
|-----------------|------------------------------|
| Cloud source reachable from the internet (Azure SQL, SharePoint Online, Snowflake, a web API) | Credentials entered in the semantic model's settings |
| On-premises (a SQL Server in your data centre, a file share, a local CSV) | An **on-premises data gateway** plus credentials |
| Files on your own laptop (`C:\Users\...`) | Not realistic — move the file to SharePoint/OneDrive or a share the gateway can reach |

DirectQuery and live-connection models don't import data, so they don't have a data refresh
in the same sense; the gateway is still needed for on-premises DirectQuery sources.

## Step by step: scheduling a refresh (cloud source)

Assume the model reads an Excel file stored in SharePoint Online.

1. Publish the report.
2. In the workspace, find the **semantic model** → **…** → **Settings**.
3. Expand **Data source credentials** → **Edit credentials**. Choose **OAuth2** (organizational
   account), sign in, and set the **Privacy level** (Organizational is typical for company
   data).
4. Expand **Refresh** (called "Scheduled refresh" in some releases) → turn it **On**.
5. Pick a **time zone**, frequency (Daily/Weekly) and times. Shared-capacity (Pro) models
   allow a limited number of scheduled refreshes per day; capacity and PPU allow more.
   Check current limits in Microsoft's documentation.
6. Tick **Send refresh failure notifications** to the owner (and optionally other contacts).
7. **Apply**, then use **Refresh now** from the model's menu to test.
8. Check **Refresh history** (in settings, or the model's details page) for status,
   start/end times and error messages.

## The on-premises data gateway

The gateway is a Windows service you install on a machine that can reach both your data
sources (inside the network) and Azure (outbound HTTPS). Conceptual setup:

1. Download and install the **On-premises data gateway** (standard mode) on an always-on
   server — not a laptop.
2. Sign in with an organizational account and **register** the gateway with a name and a
   **recovery key** (store the key safely; you need it to move or restore the gateway).
3. Optionally add more machines to form a **cluster** for high availability.
4. In the service: **Settings (gear) → Manage connections and gateways**. Create a
   **connection** on the gateway for each source (server + database, or file path), with
   credentials the gateway will use.
5. In the semantic model's settings, under **Gateway and cloud connections**, map each
   source in the model to a gateway connection.

A **personal mode** gateway also exists; it runs under one user's account and serves only that
user. It's fine for experiments, not for shared production content.

## Worked example: diagnosing a failure

Refresh history shows:

```text
Data source error: The credentials provided for the File source are invalid.
Cluster URI: ...
Activity ID: ...
Table: FactSales.
```

Work through it:

1. **Which source?** "File" source for `FactSales`. Open Desktop → Transform data → FactSales →
   Source step: `File.Contents("C:\Data\trailhead_2025.csv")`.
2. **Can the service reach it?** A `C:\` path is on someone's PC. Either the gateway machine has
   the same path (rare, fragile) or it doesn't.
3. **Fix:** move the file to a UNC share the gateway can read (`\\fileserver\bi\trailhead_2025.csv`)
   and create a gateway connection for it, or move it to SharePoint Online and use the
   SharePoint connector (no gateway needed).
4. **Re-test** with Refresh now and read the new history entry.

Other frequent errors and their usual meaning:

| Message fragment | Likely cause |
|------------------|--------------|
| "credentials … invalid / expired" | Password or OAuth token expired; re-enter credentials |
| "gateway … offline / unreachable" | Gateway machine off, service stopped, or network change |
| "Formula.Firewall" | Privacy-level combination problem (lesson 06) |
| "timeout" | Query too slow; improve folding, filter earlier, consider incremental refresh (Level 4) |
| "column … not found" | Source schema changed; a hard-coded step references a renamed column |

## How It Actually Works

A scheduled refresh is a job the service runs against the hosted semantic model. For each
table partition, the mashup engine executes the Power Query definition. For cloud sources it
runs in Microsoft's cloud; for gateway sources, the service sends the query definition to the
gateway through **Azure Relay** — the gateway maintains an **outbound** connection, so no
inbound firewall port is opened in your network. The gateway runs the mashup engine locally,
connects to the source with the stored credentials (encrypted with keys the gateway holds),
and streams the results back, compressed, over that channel.

The results are loaded into new copies of the tables; calculated columns, calculated tables,
relationships' internal structures and hierarchies are then rebuilt (**process recalc**).
Only when all of that succeeds does the service commit the new version, so readers keep seeing
the previous data during refresh and after a failure — a refresh is transactional at the model
level by default. This is also why a refresh needs roughly *twice* the model's memory at peak:
old and new copies coexist until commit.

Query folding matters enormously here: a folded query sends one `SELECT … WHERE …` to the
source and receives only the rows needed; a non-folded one pulls whole tables through the
gateway and transforms them on the gateway machine.

## Common mistakes

- **Scheduling refresh for a file on a laptop.**
- **Gateway on a workstation** that sleeps at night.
- **Personal-account credentials** on production connections that break when the person
  leaves. Use service accounts or managed identities where supported.
- **Ignoring failure notifications**, so a report quietly shows last week's data. Put a
  "Data as of" card on the report: create a one-row Power Query table with
  `DateTimeZone.UtcNow()` at refresh time and show it.

## Exercise

1. Publish a model whose source is a file in OneDrive/SharePoint, configure credentials and a
   daily scheduled refresh, run Refresh now and read the history.
2. Add a "Data as of" query:
   `let Source = #table(type table [RefreshedUTC = datetimezone], {{DateTimeZone.UtcNow()}}) in Source`
   and show it in a card. Refresh twice and confirm it changes.
3. For your own organization (or an imagined one), write a one-paragraph gateway plan: which
   machine, clustering, who owns the recovery key, which connections, which credentials.
