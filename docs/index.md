---
title: "Free Power BI Course: Power Query, Data Modeling, DAX & Enterprise Reporting"
description: "Free Power BI course — 40 lessons on Power Query, star schemas, DAX, time intelligence, row-level security, performance tuning, and enterprise deployment."
---

# Power BI Mastery Path


<div class="mp-hero-lockup">
  <div class="mp-hero-globe">
    <svg viewBox="0 0 160 160" class="mp-globe-svg mp-mark">
      <circle cx="80" cy="80" r="74" fill="var(--md-default-bg-color, #FAF7EF)" stroke="#2FA98D" stroke-width="5"/>
      <rect class="mp-bar mp-bar-1" x="36" y="90" width="17" height="24" rx="5" fill="#2FA98D"/>
      <rect class="mp-bar mp-bar-2" x="59" y="74" width="17" height="40" rx="5" fill="#7FC9B7"/>
      <rect class="mp-bar mp-bar-3" x="82" y="56" width="17" height="58" rx="5" fill="#E0B457"/>
      <rect class="mp-bar mp-bar-4" x="105" y="38" width="17" height="76" rx="5" fill="#C97B4A"/>
      <circle class="mp-dot" cx="44.5" cy="82" r="7" fill="#22261f"/>
    </svg>
  </div>
  <div class="mp-hero-divider"></div>
  <div class="mp-hero-text">
    <div class="mp-hero-title">Mastery Path</div>
    <div class="mp-hero-sub">BOOTCAMP</div>
    <div class="mp-hero-rule"></div>
    <div class="mp-hero-byline">VINOD BHANUJA</div>
  </div>
</div>

Power BI is Microsoft's business-intelligence platform: a desktop authoring tool where
you shape data and build a model, a cloud service where that model is published,
refreshed, secured and shared, and a mobile app for reading the results. Most people
meet it as "the tool that makes dashboards," but the dashboards are the easy part. The
work that decides whether a report is right, fast, and trusted happens underneath — in
the Power Query steps that clean the data, the star schema that organizes it, and the
DAX measures that compute every number on the page.

This course is built around that idea. Level 1 gets you from a CSV file to a published
dashboard, but it spends as much time on data types and the measure-versus-column
distinction as on charts. Level 2 is modeling: fact and dimension tables, relationships
and filter direction, `CALCULATE`, filter context and row context, a proper date table,
merges and parameters in M, drill-through, and row-level security. Level 3 goes deep on
DAX evaluation (iterators, context transition, `ALL`/`REMOVEFILTERS`/`KEEPFILTERS`),
the VertiPaq engine and performance, storage modes, calculation groups, and field
parameters. Level 4 is the enterprise layer: architecture, Microsoft Fabric,
governance, storytelling, embedding, incremental refresh, source control with PBIP,
accessibility, and testing a semantic model.

Every lesson has a **How It Actually Works** section: how VertiPaq compresses a column
with dictionary and run-length encoding, how a filter on a dimension travels across a
one-to-many relationship, why `CALCULATE` inside a row context turns the current row
into a filter, how Power Query decides whether a step can "fold" back into SQL.

!!! note "About the software and the examples"
    Power BI Desktop runs only on Windows; the Power BI service runs in any modern
    browser. Microsoft ships Desktop updates roughly monthly, and menu names, ribbon
    layouts and pane labels move around between releases — so menu paths in these
    lessons describe where things live at the time of writing, and you may need to
    look one tab over. The lessons do not include screenshots, and they never claim an
    output that was not computed. Instead, DAX and M examples come with a small sample
    table defined in the lesson and the expected result **worked out by hand**, so you
    can check both your understanding and your model against it. Licensing tiers are
    described in general terms only; check Microsoft's current pricing pages for
    specifics.

## How the program is organized

| Level | Focus | Modules |
|-------|-------|---------|
| [Level 1 · Entry](level-1/index.md) | Desktop vs service, getting data, Power Query, data types, visuals, filters & slicers, measures vs columns, formatting, publishing | 9 topics + 1 project (sales dashboard from CSV) |
| [Level 2 · Intermediate](level-2/index.md) | Star schema, relationships, DAX fundamentals, CALCULATE & contexts, time intelligence, M merges & parameters, drill-through, RLS, refresh | 9 topics + 1 project (multi-table sales model) |
| [Level 3 · Advanced](level-3/index.md) | Iterators & variables, context transition, filter modifiers, VertiPaq performance, storage modes, calculation groups, field parameters, visuals, deployment pipelines | 9 topics + 1 project (optimized finance model) |
| [Level 4 · Master](level-4/index.md) | Architecture, Fabric, governance, storytelling, embedding, incremental refresh, PBIP & Git, accessibility, model testing | 9 topics + 1 capstone (enterprise sales analytics) |

