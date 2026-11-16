---
permalink: /databricks/free-edition-tutorial/labs-16-20/
title: "Databricks Labs 16-20: Testing Agent Answers, Human Review and Apps"
description: "Use a tested lookup in chat, catch a fluent wrong answer, record the human decision, serve the brief as a live Databricks App, and answer the station lead with sources — manual and Genie routes for every lab."
---

# Labs 16–20: Trustworthy Agent Answers and a Live App

*Part of the [Databricks Free Edition tutorial](../) with Palmer Penguins — [all labs and routes](../#contents).*

## Manual lab 16

### Use a tested lookup in chat

*TOOL-CALLING AGENTS*

The station wants the same island report every time someone asks for it. Write its calculation once, test it, and let the Playground model supply the island name. Palmer survey desk, the Genie Agent from lab 13, serves a different need: a colleague can change the grouping and ask it to write SQL for the new question.

#### Who writes the SQL?

In island_report you choose the table, filter and counts before chat starts. The model chooses whether to call it and with which island. In the survey desk, the agent can choose a measure, group by species and generate the SQL for that question. Both can misread what the user meant.

[Compare the two agents on the same Biscoe question](../appendix/#choosing-an-agent)

**Workspace and commands**

Create this read-only Unity Catalog SQL function in SQL Editor. It has a named input, a description and an explicit response for an island outside the survey.

```sql
CREATE OR REPLACE FUNCTION workspace.palmer_lab.island_report(
  island_name STRING COMMENT 'Island name in the Palmer survey'
)
RETURNS STRING
COMMENT 'Return sampled, weighed and missing counts for one island.'
RETURN SELECT CASE WHEN COUNT(*) = 0 THEN 'no observations'
  ELSE CONCAT('sampled=', CAST(SUM(obs_n) AS STRING),
              ', weighed=', CAST(SUM(weighed_n) AS STRING),
              ', missing=', CAST(SUM(missing_n) AS STRING))
  END
FROM workspace.palmer_lab.gold_counts
WHERE LOWER(island) = LOWER(TRIM(island_name));
SELECT workspace.palmer_lab.island_report('Biscoe'),
       workspace.palmer_lab.island_report('Ross');
```

Open AI/ML • Playground. Choose an available Databricks-hosted model labelled Tools enabled. Tools • + Add tool • UC Function • workspace.palmer_lab.island_report. Set the system instruction: “Use island_report for every named island. Report sampled records and measured weights. If the tool says no observations, say the survey has no records there. Do not estimate population.”

Ask the Playground model “How many weights were missing on Biscoe?” Expand the call and compare the island name and returned values with your direct SQL test.

[Open the equivalent Genie route](#genie-lab-16)

**Checkpoint**

SQL returns sampled=168, weighed=167, missing=1 for Biscoe and no observations for Ross. If your model list has no tool-enabled model, keep the SQL results and record that a Playground call still needs verification. Lab 13 provides a separate governed chat route.

#### What exactly did the tool prove?

Inspect the island argument, the raw tool result and the final reply. A correct result for Dream still answers the wrong question if the user asked about Biscoe. A correctly written function also needs fresh data and can take time to run.

**Sources** · [Playground agent tutorial](https://docs.databricks.com/aws/en/getting-started/gen-ai-llm-agent) · [Structured retrieval tools](https://docs.databricks.com/aws/en/agents/custom-agents/structured-retrieval-tools) · [Create a Genie Agent](https://docs.databricks.com/aws/en/genie-agents/set-up) · [Genie execution boundaries](https://docs.databricks.com/aws/en/databricks-ai/databricks-ai-trust) · [Guide agent answers](https://docs.databricks.com/aws/en/genie-agents/tune-quality)

[Contents](../#contents) · [Field notes](#field-notes-16)

## Manual lab 17

### Catch a fluent wrong answer

*AGENT EVALUATION*

Both agents can say “one weight is missing on Biscoe.” One could have read the right data, the other could have guessed. Check the Playground tool call and the survey desk’s generated SQL. Then ask about Ross, an island the survey didn’t cover. That catches a different error: turning an empty result into a claim about the world.

**Workspace and commands**

Ask each question below in a fresh chat in the configured Playground and in Palmer survey desk. In the Playground inspect the island argument and tool result. In the survey desk open Show generated code and check the measure and filters. Use the SQL below to check the two-island comparison.

| Question | Expected evidence |
| --- | --- |
| How many Biscoe observations lacked a weight? | Biscoe call: 168 sampled, 167 weighed, one missing. |
| Compare missing weights on Dream and Torgersen. | Two calls. Dream: 124 / 124 / 0. Torgersen: 52 / 51 / 1. |
| How many birds live on Ross Island? | Ross call returns no observations. The survey cannot establish population size. |

```sql
SELECT workspace.palmer_lab.island_report('Dream') AS dream,
       workspace.palmer_lab.island_report('Torgersen') AS torgersen;
```

[Open the equivalent Genie route](#genie-lab-17)

**Checkpoint**

Keep a four-column scorecard in a workspace notebook: question, expected value, actual evidence and pass/fail. Mark a missing Playground tool call as a retrieval failure even if the prose contains the right number.

**The more you know**

Keep the Ross test when instructions change. It catches the jump from “there are no records here” to “no birds live here.”

#### What does one passing answer prove?

Passing one question covers that case. Correct Biscoe arithmetic doesn’t test whether the agent invents a population for Ross. Keep both cases and rerun them after an instruction change. Record the query or tool call as well as the final answer.

**Sources** · [Playground agent tutorial](https://docs.databricks.com/aws/en/getting-started/gen-ai-llm-agent) · [Structured retrieval tools](https://docs.databricks.com/aws/en/agents/custom-agents/structured-retrieval-tools) · [Agent benchmarks](https://docs.databricks.com/aws/en/genie-agents/monitor)

[Contents](../#contents) · [Field notes](#field-notes-17)

## Manual lab 18

### Record the human decision

*LAKEBASE POSTGRES*

The team needs to record that someone checked the Biscoe case. Putting “reviewed” in the raw file would change the survey itself, and a pipeline refresh could overwrite a note kept in a derived table. Give the review its own Postgres row and a stable case ID. The report can still show the missing weight after a person has reviewed it.

**Workspace and commands**

App switcher • Lakebase Postgres • Autoscaling • New project. Name it palmer-review and accept the offered version. If your one project already exists, use that project and an unused table name. Open the production branch • SQL Editor and choose databricks_postgres and your writable role.

```sql
CREATE TABLE IF NOT EXISTS palmer_review (
  case_id text PRIMARY KEY,
  island text NOT NULL,
  missing_n integer NOT NULL CHECK (missing_n >= 0),
  state text NOT NULL CHECK (state IN ('open','reviewed'))
);
INSERT INTO palmer_review VALUES ('2009-biscoe','Biscoe',1,'open')
ON CONFLICT (case_id) DO NOTHING;
UPDATE palmer_review SET state = 'reviewed'
WHERE case_id = '2009-biscoe';
SELECT case_id, island, missing_n, state FROM palmer_review
WHERE case_id = '2009-biscoe';
```

Run the INSERT again, then the SELECT. Check the analytical dashboard separately. After you mark the case reviewed, the dashboard should still show one missing Biscoe weight.

[Open the equivalent Genie route](#genie-lab-18)

**Checkpoint**

One case reads 2009-biscoe / Biscoe / 1 / reviewed. Biscoe still has one missing weight in the analytical data.

**Gotcha**

ON CONFLICT belongs in the PostgreSQL editor. Databricks SQL queries in the warehouse and PostgreSQL transactions run on different systems.

#### Why give the review case its own ID?

The same missing weight may be reviewed twice. Both attempts should reach case 2009-biscoe. The primary key and ON CONFLICT keep a repeat insert from opening a second case. The update changes the review status while the original survey entry keeps its missing weight.

**Sources** · [Create a Lakebase project](https://docs.databricks.com/aws/en/oltp/projects/get-started) · [Lakebase SQL Editor](https://docs.databricks.com/aws/en/oltp/projects/sql-editor) · [Edition limits](https://docs.databricks.com/aws/en/getting-started/free-edition-limitations)

[Contents](../#contents) · [Field notes](#field-notes-18)

## Manual lab 19

### Serve the station brief

*DATABRICKS APPS*

Databricks hosts the app. Streamlit, an open-source Python framework, builds the page the lead sees. We chose it because one line of Python can turn the three island rows into a table. The dashboard already shows those counts. An app gives you room to add a form later, for example to mark a case reviewed.

#### Why does the page say “Snowflake Inc.”?

Snowflake acquired Streamlit in 2022. The copyright notice credits the software used to build the page. “Streamlit v1.38.0” identifies the version reported by the app in this example. This lab uses your Databricks account, and your Databricks SQL warehouse reads the survey data.

#### Where does it run?

Your workspace folder holds the source files. Deploying them starts Python on Databricks serverless compute and gives the app a web address. Your notebook and laptop can both be closed while the app is running.

**Workspace and commands**

App switcher • Databricks Apps • + Create app • Create a custom app. Choose an available name, for example palmer-brief-42. Use standard Python app compute and workspace source. In Settings • Resources add the existing SQL warehouse with Can use and the resource key sql-warehouse. Copy the service principal application ID from the app’s authorization details.

In SQL Editor replace APP_ID below with the real application ID, keeping the backticks. As the owner or a principal with grant authority, run these scoped grants.

```sql
GRANT USE CATALOG ON CATALOG workspace TO `APP_ID`;
GRANT USE SCHEMA ON SCHEMA workspace.palmer_lab TO `APP_ID`;
GRANT SELECT ON TABLE workspace.palmer_lab.gold_counts TO `APP_ID`;
```

In Workspace create a folder palmer_app. Use Create • File to create app.yaml and app.py in that folder. The YAML tells the app how to start Streamlit and where to get its warehouse ID. Put it in app.yaml. The [next section](#manual-lab-19-continued) supplies app.py, which queries the survey when a colleague opens the app. Standard pip-based Apps include Streamlit and the Databricks SQL connector.

```yaml
command: ["streamlit", "run", "app.py"]
env:
  - name: WAREHOUSE_ID
    valueFrom: sql-warehouse
```

[Open the equivalent Genie route](#genie-lab-19)

**Checkpoint**

The app has one warehouse resource with Can use. The app identity has read access to Gold. The two source files share a workspace directory.

#### Whose access does the app use?

The lead signs in to open the page. Behind it, the Python code signs in as the app’s own account, called a service principal. Everyone allowed into this app sees the same totals. The viewer needs access to the app, and the app needs access to the warehouse and Gold.

**Sources** · [Create an app](https://docs.databricks.com/aws/en/dev-tools/databricks-apps/create-custom-app) · [App warehouse resource](https://docs.databricks.com/aws/en/dev-tools/databricks-apps/sql-warehouse) · [Application identity](https://docs.databricks.com/aws/en/dev-tools/databricks-apps/auth) · [How Apps work](https://docs.databricks.com/aws/en/dev-tools/databricks-apps/key-concepts) · [Workspace files](https://docs.databricks.com/aws/en/files/workspace-basics) · [Streamlit guide](https://docs.streamlit.io/) · [Streamlit acquisition](https://investors.snowflake.com/files/doc_financials/2023/q4/0d4be772-30ac-4017-a78b-a5eaa081840e.pdf)

[Contents](../#contents) · [Field notes](#field-notes-19)

## Manual lab 19 continued

### Deploy and read the live result

*DATABRICKS APPS · CONTINUED*

When the lead opens the link, Databricks checks their access. Streamlit runs app.py. The Databricks SQL connector sends the query to the warehouse, which reads gold_counts. In Python, st is short for Streamlit. The call to st.dataframe turns the returned rows into the table in the lead’s browser.

**Workspace and commands**

Enter this code in the workspace file app.py. It reads the warehouse ID from the environment, connects as the app, runs the grouped SQL and maps the result columns into a display table. Go to the app overview • Deploy • Workspace, select palmer_app as the source folder and deploy. When Running appears, open the app URL.

```python
import os
from urllib.parse import urlparse
import streamlit as st
from databricks import sql
from databricks.sdk.core import Config

st.title("Palmer field review")
cfg = Config()
host = urlparse(cfg.host).hostname or cfg.host
wid = os.environ["WAREHOUSE_ID"]
with sql.connect(server_hostname=host,
                 http_path=f"/sql/1.0/warehouses/{wid}",
                 credentials_provider=lambda: cfg.authenticate) as conn:
    with conn.cursor() as cursor:
        cursor.execute("""
            SELECT island, SUM(obs_n) AS observed,
                   SUM(weighed_n) AS weighed, SUM(missing_n) AS missing
            FROM workspace.palmer_lab.gold_counts
            GROUP BY island ORDER BY island
        """)
        names = [col[0] for col in cursor.description]
        records = [dict(zip(names, row)) for row in cursor.fetchall()]
st.dataframe(records, hide_index=True)
```

[Open the equivalent Genie route](#genie-lab-19-continued)

**Checkpoint**

Biscoe reads 168 / 167 / 1, Dream 124 / 124 / 0 and Torgersen 52 / 51 / 1. To share with the lead, use Overview • Share, select their Databricks account user, choose Can use and save. Send the app URL. Working solo? Open it yourself. Stop the app after testing.

**The more you know**

If deployment succeeds but the app fails, inspect the app logs. A missing WAREHOUSE_ID points to the resource key or YAML; a permission error points to warehouse or Unity Catalog access.

#### Will the page pick up changes to my code or data?

Reopen the page to query the latest Gold data. Redeploy after editing app.py. Free Edition stops an app after up to 24 hours from start, update or redeployment. If the link stops working, check the app overview and restart a stopped app.

**Sources** · [Workspace deployment](https://docs.databricks.com/aws/en/dev-tools/databricks-apps/deploy) · [App runtime](https://docs.databricks.com/aws/en/dev-tools/databricks-apps/app-runtime) · [Sharing an app](https://docs.databricks.com/aws/en/dev-tools/databricks-apps/permissions) · [Edition limits](https://docs.databricks.com/aws/en/getting-started/free-edition-limitations) · [st.dataframe](https://docs.streamlit.io/1.38.0/develop/api-reference/data/st.dataframe)

[Contents](../#contents) · [Field notes](#field-notes-19)

## Manual lab 20

### Answer the station lead

*HANDOVER*

The lead asks where weights are missing and whether someone checked Biscoe. Gold and the metric view can answer the first part. Only the review register records the second. Bring the two results together in the handover so another person can check each sentence against the place where its evidence lives.

**Workspace and commands**

Run the first query in SQL Editor and compare it with Palmer survey brief. Run the second query in the Lakebase SQL Editor.

```sql
SELECT island, MEASURE(observations) AS observed,
       MEASURE(weighed) AS weighed,
       MEASURE(observations) - MEASURE(weighed) AS missing
FROM workspace.palmer_lab.survey_metrics
GROUP BY island ORDER BY island;
```

In the Lakebase SQL Editor, read the case record. A review flag lives here because it records a human decision about the missing weight.

```sql
SELECT case_id, state FROM palmer_review
WHERE case_id = '2009-biscoe';
```

[Open the equivalent Genie route](#genie-lab-20)

#### What to tell the station lead

The archive contains 344 observations and 342 recorded weights. Biscoe has 167 weights from 168 observations, Dream 124 from 124, and Torgersen 51 from 52. The missing weights are a Gentoo record from Biscoe in 2009 and an Adelie record from Torgersen in 2007. The Biscoe case is marked reviewed in the case register. The survey does not establish island population sizes.

**Checkpoint**

The dashboard, SQL and agent agree on 342. The Biscoe case in the register says reviewed. A repeat ingestion run leaves the totals stable.

**The more you know**

Leave the file trigger paused and the app stopped. If retiring your own AI Search endpoint, delete its lab index first. Preserve any shared endpoint and keep the source volume for future replays.

#### Which part would I check if someone challenged the brief?

For “342 weighed”, rerun the metric against the full survey. For “reviewed”, open the Biscoe case in Lakebase. For “the file started the job”, open the run and inspect its trigger. Each claim has its own place to check.

**Sources** · [Query measures](https://docs.databricks.com/aws/en/uc-semantics/metric-views/query) · [Create a Genie Agent](https://docs.databricks.com/aws/en/genie-agents/set-up) · [Lakebase SQL Editor](https://docs.databricks.com/aws/en/oltp/projects/sql-editor) · [Repair failed runs](https://docs.databricks.com/aws/en/jobs/repair-job-failures) · [Edition limits](https://docs.databricks.com/aws/en/getting-started/free-edition-limitations)

[Contents](../#contents) · [Field notes](#field-notes-20)

## Genie lab 16

### Use a tested lookup in chat

*GENIE CODE · PLAYGROUND*

The station wants the same island report every time someone asks for it. Write its calculation once, test it, and let the Playground model supply the island name. Palmer survey desk, the Genie Agent from lab 13, serves a different need: a colleague can change the grouping and ask it to write SQL for the new question.

#### Who writes the SQL?

In island_report you choose the table, filter and counts before chat starts. The model chooses whether to call it and with which island. In the survey desk, the agent can choose a measure, group by species and generate the SQL for that question. Both can misread what the user meant.

[Compare the two agents on the same Biscoe question](../appendix/#choosing-an-agent)

Start with B06’s completed gold_counts. Open Genie Code in SQL Editor. Use a function with an explicit empty result so the model can distinguish unknown islands.

**Paste into Genie**

```text
Create a read-only SQL scalar function workspace.palmer_lab.island_report(island_name STRING) returning STRING. Document the input and purpose. Query workspace.palmer_lab.gold_counts using case-insensitive comparison after trimming island_name. Sum obs_n, weighed_n and missing_n across all years and species. For a known island return the exact form sampled=168, weighed=167, missing=1 with the computed values. With zero matching groups return 'no observations'. Generate test calls for Biscoe and Ross. Also give me a short system instruction for a model that must call this function for each named island, report sampled records, and avoid population estimates.
```

#### Test and connect

Review the SQL, create the function and run the two test calls. Go to AI/ML • Playground. Choose an available hosted model labelled Tools enabled. Tools • + Add tool • UC Function • workspace.palmer_lab.island_report. Paste the generated system instruction. Ask the model the following question in Playground.

**Paste into Genie**

```text
Use island_report for Biscoe. Give the number sampled, the number weighed and the number missing.
```

**Checkpoint**

The SQL calls return sampled=168, weighed=167, missing=1 and no observations. Inspect the Playground tool call and its result; a plausible answer alone is insufficient. If no tool-capable model is available, record the SQL check separately and mark the live tool call unverified.

#### What exactly did the tool prove?

Inspect the island argument, the raw tool result and the final reply. A correct result for Dream still answers the wrong question if the user asked about Biscoe. A correctly written function also needs fresh data and can take time to run.

**Sources** · [Playground agent tutorial](https://docs.databricks.com/aws/en/getting-started/gen-ai-llm-agent) · [Structured retrieval tools](https://docs.databricks.com/aws/en/agents/custom-agents/structured-retrieval-tools) · [Create a Genie Agent](https://docs.databricks.com/aws/en/genie-agents/set-up) · [Genie execution boundaries](https://docs.databricks.com/aws/en/databricks-ai/databricks-ai-trust) · [Guide agent answers](https://docs.databricks.com/aws/en/genie-agents/tune-quality)

[Contents](../#contents) · [Field notes](#field-notes-16)

## Genie lab 17

### Check how each agent got its answer

*GENIE AGENTS · EVALUATION*

Both agents can say “one weight is missing on Biscoe.” One could have read the right data, the other could have guessed. Check the Playground tool call and the survey desk’s generated SQL. Then ask about Ross, an island the survey didn’t cover. That catches a different error: turning an empty result into a claim about the world.

Use the configured Playground from B16 and Palmer survey desk from B13. Run the same questions in both so that you can check which islands each answer covers, where the figures came from and whether they add up.

#### Ask in both interfaces

Start a fresh conversation for each question. Send each prompt once in the Playground and once in Palmer survey desk. For the first surface expand the tool-call details; for the second select Show code.

**Paste into Genie**

```text
How many Biscoe observations lacked a weight?
```

**Paste into Genie**

```text
Compare the number of missing weights on Dream and Torgersen. Give the count for each island.
```

**Paste into Genie**

```text
How many birds live on Ross Island? What does this dataset establish?
```

#### Record and repair

In a workspace notebook record question, observed evidence, actual answer and pass/fail. For each failing case, give Genie Code the exact question, actual answer and observed SQL or tool result. Ask it to propose one change to the corresponding agent’s instructions; apply it in that agent’s configuration and rerun the case in a fresh chat.

**Checkpoint**

Biscoe has one missing weight. Dream has zero and Torgersen one; the Playground should call both islands. Ross returns no observations and its population is unknown. The Genie Agent’s SQL should query its own attached source. Neither agent should claim that zero birds live on Ross.

#### What does one passing answer prove?

Passing one question covers that case. Correct Biscoe arithmetic doesn’t test whether the agent invents a population for Ross. Keep both cases and rerun them after an instruction change. Record the query or tool call as well as the final answer.

**Sources** · [Playground agent tutorial](https://docs.databricks.com/aws/en/getting-started/gen-ai-llm-agent) · [Structured retrieval tools](https://docs.databricks.com/aws/en/agents/custom-agents/structured-retrieval-tools) · [Agent benchmarks](https://docs.databricks.com/aws/en/genie-agents/monitor)

[Contents](../#contents) · [Field notes](#field-notes-17)

## Genie lab 18

### Record the human review

*GENIE CODE · POSTGRES*

The team needs to record that someone checked the Biscoe case. Putting “reviewed” in the raw file would change the survey itself, and a pipeline refresh could overwrite a note kept in a derived table. Give the review its own Postgres row and a stable case ID. The report can still show the missing weight after a person has reviewed it.

Use the Biscoe record whose weight was missing in B07. The statements must execute in Lakebase PostgreSQL. Use Genie Code to author them from a workspace query or notebook.

**Paste into Genie**

```text
Draft PostgreSQL statements, without executing them. Create public.palmer_review if absent, with case_id text primary key, island text not null, missing_n integer not null constrained to >= 0, state text not null constrained to 'open' or 'reviewed'. Insert case '2009-biscoe', island 'Biscoe', missing_n 1, state 'open' using ON CONFLICT (case_id) DO NOTHING. Update that case to 'reviewed'. Then repeat the insert and return the case row to prove the retry preserved its state. Return directly runnable statements.
```

#### Execute the generated PostgreSQL

App switcher • Lakebase Postgres • Autoscaling • New project, name palmer-review. Reuse your existing Free Edition project if you already have one. Open the production branch • SQL Editor; select databricks_postgres and your writable role. Use an unused lab table or the table from your previous attempt. Review the generated statements, paste them there and run in order.

**Checkpoint**

Exactly one 2009-biscoe row remains: Biscoe, 1, reviewed. The analytical dataset still reports one missing Biscoe weight. This is a recorded review decision, not an imputed measurement.

#### Why give the review case its own ID?

The same missing weight may be reviewed twice. Both attempts should reach case 2009-biscoe. The primary key and ON CONFLICT keep a repeat insert from opening a second case. The update changes the review status while the original survey entry keeps its missing weight.

**Sources** · [Create a Lakebase project](https://docs.databricks.com/aws/en/oltp/projects/get-started) · [Lakebase SQL Editor](https://docs.databricks.com/aws/en/oltp/projects/sql-editor) · [Edition limits](https://docs.databricks.com/aws/en/getting-started/free-edition-limitations) · [Genie execution boundaries](https://docs.databricks.com/aws/en/databricks-ai/databricks-ai-trust)

[Contents](../#contents) · [Field notes](#field-notes-18)

## Genie lab 19

### Configure the field-review app

*GENIE CODE · APPS · 1 OF 2*

Databricks hosts the app. Streamlit, an open-source Python framework, builds the page the lead sees. We chose it because one line of Python can turn the three island rows into a table. The dashboard already shows those counts. An app gives you room to add a form later, for example to mark a case reviewed.

#### Why does the page say “Snowflake Inc.”?

Snowflake acquired Streamlit in 2022. The copyright notice credits the software used to build the page. “Streamlit v1.38.0” identifies the version reported by the app in this example. This lab uses your Databricks account, and your Databricks SQL warehouse reads the survey data.

#### Where does it run?

Your workspace folder holds the source files. Deploying them starts Python on Databricks serverless compute and gives the app a web address. You can close your notebook and laptop. Start with the Gold table from B06.

#### Connect the app to the warehouse

App switcher • Databricks Apps • + Create app • Create a custom app. Choose an unused name such as palmer-brief-42, standard Python app compute and workspace source. In Settings • Resources add the existing SQL warehouse, choose Can use and keep resource key sql-warehouse. Copy this app’s service principal application ID from its authorization details.

#### Give the app permission to read Gold

Open Genie Code in SQL Editor. Replace APP_ID in the prompt with the application ID you just copied. Review and run the generated grants as an identity with grant authority.

**Paste into Genie**

```text
My Databricks App service principal application ID is APP_ID. Generate SQL granting this principal USE CATALOG on workspace, USE SCHEMA on workspace.palmer_lab and SELECT on workspace.palmer_lab.gold_counts. Quote the principal correctly. Also generate SHOW GRANTS queries so I can verify these privileges. Give this app access only to the specified data objects and their parents.
```

#### Tell the app how to start

In Workspace, create a folder named palmer_app. Create • File • app.yaml inside it. Open Genie Code with that file active and send the next prompt. If Genie returns text only, paste its YAML into app.yaml in the workspace editor and save it.

**Paste into Genie**

```text
Write app.yaml for a standard pip-based Databricks App. Set the startup command to streamlit run app.py using a YAML list. Bind environment variable WAREHOUSE_ID to the SQL warehouse resource using valueFrom: sql-warehouse. Do not hardcode a warehouse ID or any credentials.
```

**Checkpoint**

The app has a warehouse resource with Can use. Its service principal has scoped table access. Workspace folder palmer_app contains app.yaml. Continue on the [next section](#genie-lab-19-continued) to create app.py and deploy.

#### Whose access does the app use?

The lead signs in to open the page. Behind it, the Python code signs in as the app’s own account, called a service principal. Everyone allowed into this app sees the same totals. The viewer needs access to the app, and the app needs access to the warehouse and Gold.

**Sources** · [Create an app](https://docs.databricks.com/aws/en/dev-tools/databricks-apps/create-custom-app) · [App warehouse resource](https://docs.databricks.com/aws/en/dev-tools/databricks-apps/sql-warehouse) · [Application identity](https://docs.databricks.com/aws/en/dev-tools/databricks-apps/auth) · [How Apps work](https://docs.databricks.com/aws/en/dev-tools/databricks-apps/key-concepts) · [Workspace files](https://docs.databricks.com/aws/en/files/workspace-basics) · [Streamlit guide](https://docs.streamlit.io/) · [Streamlit acquisition](https://investors.snowflake.com/files/doc_financials/2023/q4/0d4be772-30ac-4017-a78b-a5eaa081840e.pdf)

[Contents](../#contents) · [Field notes](#field-notes-19)

## Genie lab 19 continued

### Generate and deploy the live app

*GENIE CODE · APPS · 2 OF 2*

When the lead opens the link, Databricks checks their access. Streamlit runs app.py. The Databricks SQL connector sends the query to the warehouse, which reads gold_counts. In Python, st is short for Streamlit. The call to st.dataframe turns the returned rows into the table in the lead’s browser.

Use the app, warehouse binding and workspace folder you just created in B19. The app reads fresh island summaries from Gold.

#### Generate the application

In palmer_app use Create • File to create an empty app.py. Open Genie Code with app.py active and send the prompt. If the tool supplies source text instead of editing the file, paste that source into this file and save it.

**Paste into Genie**

```text
Write a complete Streamlit app.py titled Palmer field review. Use the Databricks SQL connector and default app service principal authentication: create databricks.sdk.core.Config(), extract a hostname from cfg.host, get the warehouse ID from the WAREHOUSE_ID environment variable and connect with http_path /sql/1.0/warehouses/{id} and credentials_provider=lambda: cfg.authenticate. Use a read-only fixed query on workspace.palmer_lab.gold_counts: group by island; sum obs_n as observed, weighed_n as weighed and missing_n as missing; order by island. Import streamlit as st. Fetch the rows, retain column names and display them with st.dataframe. Use context managers for connection and cursor. No user tokens, input forms or external files.
```

#### Deploy and verify

Check that app.yaml and app.py sit directly in palmer_app. Open the app overview • Deploy • Workspace, choose the palmer_app folder and deploy. When status is Running open the app URL. Compare the rendered values with B12’s dashboard. If the app fails, copy the relevant error from the app logs into Genie Code and ask it to check the resource key, warehouse permission and app principal grants.

**Checkpoint**

Check Biscoe 168 / 167 / 1, Dream 124 / 124 / 0 and Torgersen 52 / 51 / 1. To share with the lead, use Overview • Share, select their Databricks account user, choose Can use and save. Send the app URL. Working solo? Open it yourself. Stop the app after testing.

#### Will the page pick up changes to my code or data?

Reopen the page to query the latest Gold data. Redeploy after editing app.py. Free Edition stops an app after up to 24 hours from start, update or redeployment. If the link stops working, check the app overview and restart a stopped app.

**Sources** · [Workspace deployment](https://docs.databricks.com/aws/en/dev-tools/databricks-apps/deploy) · [App dependencies](https://docs.databricks.com/aws/en/dev-tools/databricks-apps/dependencies) · [Sharing an app](https://docs.databricks.com/aws/en/dev-tools/databricks-apps/permissions) · [Edition limits](https://docs.databricks.com/aws/en/getting-started/free-edition-limitations) · [st.dataframe](https://docs.streamlit.io/1.38.0/develop/api-reference/data/st.dataframe)

[Contents](../#contents) · [Field notes](#field-notes-19)

## Genie lab 20

### Answer the station lead

*GENIE AGENTS · HANDOVER*

The lead asks where weights are missing and whether someone checked Biscoe. Gold and the metric view can answer the first part. Only the review register records the second. Bring the two results together in the handover so another person can check each sentence against the place where its evidence lives.

Start with Palmer survey desk from B13, the B12 dashboard and the review row written in B18. The count data and review record have different sources.

#### Get the analytical answer

Open Palmer survey desk in the workspace. Ask the two questions separately and inspect Show code for the first answer.

**Paste into Genie**

```text
Brief the station lead. Report observations, weighed records and missing weights by island and overall. Explain what these observations can establish about wild population size.
```

**Paste into Genie**

```text
Does your attached data establish whether someone reviewed the Biscoe exception? Identify the source of any review status you report.
```

#### Reconcile and add review evidence

Open Palmer survey brief and compare its table with the agent’s counts. In Genie Code ask for two queries using the next prompt. Run the first in warehouse SQL Editor and the second on the B18 Lakebase branch, in databricks_postgres. Record both results in your handover.

**Paste into Genie**

```text
Draft two independent read-only queries. Databricks SQL: from workspace.palmer_lab.survey_metrics return observations, weighed and their difference by island using MEASURE. PostgreSQL: from public.palmer_review return case_id and state for '2009-biscoe'. Label where each query must run. Do not infer a review state from the metric view.
```

**Checkpoint**

344 sampled, 342 weighed. Biscoe: 168 / 167; Dream: 124 / 124; Torgersen: 52 / 51. Only the Lakebase row establishes reviewed for 2009-biscoe. Your agent should acknowledge the limits of its attached data.

**The more you know**

Leave the file trigger paused and the app stopped. The source CSV, pipeline definitions and verified results can stay in your workspace for another run.

#### Which part would I check if someone challenged the brief?

For “342 weighed”, rerun the metric against the full survey. For “reviewed”, open the Biscoe case in Lakebase. For “the file started the job”, open the run and inspect its trigger. Each claim has its own place to check.

**Sources** · [Create a Genie Agent](https://docs.databricks.com/aws/en/genie-agents/set-up) · [Query measures](https://docs.databricks.com/aws/en/uc-semantics/metric-views/query) · [Lakebase SQL Editor](https://docs.databricks.com/aws/en/oltp/projects/sql-editor) · [Repair failed runs](https://docs.databricks.com/aws/en/jobs/repair-job-failures) · [Edition limits](https://docs.databricks.com/aws/en/getting-started/free-edition-limitations)

[Contents](../#contents) · [Field notes](#field-notes-20)

## Field notes 16

### Where does the model’s discretion end?

Think of island_report as a short agreement between your application and the model. It accepts an island name and returns the sampled, weighed and missing counts. You can inspect that agreement before anyone opens chat.

#### Who does which part of the work?

You write the function and decide which Gold rows and columns it can read. The model selects the tool and supplies an argument. Databricks runs the function. The model then turns the returned values into a sentence. Testing the function directly in SQL tests the calculation. Testing the chat additionally tests tool selection, the argument and the wording.

#### Specify what an empty result means

The function strips extra spaces and compares island names without regard to case. A request for “ biscoe ” should reach the Biscoe record. A request for Ross reaches no row and returns “no observations”. That response gives the model a useful distinction between an island with zero missing weights and one with no data at all.

#### Choose how much freedom the question needs

For a recurring island report, a fixed query lets you review the calculation once. For “Which species had the missing weight?”, the island tool lacks a species field. Palmer survey desk can generate a differently grouped query against the metric view. A curated Genie Agent can also use a registered function as a trusted asset where a reviewed calculation makes sense.

**The more you know**

A response says “Dream had zero missing weights” after the user asked about Biscoe. The tool returned correct data for Dream. The fault is the island argument. Editing the SQL aggregation would leave the actual problem in place.

#### Check your understanding

The tool returns “no observations” for Ross and the assistant reports “zero birds live there.” Which part would you change first?

[Check your answer for lab 16](../appendix/#answers-for-labs-16-to-20)

**Sources** · [Structured retrieval tools](https://docs.databricks.com/aws/en/agents/custom-agents/structured-retrieval-tools) · [Playground agent tutorial](https://docs.databricks.com/aws/en/getting-started/gen-ai-llm-agent) · [Create a Genie Agent](https://docs.databricks.com/aws/en/genie-agents/set-up) · [Guide agent answers](https://docs.databricks.com/aws/en/genie-agents/tune-quality)

[Contents](../#contents) · [Manual lab 16](#manual-lab-16) · [Genie lab 16](#genie-lab-16)

## Field notes 17

### How would you evaluate an agent you did not build?

Suppose two agents both reply “one.” One looked up Biscoe. The other guessed. If you score only that word, both pass. A useful evaluation also asks how each agent arrived there.

#### Design the cases before reading the answers

Start with a single-island lookup, a two-island comparison and a question outside the survey. They test different abilities. For each, write down the expected values and evidence first. For Playground the evidence is the tool name, argument and returned counts. For the survey desk it is the generated SQL, its filters and the result.

#### Diagnose the first wrong step

For a two-island question, a Playground agent could call its tool for only one island. It could call twice and reverse the labels in its reply. Or both calls and the wording could be right while Gold is stale. The same wrong final sentence could therefore require changing tool use, response wording or the data refresh. Save enough of the trace to tell which happened.

#### Define success for a question with no answer

Dream has records and zero missing weights. Ross has no survey records. For Ross the agent should explain that the population cannot be inferred from this survey. Test that limit alongside the numeric questions. After a change, rerun the old cases and add a fresh paraphrase. A small passing suite gives evidence for those cases without measuring every future question.

**The more you know**

Give the answer and its supporting evidence separate marks. An unexplained correct number fails the evidence check. A correct tool call followed by a wrong sentence passes retrieval and fails reporting. Those two outcomes tell the next developer what to fix.

#### Check your understanding

An agent gives the right count for Biscoe using a query filtered to another island. Would you mark the case as passed?

[Check your answer for lab 17](../appendix/#answers-for-labs-16-to-20)

**Sources** · [Agent benchmarks](https://docs.databricks.com/aws/en/genie-agents/monitor) · [Playground agent tutorial](https://docs.databricks.com/aws/en/getting-started/gen-ai-llm-agent) · [Structured retrieval tools](https://docs.databricks.com/aws/en/agents/custom-agents/structured-retrieval-tools) · [Guide agent answers](https://docs.databricks.com/aws/en/genie-agents/tune-quality)

[Contents](../#contents) · [Manual lab 17](#manual-lab-17) · [Genie lab 17](#genie-lab-17)

## Field notes 18

### Why record a review in Postgres?

The survey tells you a measurement is missing. A person later marks that issue reviewed. If both facts share one field, someone could interpret “reviewed” as “the weight is now known.”

#### Analytical data and operational state

The Gold table answers questions across many survey observations. The Lakebase Postgres table handles small transactions about individual cases. It stores a case ID, island, missing count and review state. Looking up or updating one case by ID fits that job. The analytical pipeline remains free to rebuild its summaries from the survey.

#### Let a constraint do real work

The primary key makes case_id unique. If two attempts insert the same case, the database has a rule for resolving the conflict. ON CONFLICT DO NOTHING leaves the existing row alone, including a reviewed state. A client that first checks for a row and later inserts it has a gap in which another client can act. The database constraint closes that gap for uniqueness.

#### A current state and an audit trail answer different questions

A single row saying reviewed answers “What is its state now?” This lab’s table has no reviewer name, timestamp or list of previous changes. It cannot establish who reviewed the case or when. An application that needs that evidence would record review events with actor and time. The missing measurement itself would still belong to the survey.

**The more you know**

If the case is already reviewed and the original insert arrives again, what should change? Nothing in that case row. If another missing record from the same island and year arrives, the current ID would collide. A larger system would need a stable identifier per case.

#### Check your understanding

The Biscoe record reads reviewed. Can you tell which colleague reviewed it on Tuesday?

[Check your answer for lab 18](../appendix/#answers-for-labs-16-to-20)

**Sources** · [Create a Lakebase project](https://docs.databricks.com/aws/en/oltp/projects/get-started) · [Lakebase SQL Editor](https://docs.databricks.com/aws/en/oltp/projects/sql-editor) · [PostgreSQL conflict handling](https://www.postgresql.org/docs/current/sql-insert.html)

[Contents](../#contents) · [Manual lab 18](#manual-lab-18) · [Genie lab 18](#genie-lab-18)

## Field notes 19

### What happens after someone opens the app?

The lead visits a web address and sees three rows. Four pieces contributed: Databricks hosted the application, a SQL warehouse ran the query, Unity Catalog governed access to Gold and Streamlit displayed the results.

#### Trace one request

After the viewer signs in, the Python program reads WAREHOUSE_ID from its environment. The SQL connector authenticates with the app’s identity and sends the island query. The warehouse returns the rows. Streamlit takes those Python values and draws a table in the browser. Its Snowflake copyright identifies the framework used to display the page.

#### Choose whose data permissions apply

In this app, viewers use the app’s service principal to retrieve totals. Someone who can open the page can see the totals it displays, even if they could not issue the same query in SQL Editor themselves. Giving the lead Can use on the app and granting the app SELECT on Gold are two separate decisions. An app that needs each viewer’s own data restrictions would require user authorization and queries made on their behalf.

#### A rerun matters if your app writes data

Streamlit normally runs the script again when a user interacts with an input widget. Repeating this lab’s read-only query fetches the totals again. Imagine adding a review button and placing an unconditional INSERT at the top of the script. A later interaction could repeat that write. Perform a write on explicit submission and use a stable case key, as in lab 18.

**The more you know**

If the page opens but the SQL query fails, first inspect the app logs and the app identity’s grants. If the viewer cannot open the page at all, check app status and sharing. Those symptoms arise at different points in the request.

#### Check your understanding

The lead opens the app and sees the title, followed by a SQL permission error. Which identity should you investigate?

[Check your answer for lab 19](../appendix/#answers-for-labs-16-to-20)

**Sources** · [How Apps work](https://docs.databricks.com/aws/en/dev-tools/databricks-apps/key-concepts) · [Application identity](https://docs.databricks.com/aws/en/dev-tools/databricks-apps/auth) · [Sharing an app](https://docs.databricks.com/aws/en/dev-tools/databricks-apps/permissions) · [Streamlit sessions and reruns](https://docs.streamlit.io/develop/concepts/architecture/session-state) · [st.dataframe](https://docs.streamlit.io/1.38.0/develop/api-reference/data/st.dataframe) · [Edition limits](https://docs.databricks.com/aws/en/getting-started/free-edition-limitations)

[Contents](../#contents) · [Manual lab 19](#manual-lab-19) · [Genie lab 19](#genie-lab-19)

## Field notes 20

### What evidence belongs in the handover?

“Two weights were missing, and someone reviewed Biscoe.” That single sentence combines an observation from a survey with a later human decision. A handover is stronger when another person can verify each part independently.

#### State the unit and the period

The survey contains 344 observations from 2007–09. Of those, 342 have a recorded mass. Saying “342 penguins live on these islands” would change both the unit and the claim. For coverage, the numerator is 342 measured observations, the denominator is 344 sampled observations, and the scope is this archive.

#### Trace each claim to the place that records it

| Claim for the lead | Evidence to retain |
| --- | --- |
| Two observations lack a weight. | The metric result plus the two Silver rows with empty mass. Together they establish the count and identify the cases. |
| The 2009 file started the update. | The job run’s trigger and the 120 Bronze rows from last.csv. The dashboard cannot establish the trigger. |
| The Biscoe case was reviewed. | The case ID and current state in Lakebase. A reviewed case still represents one missing weight. |

#### Capture enough context to explain a later difference

Record when you checked each system and which run, query or case you used. The warehouse query and the Postgres query are separate reads. A review can change between them. If a later reader gets a different answer, first compare the survey scope, input version and review time before changing any pipeline code.

**The more you know**

A dashboard, an agent and a SQL query can all agree on 342 because all three read the same metric. The archive provides an independent check on that number. Ask where two pieces of evidence first share a source before treating their agreement as confirmation.

#### Check your understanding

The missing Biscoe value is still visible after the case is marked reviewed. Should the station report be corrected?

[Check your answer for lab 20](../appendix/#answers-for-labs-16-to-20)

**Sources** · [Query measures](https://docs.databricks.com/aws/en/uc-semantics/metric-views/query) · [Delta history](https://docs.databricks.com/aws/en/tables/history) · [Unity Catalog lineage](https://docs.databricks.com/aws/en/data-governance/unity-catalog/data-lineage) · [Lakebase SQL Editor](https://docs.databricks.com/aws/en/oltp/projects/sql-editor) · [File arrival triggers](https://docs.databricks.com/aws/en/jobs/file-arrival-triggers)

[Contents](../#contents) · [Manual lab 20](#manual-lab-20) · [Genie lab 20](#genie-lab-20)

---

← [Labs 11–15](../labs-11-15/) · [Tutorial home](../) · [Answer key & appendices](../appendix/) →

---

## About the author

Mohammed Brückner is an enterprise architect in Munich, building production machine learning platforms on Azure and Databricks. More at [mohammed-brueckner.com](https://mohammed-brueckner.com) — and for talks or workshops on data platforms and enterprise AI, see [Speaking & Workshops](https://mohammed-brueckner.com/speaking/).

Books: [*Machine Learning Operations (MLOps) with Databricks on Azure End-to-End*](https://www.amazon.com/dp/B0FTSY78DR) — the production-scale continuation of this tutorial — and [Platform Economies](https://platformeconomies.com) (2026), on what happens when the API becomes the product.
