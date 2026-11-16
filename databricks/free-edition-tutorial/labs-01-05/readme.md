---
permalink: /databricks/free-edition-tutorial/labs-01-05/
title: "Databricks Free Edition Labs 1-5: CSV to Bronze with SQL or Genie"
description: "Create a workspace volume, stage the Palmer Penguins CSV, verify counts with TRY_CAST, load Bronze with read_files and a declarative pipeline, and deduplicate into Silver — each lab in a manual SQL route and a Genie route."
---

# Labs 01–05: From Raw CSV to Bronze and Silver

*Part of the [Databricks Free Edition tutorial](../) with Palmer Penguins — [all labs and routes](../#contents).*

## Manual lab 01

### Create your work area

*WORKSPACE*

Tomorrow a job will need to read the file you save today, after your notebook session has ended. Give both a permanent address. Use workspace as the catalog, palmer_lab as the schema for the project’s tables, and a volume called landing for its files. A path inside that volume works in both the notebook and the job.

**Workspace and commands**

Sign in to Free Edition. In SQL Editor open a query, select the existing serverless SQL warehouse and run these read-only checks. Run the first line to see your session defaults. If you already created the schema, run the second to list its volumes. On a fresh workspace, go straight to the creation statements after the first line.

```sql
SELECT current_user(), current_catalog(), current_schema();
SHOW VOLUMES IN workspace.palmer_lab;
```

#### If the objects already exist

Your Genie transcript reports that it created workspace.palmer_lab and landing. If the volume list includes landing, you have completed setup. Continue to [Manual lab 02](#manual-lab-02).

#### For a fresh run

Run these statements in order to create the schema and its managed volume. IF NOT EXISTS also makes it safe to repeat this step.

```sql
CREATE SCHEMA IF NOT EXISTS workspace.palmer_lab;
CREATE VOLUME IF NOT EXISTS workspace.palmer_lab.landing;
SHOW VOLUMES IN workspace.palmer_lab;
```

[Open the equivalent Genie route](#genie-lab-01)

**Checkpoint**

Catalog • workspace • palmer_lab • landing opens the volume. Its path is /Volumes/workspace/palmer_lab/landing.

**Gotcha**

Seeing default as the current schema is normal. The lab objects live in palmer_lab. All SQL above specifies which schema it uses.

#### Where will I find the data tomorrow?

Open Catalog and look under workspace • palmer_lab. The landing volume holds the CSV files. Later, the survey tables will appear alongside the volume. You can close the notebook and reopen it later. The saved files and tables will still be there.

**Sources** · [Free Edition overview](https://docs.databricks.com/aws/en/getting-started/free-edition) · [Free Edition vs. trial](https://docs.databricks.com/aws/en/getting-started/free-trial-vs-free-edition) · [First table and privileges](https://docs.databricks.com/aws/en/getting-started/create-table) · [Work with volume files](https://docs.databricks.com/aws/en/volumes/volume-files) · [Genie Code](https://docs.databricks.com/aws/en/genie-code/)

[Contents](../#contents) · [Field notes](#field-notes-01)

## Manual lab 02

### Split the survey into two deliveries

*NOTEBOOKS*

The publisher gives you all three seasons in one CSV. You need a second delivery to find out whether a new file can start a job. Keep the original for comparison, put 2007–08 in the inbox now and hold back 2009. Using real survey years gives each delivery a count you can check.

**Workspace and commands**

In Databricks choose New • Notebook. Name it stage_deliveries, select Python and serverless compute. Cell 1 checks for an existing archive or fetches the publisher’s copy. It normalizes line endings and compares the file’s SHA-256 fingerprint with the known source before saving. Run it first. If GitHub is blocked, run the two offline cells in [offline step 1](../appendix/#offline-archive-step-1) and [offline step 2](../appendix/#offline-archive-step-2) instead, then return here for cell 2.

```python
from pathlib import Path
from urllib.request import urlopen
import hashlib
root = Path("/Volumes/workspace/palmer_lab/landing")
target = root / "penguins.csv"
url = ("https://raw.githubusercontent.com/allisonhorst/"
       "palmerpenguins/main/inst/extdata/penguins.csv")
if target.exists():
    data = target.read_bytes()
else:
    with urlopen(url, timeout=30) as response:
        data = response.read()
data = data.replace(b"\r\n", b"\n").rstrip(b"\n") + b"\n"
digest = ("f204db2c753b0937caac3cb35258562c"
          "14f073e4bbc76be24b4c51ce22767a93")
assert hashlib.sha256(data).hexdigest() == digest, "Use pp. 29-30"
if not target.exists():
    target.write_bytes(data)
```

Cell 2 keeps the original spelling of every entry, including NA for a missing value. It writes 2007–08 into the watched inbox and 2009 into hold, where the pipeline cannot see it yet.

```python
import pandas as pd
from pathlib import Path
root = Path("/Volumes/workspace/palmer_lab/landing")
df = pd.read_csv(root / "penguins.csv", dtype=str, keep_default_na=False)
late = df["year"].eq("2009")
for name, batch in [("inbox/first.csv", df[~late]),
                    ("hold/last.csv", df[late])]:
    path = root / name
    path.parent.mkdir(exist_ok=True)
    if not path.exists():
        batch.to_csv(path, index=False)
    print(path.name, len(batch))
```

[Open the equivalent Genie route](#genie-lab-02)

**Checkpoint**

The cell prints first.csv 224 and last.csv 120. Inside inbox you should see first.csv only. Keep last.csv in hold until lab 06.

**Gotcha**

Why keep three files? penguins.csv is the original you can check again. Only inbox is watched. Putting the archive or the held file there would count those survey rows too early or twice.

**Sources** · [Publisher’s CSV](https://raw.githubusercontent.com/allisonhorst/palmerpenguins/main/inst/extdata/penguins.csv) · [Dataset and original studies](https://github.com/allisonhorst/palmerpenguins) · [CC0 licence](https://allisonhorst.github.io/palmerpenguins/LICENSE.html) · [Work with volume files](https://docs.databricks.com/aws/en/volumes/volume-files) · [Edition limits](https://docs.databricks.com/aws/en/getting-started/free-edition-limitations)

[Contents](../#contents) · [Field notes](#field-notes-02)

## Manual lab 03

### Count the records and weights

*SQL BASICS*

If the dashboard later says 342 penguins were weighed, how will you know it counted correctly? Get an answer directly from the original CSV before writing the pipeline. Count every observation, then count the weights that really are numbers. Break the totals down by year so you can check each delivery separately.

**Workspace and commands**

In SQL Editor create a query called field_inventory. Select the Free Edition warehouse and run this query against the original archive.

```sql
SELECT year, COUNT(*) AS observations,
       COUNT(TRY_CAST(body_mass_g AS INT)) AS weighed
FROM read_files(
  '/Volumes/workspace/palmer_lab/landing/penguins.csv',
  format => 'csv', header => true)
GROUP BY year ORDER BY year;
```

| Survey year | Observations | Recorded weights |
| --- | --- | --- |
| 2007 | 110 | 109 |
| 2008 | 114 | 114 |
| 2009 | 120 | 119 |

Which entries explain the difference? This query shows the two rows where the weight field contains NA.

```sql
SELECT species, island, year, body_mass_g
FROM read_files(
  '/Volumes/workspace/palmer_lab/landing/penguins.csv',
  format => 'csv', header => true)
WHERE TRY_CAST(body_mass_g AS INT) IS NULL ORDER BY year;
```

[Open the equivalent Genie route](#genie-lab-03)

**Checkpoint**

One Adelie record on Torgersen in 2007 and one Gentoo record on Biscoe in 2009 lack a weight. The archive contains 344 observations and 342 weights.

#### Why does COUNT give two answers?

The letters NA occupy a cell in the CSV. Counting that text would count it as a weight. TRY_CAST changes a value like 3750 into a number and gives NULL for NA. COUNT(number) skips that missing value. COUNT(*) still counts the observation.

**Sources** · [read_files reference](https://docs.databricks.com/aws/en/sql/language-manual/functions/read_files) · [Publisher’s CSV](https://raw.githubusercontent.com/allisonhorst/palmerpenguins/main/inst/extdata/penguins.csv) · [Genie Code](https://docs.databricks.com/aws/en/genie-code/)

[Contents](../#contents) · [Field notes](#field-notes-03)

## Manual lab 04

### Load the first file into Bronze

*LAKEFLOW CONNECT*

In lab 03 a one-off SQL query read the CSV. Running that query again won’t tell you which deliveries you’ve already loaded. This time let Auto Loader keep track of incoming files and save their rows in Bronze. Keep the original text and filename so you can trace an odd value back to the file that supplied it.

#### Choose SQL for this transformation

**Workspace and commands**

New • ETL pipeline. Name it palmer_etl and set the destination to workspace.palmer_lab. Choose SQL if asked for a language.

In the pipeline editor, check the active tab. If it ends in .py, use the pipeline file browser’s + • Transformation. Select SQL, name it bronze and click Create. Paste the block below into bronze.sql or the SQL starter tab.

Already have a working Python definition of bronze_obs? Keep it and skip to the row-count query below.

```sql
CREATE OR REFRESH STREAMING TABLE bronze_obs AS
SELECT species, island, year, bill_length_mm, bill_depth_mm,
       flipper_length_mm, body_mass_g, sex,
       _metadata.file_name AS source_file
FROM STREAM read_files(
  '/Volumes/workspace/palmer_lab/landing/inbox',
  format => 'csv', header => true,
  schema => 'species STRING, island STRING, bill_length_mm STRING,
             bill_depth_mm STRING, flipper_length_mm STRING,
             body_mass_g STRING, sex STRING, year STRING'
);
```

Click Run pipeline. After it succeeds, run this query in SQL Editor. Run the pipeline once more with the same inbox and check again.

```sql
SELECT source_file, COUNT(*) AS rows
FROM workspace.palmer_lab.bronze_obs GROUP BY source_file;
```

[Open the equivalent Genie route](#genie-lab-04)

**Checkpoint**

Both updates give first.csv and 224 rows. A second update does not duplicate the first delivery.

**Gotcha**

PYTHON.SYNTAX_ERROR at CREATE usually means SQL was pasted into a .py file. Move that SQL into a SQL transformation and clear the misplaced block from the Python file. Keep a single definition of bronze_obs.

#### Does “streaming table” mean it runs all night?

This pipeline uses triggered updates. When an update runs, Auto Loader picks up the files it has yet to process. Between updates the table stays available to query. In lab 06 a separate file-arrival trigger tells the job when to start.

**Sources** · [Transformation files](https://docs.databricks.com/aws/en/ldp/multi-file-editor) · [read_files reference](https://docs.databricks.com/aws/en/sql/language-manual/functions/read_files) · [Pipeline editor tutorial](https://docs.databricks.com/aws/en/ldp/tutorial-get-started) · [Genie in pipelines](https://docs.databricks.com/aws/en/ldp/de-agent)

[Contents](../#contents) · [Field notes](#field-notes-04)

## Manual lab 05

### Keep each observation, then count by group

*DECLARATIVE PIPELINES*

In 2007 the team recorded 20 Adelie penguins on Torgersen, but only 19 weights. Dropping the incomplete row loses a sighting. Treating its NA as zero invents a weight. Silver keeps the sighting and leaves its numeric weight empty. Gold turns those rows into a report with both counts. Keep Silver too: the two totals alone can’t tell you which entry needs checking.

**Workspace and commands**

In palmer_etl choose + • Transformation. Select SQL, name it clean and click Create. Paste both definitions into clean.sql and click Run pipeline. Bronze can stay in its original Python or SQL file.

```sql
CREATE OR REFRESH MATERIALIZED VIEW silver_obs (
  CONSTRAINT mass_present EXPECT (mass_g IS NOT NULL AND mass_g > 0)
) AS SELECT species, island, TRY_CAST(year AS INT) AS yr,
       TRY_CAST(body_mass_g AS INT) AS mass_g,
       TRY_CAST(bill_length_mm AS DOUBLE) AS bill_mm,
       TRY_CAST(flipper_length_mm AS INT) AS flipper_mm,
       source_file
FROM bronze_obs;

CREATE OR REFRESH MATERIALIZED VIEW gold_counts AS
SELECT yr, island, species, COUNT(*) AS obs_n,
       COUNT(mass_g) AS weighed_n,
       COUNT(*) - COUNT(mass_g) AS missing_n,
       SUM(mass_g) AS total_mass_g
FROM silver_obs GROUP BY yr, island, species;
```

Select silver_obs in the graph. The weight check flags the row with NA and keeps it in the survey. If expectations are unavailable, remove the parenthesized CONSTRAINT clause and keep silver_obs AS SELECT. Gold still counts missing values. Check the first delivery in SQL Editor.

```sql
SELECT SUM(obs_n), SUM(weighed_n), SUM(missing_n)
FROM workspace.palmer_lab.gold_counts;
```

[Open the equivalent Genie route](#genie-lab-05)

**Checkpoint**

First delivery: 224 observations, 223 weights, one missing weight.

**The more you know**

Read the two COUNT expressions side by side. COUNT(*) counts every survey row. COUNT(mass_g) skips a row when its weight is null. Their difference tells you how many weights are missing.

#### Should a failed weight check remove the row?

Here the expectation records a warning. The team really observed the bird, and losing its row would make the survey look more complete than it was. Drop or fail rules would change what happens when the same check finds a missing weight.

**Sources** · [Pipeline editor tutorial](https://docs.databricks.com/aws/en/ldp/tutorial-get-started) · [Expectations](https://docs.databricks.com/aws/en/ldp/expectations) · [Pipeline editions](https://docs.databricks.com/aws/en/ldp/configure-pipeline) · [Genie in pipelines](https://docs.databricks.com/aws/en/ldp/de-agent)

[Contents](../#contents) · [Field notes](#field-notes-05)

## Genie lab 01

### Create your work area

*GENIE CODE · SQL EDITOR*

Tomorrow a job will need to read the file you save today, after your notebook session has ended. Give both a permanent address. Use workspace as the catalog, palmer_lab as the schema for the project’s tables, and a volume called landing for its files. A path inside that volume works in both the notebook and the job.

Start here with a signed-in Free Edition workspace and its existing serverless SQL warehouse. No tutorial objects or previous chat are required.

#### Open

Sign in, open SQL Editor, start a new query, select the warehouse and open Genie Code beside the query. Send the prompt below. If Genie offers SQL without executing it, insert the statements into the query and click Run.

**Paste into Genie**

```text
Check my current user, catalog and schema. Use the existing writable catalog workspace. Create the schema workspace.palmer_lab if it is missing. Create the managed volume workspace.palmer_lab.landing if it is missing. List the volumes in workspace.palmer_lab and return the full /Volumes path for landing. Preserve any existing objects.
```

**Checkpoint**

The returned volume is landing; its path is /Volumes/workspace/palmer_lab/landing. Catalog • workspace • palmer_lab • landing opens it. Seeing default as your session schema is fine.

**The more you know**

Your earlier Genie response already confirms this checkpoint. You can go straight to B02 in [Genie lab 02](#genie-lab-02).

#### Where will I find the data tomorrow?

Open Catalog and look under workspace • palmer_lab. The landing volume holds the CSV files. Later, the survey tables will appear alongside the volume. You can close the notebook and reopen it later. The saved files and tables will still be there.

**Sources** · [Free Edition overview](https://docs.databricks.com/aws/en/getting-started/free-edition) · [First table and privileges](https://docs.databricks.com/aws/en/getting-started/create-table) · [Work with volume files](https://docs.databricks.com/aws/en/volumes/volume-files) · [Genie Code](https://docs.databricks.com/aws/en/genie-code/)

[Contents](../#contents) · [Field notes](#field-notes-01)

## Genie lab 02

### Split the survey into two deliveries

*GENIE CODE · NOTEBOOK*

The publisher gives you all three seasons in one CSV. You need a second delivery to find out whether a new file can start a job. Keep the original for comparison, put 2007–08 in the inbox now and hold back 2009. Using real survey years gives each delivery a count you can check.

New • Notebook: create a blank Python notebook named stage_deliveries, select serverless compute and use the managed volume created in B01. A cell is one executable block; + Genie Code generates a block for you to inspect and run.

#### Cell 1 • Fetch and verify the archive

At the bottom of the blank cell, hover and choose + Genie Code. Paste the first prompt, including its URL and digest. The digest is a fingerprint of the publisher’s file. Accept the generated cell, inspect it, then run it.

**Paste into Genie**

```text
Generate one Python notebook cell. The destination is /Volumes/workspace/palmer_lab/landing/penguins.csv. If it exists, read its bytes. Otherwise fetch the exact URL below with urllib.request.urlopen and a 30-second timeout. Normalize CRLF to LF and ensure exactly one trailing LF. Compute SHA-256 and compare with the exact digest below. Only write a new file after the check passes. Print the path and number of data rows. On a fetch or checksum failure, stop; never fabricate or substitute data. Generate the cell without running it.
URL: https://raw.githubusercontent.com/allisonhorst/palmerpenguins/main/inst/extdata/penguins.csv
SHA-256: f204db2c753b0937caac3cb35258562c14f073e4bbc76be24b4c51ce22767a93
```

#### Cell 2 • Split into two deliveries

Keep the original for checking. The pipeline watches inbox only, so a file in hold can be ready without being counted yet. After cell 1 reports 344 records, add another cell with + Genie Code. Accept and run the second prompt’s code.

**Paste into Genie**

```text
Generate one independent Python cell. With pandas, read /Volumes/workspace/palmer_lab/landing/penguins.csv as strings, preserving literal NA. Write rows with year 2007 or 2008 to /Volumes/workspace/palmer_lab/landing/inbox/first.csv and 2009 to /Volumes/workspace/palmer_lab/landing/hold/last.csv. Create the subdirectories. Include the original header, column order and row order; omit the pandas index. Preserve existing output files. Read back the two saved files and print their actual row counts. Leave last.csv in hold. Generate the cell without running it.
```

**Checkpoint**

inbox/first.csv has 224 data rows. hold/last.csv has 120. Do not release the late batch yet. If the fetch is blocked, use the B02 offline prompt in [Genie offline archive](../appendix/#genie-offline-archive), then run the second generated cell.

**Sources** · [Generate notebook cells](https://docs.databricks.com/aws/en/notebooks/basic-editing) · [Publisher’s CSV](https://raw.githubusercontent.com/allisonhorst/palmerpenguins/main/inst/extdata/penguins.csv) · [CC0 licence](https://allisonhorst.github.io/palmerpenguins/LICENSE.html) · [Work with volume files](https://docs.databricks.com/aws/en/volumes/volume-files) · [Edition limits](https://docs.databricks.com/aws/en/getting-started/free-edition-limitations)

[Contents](../#contents) · [Field notes](#field-notes-02)

## Genie lab 03

### Inspect the source archive

*GENIE CODE · SQL EDITOR*

If the dashboard later says 342 penguins were weighed, how will you know it counted correctly? Get an answer directly from the original CSV before writing the pipeline. Count every observation, then count the weights that really are numbers. Break the totals down by year so you can check each delivery separately.

Start with B02’s verified volume file. This check uses the full archive even though only the first delivery has been released.

#### Open and ask

In SQL Editor open a new query, choose the existing SQL warehouse and name the query field_inventory. Open Genie Code and send this prompt. Apply the SQL and run both result sets.

**Paste into Genie**

```text
Write two Databricks SQL queries using read_files on '/Volumes/workspace/palmer_lab/landing/penguins.csv' with CSV format and header=true. First, group by year and report COUNT(*) as observations and COUNT(TRY_CAST(body_mass_g AS INT)) as weighed, ordered by year. Second, return species, island, year and the source body_mass_g for rows where the numeric mass is null, ordered by year. Use the root archive, not the inbox files.
```

**Checkpoint**

The result by year is 2007: 110 / 109, 2008: 114 / 114, 2009: 120 / 119. Missing weights belong to an Adelie record from Torgersen in 2007 and a Gentoo record from Biscoe in 2009.

**Gotcha**

An expression that counts the literal string NA can report 344 weights. Check the generated query for a numeric conversion before COUNT.

#### Why does COUNT give two answers?

The letters NA occupy a cell in the CSV. Counting that text would count it as a weight. TRY_CAST changes a value like 3750 into a number and gives NULL for NA. COUNT(number) skips that missing value. COUNT(*) still counts the observation.

**Sources** · [read_files reference](https://docs.databricks.com/aws/en/sql/language-manual/functions/read_files) · [Publisher’s CSV](https://raw.githubusercontent.com/allisonhorst/palmerpenguins/main/inst/extdata/penguins.csv) · [Genie Code](https://docs.databricks.com/aws/en/genie-code/)

[Contents](../#contents) · [Field notes](#field-notes-03)

## Genie lab 04

### Load the first file into Bronze

*GENIE CODE · PIPELINE EDITOR*

In lab 03 a one-off SQL query read the CSV. Running that query again won’t tell you which deliveries you’ve already loaded. This time let Auto Loader keep track of incoming files and save their rows in Bronze. Keep the original text and filename so you can trace an odd value back to the file that supplied it.

#### Match the code to the open file

Use the inbox from B02. Open palmer_etl or choose New • ETL pipeline to create it. Set the destination to workspace.palmer_lab. Open Genie Code beside the transformation. A .py tab takes Python and a .sql tab takes SQL.

If Genie already repaired bronze_obs, go straight to “Run and check”. Otherwise send this prompt.

**Paste into Genie**

```text
Define bronze_obs once in this pipeline. Use valid pipeline Python for a .py file or pipeline SQL for a .sql file. Use Auto Loader to stream CSVs with headers from /Volumes/workspace/palmer_lab/landing/inbox. Read these columns as strings in order: species, island, bill_length_mm, bill_depth_mm, flipper_length_mm, body_mass_g, sex, year. Keep all eight and add _metadata.file_name as source_file. Reuse any existing bronze_obs definition. Apply the edit.
```

#### Run and check

Click Run pipeline. After it succeeds, open Genie Code in SQL Editor. Send the prompt below and run its query. Repeat the pipeline run with the same inbox and check again.

**Paste into Genie**

```text
Write a query counting rows by source_file in workspace.palmer_lab.bronze_obs. Show the filename and count.
```

**Checkpoint**

Both runs return first.csv with 224 rows. The notebook’s held file remains outside the watched directory.

#### Does “streaming table” mean it runs all night?

This pipeline uses triggered updates. When an update runs, Auto Loader picks up the files it has yet to process. Between updates the table stays available to query. In lab 06 a separate file-arrival trigger tells the job when to start.

**Sources** · [Transformation files](https://docs.databricks.com/aws/en/ldp/multi-file-editor) · [read_files reference](https://docs.databricks.com/aws/en/sql/language-manual/functions/read_files) · [Pipeline editor tutorial](https://docs.databricks.com/aws/en/ldp/tutorial-get-started) · [Genie in pipelines](https://docs.databricks.com/aws/en/ldp/de-agent)

[Contents](../#contents) · [Field notes](#field-notes-04)

## Genie lab 05

### Keep each observation, then count by group

*GENIE CODE · PIPELINE EDITOR*

In 2007 the team recorded 20 Adelie penguins on Torgersen, but only 19 weights. Dropping the incomplete row loses a sighting. Treating its NA as zero invents a weight. Silver keeps the sighting and leaves its numeric weight empty. Gold turns those rows into a report with both counts. Keep Silver too: the two totals alone can’t tell you which entry needs checking.

Open palmer_etl from B04. Keep the working Bronze file, including a Python version repaired by Genie. In the pipeline editor, open Genie Code and send this prompt.

**Paste into Genie**

```text
Create a separate SQL transformation named clean.sql. Leave the existing bronze_obs definition in its own file. Create a materialized view silver_obs over bronze_obs with species, island, TRY_CAST(year AS INT) AS yr, TRY_CAST(body_mass_g AS INT) AS mass_g, TRY_CAST(bill_length_mm AS DOUBLE) AS bill_mm, TRY_CAST(flipper_length_mm AS INT) AS flipper_mm, and source_file. Keep every observation. Monitor mass_g IS NOT NULL AND mass_g > 0 with a warning-only expectation named mass_present. Create a materialized view gold_counts over silver_obs grouped by yr, island, species. Publish COUNT(*) AS obs_n, COUNT(mass_g) AS weighed_n, their difference AS missing_n and SUM(mass_g) AS total_mass_g.
```

#### Check the totals

Check that the SQL keeps a penguin’s row when its weight is missing. Click Run pipeline. Select silver_obs in the graph to inspect quality metrics. If expectations are unavailable, ask Genie Code to remove only the expectation and keep all the rows and both counts; run again. Open SQL Editor with Genie Code for the check.

**Paste into Genie**

```text
Return SUM(obs_n), SUM(weighed_n) and SUM(missing_n) across workspace.palmer_lab.gold_counts.
```

**Checkpoint**

The graph shows bronze_obs feeding silver_obs and then gold_counts. The first delivery totals 224 / 223 / 1.

#### Should a failed weight check remove the row?

Here the expectation records a warning. The team really observed the bird, and losing its row would make the survey look more complete than it was. Drop or fail rules would change what happens when the same check finds a missing weight.

**Sources** · [Pipeline editor tutorial](https://docs.databricks.com/aws/en/ldp/tutorial-get-started) · [Expectations](https://docs.databricks.com/aws/en/ldp/expectations) · [Pipeline editions](https://docs.databricks.com/aws/en/ldp/configure-pipeline) · [Genie in pipelines](https://docs.databricks.com/aws/en/ldp/de-agent)

[Contents](../#contents) · [Field notes](#field-notes-05)

## Field notes 01

### Where does my work actually live?

Close the browser after lab 01 and reopen it tomorrow. The notebook, the CSV you will save and the table you will query live in three different places. Knowing which one to look for makes the workspace much easier to use.

#### The workspace and the catalog

Workspace is where you edit and organize things such as notebooks. Catalog Explorer is where you find governed data. In this account, workspace is also the name of a catalog. Inside that catalog, palmer_lab is a schema, a named collection of data objects. Its landing volume gives ordinary files a governed home. A table in the same schema gives SQL users named columns and rows.

#### What serverless changes

Compute is the service that runs your code. The SQL warehouse executes a query. Notebook serverless compute can execute a Python cell. Both can reach saved data when the identity using them has access. Stopping compute does not erase the CSV. A Python variable, on the other hand, belongs to a running session.

#### An experienced practitioner checks the resolved name

The metastore is the top-level registry that holds your catalogs. An unqualified name such as taxon is interpreted relative to the active catalog and schema. Two colleagues can paste the same SQL and reach different tables. Use workspace.palmer_lab.taxon when a query will be saved, scheduled or handed to someone else. The full name identifies the object independently of the session defaults.

**The more you know**

Tomorrow the notebook opens, but a Python variable named root is undefined. Rerun the cell that defines root. Then look for the saved file in the volume. Recreating the volume would solve a different problem.

#### Check your understanding

A scheduled query worked in your notebook yesterday but now reports that taxon cannot be found. What would you check before creating a new table?

[Check your answer for lab 01](../appendix/#answers-for-labs-01-to-05)

**Sources** · [Work with volume files](https://docs.databricks.com/aws/en/volumes/volume-files) · [Workspace files](https://docs.databricks.com/aws/en/files/workspace-basics) · [Choose notebook compute](https://docs.databricks.com/aws/en/compute/serverless/notebooks) · [First table and privileges](https://docs.databricks.com/aws/en/getting-started/create-table)

[Contents](../#contents) · [Manual lab 01](#manual-lab-01) · [Genie lab 01](#genie-lab-01)

## Field notes 02

### Can another run reproduce the delivery?

Two notebooks can print “224 rows” and still have produced different files. The starting archive, the split rule and what happens on a second run all matter.

#### What a notebook remembers

A notebook stores the cells you wrote. While a session is running, Python also remembers variables and imports from cells you already executed. A job starts its own run. Put the imports, paths and settings the job needs in its saved code. A notebook that only works after you click its cells in a lucky order is difficult to hand over.

#### Preserve an input you can identify

The publisher’s URL points to a file on a branch that can change. The fingerprint in the notebook identifies the contents this guide was tested with. The cell normalizes line endings before it computes that fingerprint. A matching digest lets you start from the same text when the download works and when you use the offline copy.

#### A retry should have a predictable effect

Keeping 2009 in hold makes the late delivery deliberate. Keeping the archive outside inbox prevents the pipeline from ingesting the whole survey along with the split files. The staging cell skips an output path that already exists. That protects a correct delivery from being overwritten, but it also preserves an incomplete file left by an earlier attempt.

**The more you know**

Suppose first.csv already exists with only its header. The staging cell can still print 224 because it reports the size of the batch in memory. Count the records in the saved file before trusting that message. The source archive should remain available for comparison.

#### Check your understanding

A colleague asks whether matching SHA-256 fingerprints prove that all the weighing scales were calibrated. What can the fingerprint establish?

[Check your answer for lab 02](../appendix/#answers-for-labs-01-to-05)

**Sources** · [Dataset and original studies](https://github.com/allisonhorst/palmerpenguins) · [Work with volume files](https://docs.databricks.com/aws/en/volumes/volume-files) · [Generate notebook cells](https://docs.databricks.com/aws/en/notebooks/basic-editing) · [Choose notebook compute](https://docs.databricks.com/aws/en/compute/serverless/notebooks)

[Contents](../#contents) · [Manual lab 02](#manual-lab-02) · [Genie lab 02](#genie-lab-02)

## Field notes 03

### What exactly did you count?

A field worker saw the penguin on Torgersen and wrote down its species. The weight cell contains NA. The right count depends on whether the question is about the sighting or the measurement.

#### Give a row a meaning

One row represents one survey observation. A spreadsheet row, an observed bird and a unique individual are different counting units. This file has enough information to count observations by island. It has no permanent bird identifier that would let you prove how many distinct individuals returned in a later season.

#### Why the type matters

In a text field, NA is a value someone entered. Turning the weight into an integer with TRY_CAST gives SQL a number when conversion succeeds and NULL when a string such as NA cannot be converted. COUNT(*) counts the observation. COUNT of that numeric expression counts the rows with usable weights.

#### A tolerant conversion can hide a new problem

If a future file contains 37OO with two letter O’s, the same conversion also yields NULL. A missing measurement and a typing error now look alike in the numeric column. Group the original text values for rows where conversion returned NULL. That tells you which entries follow the known missing-value convention and which deserve inspection.

**The more you know**

If someone filled both missing weights with zero, the observation count would stay at 344, the recorded-weight count would rise to 344, and the mean would fall. An unchanged row count is a weak test of measurement quality.

#### Check your understanding

If a technician accidentally enters a weight of −1, will COUNT(TRY_CAST(body_mass_g AS INT)) include it? Which check would detect it?

[Check your answer for lab 03](../appendix/#answers-for-labs-01-to-05)

**Sources** · [read_files reference](https://docs.databricks.com/aws/en/sql/language-manual/functions/read_files) · [TRY_CAST](https://docs.databricks.com/aws/en/sql/language-manual/functions/try_cast) · [Dataset and original studies](https://github.com/allisonhorst/palmerpenguins)

[Contents](../#contents) · [Manual lab 03](#manual-lab-03) · [Genie lab 03](#genie-lab-03)

## Field notes 04

### How does ingestion know what is new?

The inbox still contains first.csv after a successful run. If a later run opens the same folder, why doesn’t it add the same 224 rows again?

#### The file and the processing record

Auto Loader discovers incoming files. It keeps a checkpoint, a durable record of progress that allows the stream to resume. In this lab the pipeline manages that state and writes Bronze as a streaming table. You can query Bronze between runs. The word “streaming” describes how its input is processed, even though you start the pipeline in separate updates.

#### A new file and a new observation are different things

Suppose the station sends the identical first delivery under a different filename. Auto Loader can process it as a new file. Its record of file progress cannot tell whether two rows describe the same physical observation. Removing duplicate observations safely would require an observation identifier or an explicit matching rule agreed with the field team.

#### Two useful clues in Bronze

The checkpoint tracks what the reader processed. The Delta transaction log tracks what the table committed. A Delta version number counts table commits, not input files. The raw measurement text tells you what the file actually said. The recorded source filename tells you which delivery supplied it. Group Bronze rows by source_file before looking for a problem in the aggregation. Two batches of 224 under different names lead to a different investigation from one 224-row batch followed by a bad join.

**The more you know**

A normal update with no new files should leave the same 224 rows after the first delivery. A full refresh can rebuild from available inputs. Keeping the source files matters if you ever need to replay the pipeline.

#### Check your understanding

Two different filenames contain the same observation. Can a successful incremental pipeline update tell you which row to keep?

[Check your answer for lab 04](../appendix/#answers-for-labs-01-to-05)

**Sources** · [Auto Loader checkpoints](https://docs.databricks.com/aws/en/ingestion/cloud-object-storage/auto-loader/) · [read_files reference](https://docs.databricks.com/aws/en/sql/language-manual/functions/read_files) · [Pipeline editor tutorial](https://docs.databricks.com/aws/en/ldp/tutorial-get-started) · [Pipeline refresh behavior](https://docs.databricks.com/aws/en/ldp/concepts/refresh)

[Contents](../#contents) · [Manual lab 04](#manual-lab-04) · [Genie lab 04](#genie-lab-04)

## Field notes 05

### What changes between Bronze, Silver and Gold?

Ask two colleagues to find the unweighed bird and to prepare a weekly count. The first needs the observation itself. The second needs a summary. One table can be a poor fit for both jobs.

#### Decide what one row stands for

Bronze preserves what arrived. Silver still has one row per observation, now with a numeric mass and a year you can filter. In Gold, one row stands for a combination of year, island and species. This level of detail is called the grain. Adding a column to GROUP BY changes the grain and can change every downstream calculation.

#### What declarative means

The pipeline definitions describe the data you want in each table. The references from Gold to Silver and Silver to Bronze tell the pipeline how the tables depend on each other. A materialized view stores a query result and refreshes it as part of an update. The engine can choose an incremental refresh or a full recomputation. Seeing the right result does not tell you which refresh strategy it used.

#### Decide what should happen to a bad measurement

A warning lets the team count an incomplete observation and inspect it later. A drop rule would remove it from the result. Failing the update would hold back an entire delivery. Choose the response based on what the report promises. Also check positivity separately from presence: COUNT(mass_g) includes zero because zero is a number. Only the validation rule flags it.

**The more you know**

In the first delivery, 224 observed must equal 223 weighed plus one missing. That identity catches broken arithmetic. It can still hold after a duplicate delivery, so also compare the total with the independent count of the original file.

#### Check your understanding

Gold tells you a weight is missing on Biscoe. Can Gold alone recover the original text the worker entered?

[Check your answer for lab 05](../appendix/#answers-for-labs-01-to-05)

**Sources** · [Pipeline refresh behavior](https://docs.databricks.com/aws/en/ldp/concepts/refresh) · [Expectations](https://docs.databricks.com/aws/en/ldp/expectations) · [Pipeline editor tutorial](https://docs.databricks.com/aws/en/ldp/tutorial-get-started)

[Contents](../#contents) · [Manual lab 05](#manual-lab-05) · [Genie lab 05](#genie-lab-05)

---

[Tutorial home](../) · [Labs 06–10](../labs-06-10/) →

---

## About the author

Mohammed Brückner is an enterprise architect in Munich, building production machine learning platforms on Azure and Databricks. More at [mohammed-brueckner.com](https://mohammed-brueckner.com) — and for talks or workshops on data platforms and enterprise AI, see [Speaking & Workshops](https://mohammed-brueckner.com/speaking/).

Books: [*Machine Learning Operations (MLOps) with Databricks on Azure End-to-End*](https://www.amazon.com/dp/B0FTSY78DR) — the production-scale continuation of this tutorial — and [Platform Economies](https://platformeconomies.com) (2026), on what happens when the API becomes the product.