## How to use this site

- Work through the levels in order. Level 2's DAX makes little sense without the
  measure-versus-column lesson in Level 1, and Level 3 explains *why* the Level 2
  formulas behave as they do.
- **Build along.** Each lesson defines its sample data inline — usually a few rows you
  can paste into Power BI Desktop with **Home → Enter data** or save as a CSV. Type the
  measures yourself and compare your result with the hand-computed answer in the lesson.
- If you are on a Mac or Linux, you can still follow the modeling and DAX reasoning,
  use the Power BI service in a browser for the publishing lessons, or run Desktop in a
  Windows virtual machine.
- Do the **Exercise** at the end of each lesson; the module-10 projects reuse them.
- Power BI rewards spreadsheet and SQL skills. If joins and `GROUP BY` are unfamiliar,
  the [SQL Mastery Path](https://sigilipelli.github.io/sql-mastery-path/) and
  [Excel Mastery Path](https://sigilipelli.github.io/excel-mastery-path/) are good
  companions.

Start here → [Level 1 · Entry](level-1/index.md)


🎥 **Prefer video?** Watch the [Mastery Path video series](https://youtube.com/@sigilipelli) on YouTube — Shorts and full walkthroughs of lessons across the series.

## More from the Mastery Path series

Free, structured, module-wise training across 97 other languages, platforms and disciplines — expand a category to browse:

<div class="mastery-tracks">
<details class="mastery-card">
<summary>Languages <span class="mastery-count">20</span></summary>
<div class="mastery-grid">
  <a href="https://sigilipelli.github.io/python-mastery-path/">🐍 Python</a>
  <a href="https://sigilipelli.github.io/java-mastery-path/">☕ Java</a>
  <a href="https://sigilipelli.github.io/javascript-mastery-path/">🟨 JavaScript</a>
  <a href="https://sigilipelli.github.io/typescript-mastery-path/">🔷 TypeScript</a>
  <a href="https://sigilipelli.github.io/csharp-mastery-path/">🔵 C#</a>
  <a href="https://sigilipelli.github.io/shell-mastery-path/">🐚 Shell/Bash</a>
  <a href="https://sigilipelli.github.io/powershell-mastery-path/">💻 PowerShell</a>
  <a href="https://sigilipelli.github.io/c-mastery-path/">🇨 C</a>
  <a href="https://sigilipelli.github.io/cpp-mastery-path/">➕ C++</a>
  <a href="https://sigilipelli.github.io/go-mastery-path/">🐹 Go</a>
  <a href="https://sigilipelli.github.io/rust-mastery-path/">🦀 Rust</a>
  <a href="https://sigilipelli.github.io/sql-mastery-path/">🗄️ SQL</a>
  <a href="https://sigilipelli.github.io/ruby-mastery-path/">💎 Ruby</a>
  <a href="https://sigilipelli.github.io/php-mastery-path/">🐘 PHP</a>
  <a href="https://sigilipelli.github.io/kotlin-mastery-path/">🟣 Kotlin</a>
  <a href="https://sigilipelli.github.io/swift-mastery-path/">🐦 Swift</a>
  <a href="https://sigilipelli.github.io/dart-mastery-path/">🎯 Dart</a>
  <a href="https://sigilipelli.github.io/scala-mastery-path/">🔴 Scala</a>
  <a href="https://sigilipelli.github.io/r-mastery-path/">📊 R</a>
  <a href="https://sigilipelli.github.io/matlab-mastery-path/">🟧 MATLAB</a>
</div>
</details>
<details class="mastery-card">
<summary>Web Frameworks <span class="mastery-count">11</span></summary>
<div class="mastery-grid">
  <a href="https://sigilipelli.github.io/react-mastery-path/">⚛️ React</a>
  <a href="https://sigilipelli.github.io/nextjs-mastery-path/">▲ Next.js</a>
  <a href="https://sigilipelli.github.io/angular-mastery-path/">🅰️ Angular</a>
  <a href="https://sigilipelli.github.io/vue-mastery-path/">💚 Vue.js</a>
  <a href="https://sigilipelli.github.io/html-css-mastery-path/">🎨 HTML & CSS</a>
  <a href="https://sigilipelli.github.io/tailwind-mastery-path/">🌬️ Tailwind CSS</a>
  <a href="https://sigilipelli.github.io/django-mastery-path/">🎸 Django</a>
  <a href="https://sigilipelli.github.io/fastapi-mastery-path/">🚀 FastAPI</a>
  <a href="https://sigilipelli.github.io/graphql-mastery-path/">🕸️ GraphQL</a>
  <a href="https://sigilipelli.github.io/nodejs-mastery-path/">🟩 Node.js</a>
  <a href="https://sigilipelli.github.io/spring-boot-mastery-path/">🍃 Spring Boot</a>
</div>
</details>
<details class="mastery-card">
<summary>Mobile <span class="mastery-count">2</span></summary>
<div class="mastery-grid">
  <a href="https://sigilipelli.github.io/flutter-mastery-path/">💙 Flutter</a>
  <a href="https://sigilipelli.github.io/react-native-mastery-path/">📲 React Native</a>
</div>
</details>
<details class="mastery-card">
<summary>Testing & QA <span class="mastery-count">6</span></summary>
<div class="mastery-grid">
  <a href="https://sigilipelli.github.io/java-testing-mastery-path/">🧪 Java Testing</a>
  <a href="https://sigilipelli.github.io/cpp-testing-mastery-path/">🧪 C/C++ Testing</a>
  <a href="https://sigilipelli.github.io/python-testing-mastery-path/">🧪 Python Testing</a>
  <a href="https://sigilipelli.github.io/automotive-testing-mastery-path/">🚗 Automotive Testing</a>
  <a href="https://sigilipelli.github.io/selenium-mastery-path/">🧭 Selenium</a>
  <a href="https://sigilipelli.github.io/cypress-mastery-path/">🌲 Cypress</a>
</div>
</details>
<details class="mastery-card">
<summary>Security <span class="mastery-count">2</span></summary>
<div class="mastery-grid">
  <a href="https://sigilipelli.github.io/cybersecurity-mastery-path/">🛡️ Cybersecurity</a>
  <a href="https://sigilipelli.github.io/ethical-hacking-mastery-path/">🎯 Ethical Hacking</a>
</div>
</details>
<details class="mastery-card">
<summary>Cloud Platforms <span class="mastery-count">5</span></summary>
<div class="mastery-grid">
  <a href="https://sigilipelli.github.io/aws-mastery-path/">☁️ AWS</a>
  <a href="https://sigilipelli.github.io/azure-mastery-path/">☁️ Azure</a>
  <a href="https://sigilipelli.github.io/gcp-mastery-path/">☁️ GCP</a>
  <a href="https://sigilipelli.github.io/ibm-cloud-mastery-path/">☁️ IBM Cloud</a>
  <a href="https://sigilipelli.github.io/adobe-mastery-path/">🎨 Adobe</a>
</div>
</details>
<details class="mastery-card">
<summary>Data & Analytics <span class="mastery-count">6</span></summary>
<div class="mastery-grid">
  <a href="https://sigilipelli.github.io/data-engineering-mastery-path/">🛠️ Data Engineering</a>
  <a href="https://sigilipelli.github.io/data-science-mastery-path/">📈 Data Science</a>
  <a href="https://sigilipelli.github.io/tableau-mastery-path/">📊 Tableau</a>
  <a href="https://sigilipelli.github.io/excel-mastery-path/">📗 Excel</a>
  <a href="https://sigilipelli.github.io/pyspark-mastery-path/">⚡ PySpark</a>
  <a href="https://sigilipelli.github.io/etl-datalake-mastery-path/">🏞️ ETL & Data Lake</a>
</div>
</details>
<details class="mastery-card">
<summary>Databases <span class="mastery-count">5</span></summary>
<div class="mastery-grid">
  <a href="https://sigilipelli.github.io/postgresql-mastery-path/">🐘 PostgreSQL</a>
  <a href="https://sigilipelli.github.io/mongodb-mastery-path/">🍃 MongoDB</a>
  <a href="https://sigilipelli.github.io/redis-mastery-path/">🟥 Redis</a>
  <a href="https://sigilipelli.github.io/snowflake-mastery-path/">❄️ Snowflake</a>
  <a href="https://sigilipelli.github.io/databricks-mastery-path/">🧱 Databricks</a>
</div>
</details>
<details class="mastery-card">
<summary>AI / ML / LLM <span class="mastery-count">11</span></summary>
<div class="mastery-grid">
  <a href="https://sigilipelli.github.io/ai-ml-mastery-path/">🤖 AI/ML</a>
  <a href="https://sigilipelli.github.io/llm-dev-mastery-path/">🧠 LLM Dev</a>
  <a href="https://sigilipelli.github.io/rag-mastery-path/">📚 RAG</a>
  <a href="https://sigilipelli.github.io/edge-ai-mastery-path/">📱 Edge AI</a>
  <a href="https://sigilipelli.github.io/claude-training-mastery-path/">🔶 Claude Training</a>
  <a href="https://sigilipelli.github.io/ai-tools-mastery-path/">🧰 AI Tools</a>
  <a href="https://sigilipelli.github.io/ml-math-mastery-path/">➗ ML Math Foundations</a>
  <a href="https://sigilipelli.github.io/prompt-engineering-mastery-path/">✍️ Prompt Engineering</a>
  <a href="https://sigilipelli.github.io/ai-agents-mastery-path/">🕹️ AI Agents</a>
  <a href="https://sigilipelli.github.io/mlops-mastery-path/">🔁 MLOps</a>
  <a href="https://sigilipelli.github.io/computer-vision-mastery-path/">👁️ Computer Vision</a>
</div>
</details>
<details class="mastery-card">
<summary>Embedded Systems <span class="mastery-count">5</span></summary>
<div class="mastery-grid">
  <a href="https://sigilipelli.github.io/embedded-mastery-path/">🔌 Embedded</a>
  <a href="https://sigilipelli.github.io/embedded-linux-mastery-path/">🐧 Embedded Linux</a>
  <a href="https://sigilipelli.github.io/embedded-python-mastery-path/">🐍 Embedded Python</a>
  <a href="https://sigilipelli.github.io/freertos-mastery-path/">⏱️ FreeRTOS</a>
  <a href="https://sigilipelli.github.io/s32k-mastery-path/">🔧 S32K</a>
</div>
</details>
<details class="mastery-card">
<summary>Leadership & Management <span class="mastery-count">5</span></summary>
<div class="mastery-grid">
  <a href="https://sigilipelli.github.io/product-manager-mastery-path/">📋 Product Manager</a>
  <a href="https://sigilipelli.github.io/product-lead-mastery-path/">🧭 Product Lead</a>
  <a href="https://sigilipelli.github.io/project-manager-mastery-path/">📅 Project Manager</a>
  <a href="https://sigilipelli.github.io/ai-manager-mastery-path/">🤖 AI Manager</a>
  <a href="https://sigilipelli.github.io/servant-leadership-mastery-path/">🤝 Servant Leadership</a>
</div>
</details>
<details class="mastery-card">
<summary>Professional Skills <span class="mastery-count">2</span></summary>
<div class="mastery-grid">
  <a href="https://sigilipelli.github.io/english-fluency-mastery-path/">🗣️ English Fluency & IELTS</a>
  <a href="https://sigilipelli.github.io/workday-mastery-path/">🧑‍💼 Workday</a>
</div>
</details>
<details class="mastery-card">
<summary>Careers & Interviews <span class="mastery-count">2</span></summary>
<div class="mastery-grid">
  <a href="https://sigilipelli.github.io/dsa-interview-mastery-path/">🧩 DSA & Coding Interviews</a>
  <a href="https://sigilipelli.github.io/system-design-mastery-path/">🏗️ System Design</a>
</div>
</details>
<details class="mastery-card">
<summary>Process & APIs <span class="mastery-count">3</span></summary>
<div class="mastery-grid">
  <a href="https://sigilipelli.github.io/agile-mastery-path/">🔄 Agile/Scrum/Kanban</a>
  <a href="https://sigilipelli.github.io/rest-api-mastery-path/">🔗 REST API</a>
  <a href="https://sigilipelli.github.io/playwright-mastery-path/">🎭 Playwright</a>
</div>
</details>
<details class="mastery-card">
<summary>Infrastructure & Ops <span class="mastery-count">12</span></summary>
<div class="mastery-grid">
  <a href="https://sigilipelli.github.io/server-ops-mastery-path/">🖥️ Server Ops</a>
  <a href="https://sigilipelli.github.io/linux-mastery-path/">🐧 Linux</a>
  <a href="https://sigilipelli.github.io/nodemcu-mastery-path/">📶 NodeMCU/IoT</a>
  <a href="https://sigilipelli.github.io/terraform-mastery-path/">🌍 Terraform</a>
  <a href="https://sigilipelli.github.io/kubernetes-mastery-path/">☸️ Kubernetes</a>
  <a href="https://sigilipelli.github.io/docker-mastery-path/">🐳 Docker</a>
  <a href="https://sigilipelli.github.io/github-mastery-path/">🐙 GitHub & Git</a>
  <a href="https://sigilipelli.github.io/cicd-mastery-path/">🔁 CI/CD</a>
  <a href="https://sigilipelli.github.io/ansible-mastery-path/">🤖 Ansible</a>
  <a href="https://sigilipelli.github.io/kafka-mastery-path/">📨 Kafka</a>
  <a href="https://sigilipelli.github.io/microservices-mastery-path/">🧩 Microservices</a>
  <a href="https://sigilipelli.github.io/observability-mastery-path/">📈 Observability</a>
</div>
</details>
</div>

