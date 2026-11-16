---
permalink: /databricks/free-edition-tutorial/labs-06-10/
title: "Databricks Labs 6-10: File Arrival Triggers, Job Repair, Dashboards, Lineage"
description: "Let a late file trigger the pipeline, diagnose and repair a failed job run, rebuild the summary on a dashboard canvas, join taxonomic names, and inspect access and lineage in Unity Catalog — manual and Genie routes for every lab."
---

# Labs 06–10: Triggers, Failures, Dashboards and Lineage

*Part of the [Databricks Free Edition tutorial](../) with Palmer Penguins — [all labs and routes](../#contents).*

## Manual lab 06

### Let the late file start the job

*LAKEFLOW JOBS*

The 2009 file can arrive while nobody is looking at the inbox. A file-arrival trigger starts the job when there’s work to do. Have it refresh the tables before it checks their counts. If the check ran first, yesterday’s valid totals could pass while the new file was still waiting.

#### Save the Python check in its own notebook

**Workspace and commands**

New • Notebook. Name it verify_delivery and choose Python. Paste the cell below and save. The job will run it. For an optional test here, choose Serverless in the notebook’s compute menu. If it shows a SQL warehouse, switch to Serverless first. spark.sql(...) is a Python call, even though it contains a SQL query.

```python
counts = spark.sql("""
    SELECT SUM(obs_n), SUM(weighed_n), SUM(missing_n)
    FROM workspace.palmer_lab.gold_counts
""").first()
actual = tuple(int(n) for n in counts)
assert actual in {(224, 223, 1), (344, 342, 2)}, actual
print(actual)
```

#### Run the saved notebook as a job task

Jobs & Pipelines • Create job • palmer_delivery. Add a Pipeline task named refresh for palmer_etl, with full refresh off. Add a second task named verify. Set Type to Notebook, Source to Workspace, Path to verify_delivery, Compute to Serverless and Depends on to refresh. Save the task.

Click Run now. Open Runs • latest run • verify. Before you release 2009, the notebook output should be (224, 223, 1).

In Job details • Add trigger • File arrival, enter /Volumes/workspace/palmer_lab/landing/inbox. Test the connection and save. Once the trigger is active, run this cell in stage_deliveries on notebook Serverless compute to release 2009.

```python
from pathlib import Path
import shutil
root = Path("/Volumes/workspace/palmer_lab/landing")
late = root / "inbox" / "last.csv"
if not late.exists():
    shutil.copyfile(root / "hold" / "last.csv", late)
```

[Open the equivalent Genie route](#genie-lab-06)

**Checkpoint**

Runs shows a file-triggered run. Its verify output is (344, 342, 2). Click Run now again and confirm the same result. Pause the trigger.

**Gotcha**

If the trigger is delayed, check its active status, exact path and the new filename. A manual run verifies the tasks; only the file-triggered run verifies the trigger.

#### What if the task succeeded but I can’t see its printout?

The check accepts both deliveries, so either set of counts passes. Look at the run trigger for FILE_ARRIVAL. Then query Gold for 344 observed and 342 weighed, and Bronze for 120 rows from last.csv. Together those checks show that the second delivery landed.

**Sources** · [Configure a notebook task](https://docs.databricks.com/aws/en/jobs/tasks/notebook) · [Choose notebook compute](https://docs.databricks.com/aws/en/compute/serverless/notebooks) · [File arrival triggers](https://docs.databricks.com/aws/en/jobs/file-arrival-triggers)

[Contents](../#contents) · [Field notes](#field-notes-06)

## Manual lab 07

### Find out why the job failed

*OPERATIONS*

A job can fail even when the field data is right. You’ll change the expected number of weights to 343 and see the check fail. Use the two missing weights in the original records to decide whether to repair the test or investigate ingestion. Changing the data just to pass the test would put a made-up measurement in the report.

**Workspace and commands**

With both deliveries present and the trigger paused, replace only the assert in verify_delivery with the following line. Save and run palmer_delivery.

```python
assert actual == (344, 343, 1), actual
```

Open Jobs & Pipelines • palmer_delivery • Runs • failed run • verify. The refresh task succeeded. Read the three numbers in the error message. Then run this query to see which two survey entries have no weight.

```sql
SELECT yr, island, species, source_file
FROM workspace.palmer_lab.silver_obs
WHERE mass_g IS NULL ORDER BY yr;
```

Restore the original assert from lab 06, save the notebook and open the failed run. Select Repair run and choose verify. Inspect the repaired task’s output.

[Open the equivalent Genie route](#genie-lab-07)

**Checkpoint**

The failed task prints (344, 342, 2) in its exception. The repaired verify task succeeds on the same data. The run detail records the failure and repair.

**Gotcha**

The source file has two missing weights. Fix the number the test expects. Filling in either weight would change what the field team recorded.

#### Do I need to process the files again?

Look at the two task results. If refresh succeeded and only verify failed, repair verify after correcting its expected count. Starting the whole job again would repeat a step that already succeeded.

**Sources** · [Repair failed runs](https://docs.databricks.com/aws/en/jobs/repair-job-failures) · [Jobs quickstart](https://docs.databricks.com/aws/en/jobs/jobs-quickstart) · [ZeroOps requirements](https://docs.databricks.com/aws/en/zeroops/get-started)

[Contents](../#contents) · [Field notes](#field-notes-07)

## Manual lab 08

### Rebuild the summary on canvas

*LAKEFLOW DESIGNER*

Putting the Gold totals on two screens would repeat any mistake in the Gold query. Instead, start from the individual Silver rows and count them by island using the Designer canvas. If both methods give 168 observations and 167 weights for Biscoe, you’ve checked how the totals were built.

**Workspace and commands**

New • Visual data prep. Name it island_check. Choose Browse all data and workspace.palmer_lab.silver_obs. Add Aggregate from the Source operator’s + control. Group by island. Add COUNT on species as observations and COUNT on mass_g as weighed. Apply and choose Run all. Select Aggregate to inspect its rows.

| Island | Observations | Weighed | Missing |
| --- | --- | --- | --- |
| Biscoe | 168 | 167 | 1 |
| Dream | 124 | 124 | 0 |
| Torgersen | 52 | 51 | 1 |

Compare the first two counts with Gold. The missing count equals their difference.

```sql
SELECT island, SUM(obs_n) AS observations,
       SUM(weighed_n) AS weighed, SUM(missing_n) AS missing
FROM workspace.palmer_lab.gold_counts
GROUP BY island ORDER BY island;
```

[Open the equivalent Genie route](#genie-lab-08)

**Checkpoint**

The canvas and Gold agree for all three islands. Save the visual prep.

**The more you know**

A preview can sample the input. Choose Run all before comparing counts. Every row here has a species.

#### Why build the same summary a second time?

Designer starts with the individual Silver rows. Counting them here gives you a way to catch a mistake in the Gold query. If both screens used the same Gold result, agreement would tell you much less. Run the full input before comparing.

**Sources** · [Designer tutorial](https://docs.databricks.com/aws/en/designer/build-transformation) · [Genie on canvas](https://docs.databricks.com/aws/en/designer/genie-code) · [Designer availability](https://www.databricks.com/blog/whats-new-lakeflow-designer-october-2026-roundup)

[Contents](../#contents) · [Field notes](#field-notes-08)

## Manual lab 09

### Match the two names for each species

*SPECIES LOOKUP*

A biologist asks for Pygoscelis adeliae, while the field sheets say Adelie. Keep those names together in one reference table so a correction can reach every report that uses it. Join each observation to its species once. A left join keeps an unfamiliar label visible as a missing match, so you can correct the lookup without losing the observation.

**Workspace and commands**

In SQL Editor, create the reference table directly from the three ITIS names below. Then build a view that attaches a scientific name to each observation.

```sql
CREATE OR REPLACE TABLE workspace.palmer_lab.taxon
USING DELTA AS SELECT * FROM VALUES
  ('Adelie',    'Pygoscelis adeliae'),
  ('Chinstrap', 'Pygoscelis antarcticus'),
  ('Gentoo',    'Pygoscelis papua')
AS t(species, scientific_name);

CREATE OR REPLACE VIEW workspace.palmer_lab.field_obs
COMMENT 'One Palmer survey observation, including unweighed birds.'
AS SELECT s.*, t.scientific_name
FROM workspace.palmer_lab.silver_obs s
LEFT JOIN workspace.palmer_lab.taxon t ON s.species = t.species;

SELECT COUNT(*) AS joined, COUNT(scientific_name) AS mapped,
       COUNT(mass_g) AS weighed FROM workspace.palmer_lab.field_obs;
```

[Open the equivalent Genie route](#genie-lab-09)

**Checkpoint**

The join yields 344 joined, 344 mapped and 342 weighed. An unmapped label reduces mapped. A duplicate reference key inflates joined.

#### Is this three-row lookup all Genie Ontology does?

Here you’ve built a SQL relationship. Genie Ontology has a wider job: it helps Genie interpret terms using definitions people publish and clues from queries, dashboards and agents. In lab 11 you publish a rule for “weighed”. If a popular query counts every observation, that query can be a clue, but the explicit metric definition carries more weight. For a Genie One answer with citations, open Genie Ontology under the response to inspect the sources it actually used. A new lookup may have no snippet to inspect yet.

**Sources** · [ITIS taxonomic record](https://www.itis.gov/servlet/SingleRpt/SingleRpt?search_topic=TSN&search_value=174444) · [Dataset and original studies](https://github.com/allisonhorst/palmerpenguins) · [Genie Ontology](https://docs.databricks.com/aws/en/genie/genie-ontology/) · [November release notes](https://docs.databricks.com/aws/en/release-notes/product/2026/november)

[Contents](../#contents) · [Field notes](#field-notes-09)

## Manual lab 10

### Inspect access and lineage

*UNITY CATALOG*

The colleague reviewing a species name needs to know its source and whether the lookup table changed. Lineage traces the name back to taxon. Table history shows writes to taxon. Opening either as the owner tells you little about the colleague’s access, so you’ll also check history under their sign-in.

**Workspace and commands**

In Catalog • workspace • palmer_lab • field_obs • Lineage, follow scientific_name back to taxon. Open taxon • History to see the latest write and taxon • Permissions to see the grants. Working alone? Record the latest operation as the owner. A reader test takes a second sign-in.

#### 1. Get the reader into this workspace and SQL Editor

The workspace admin checks Settings • Identity and access • Users • Manage. Assign the colleague to this workspace or add them if needed. They need Databricks SQL access. For the SQL test, a warehouse manager opens the existing SQL warehouse’s permissions and gives the user or their group Can use if missing.

#### 2. Give the reader access to the table and both parents

Use the colleague’s Databricks sign-in email in place of READER_EMAIL. Check each object’s Permissions first. An owner or someone who can grant privileges on that object runs only the missing grants. Group access counts. SELECT also allows the colleague to read the rows in taxon.

```sql
GRANT USE CATALOG ON CATALOG workspace TO `READER_EMAIL`;
GRANT USE SCHEMA ON SCHEMA workspace.palmer_lab TO `READER_EMAIL`;
GRANT SELECT ON TABLE workspace.palmer_lab.taxon TO `READER_EMAIL`;
```

#### 3. Have the reader test it

The colleague opens your workspace URL in their own session, selects the warehouse in SQL Editor and runs these separately. They can also open Catalog • workspace • palmer_lab • taxon • History.

```sql
SELECT session_user();
DESCRIBE HISTORY workspace.palmer_lab.taxon LIMIT 1;
```

[Open the equivalent Genie route](#genie-lab-10)

**Checkpoint**

The first result identifies the colleague. The second shows a version, timestamp and write operation. Seeing taxon in a search, or running the query as its owner, doesn’t establish that the colleague can read its history.

**Gotcha**

If SQL Editor won’t run a query, check SQL access and Can use. If it runs but names a denied catalog, schema or table, check the grant on that object. For temporary access, remove only the grants added for this test after the reader finishes.

#### Would the history show who read the table?

History lists writes, including creation or replacement. Lineage shows which data fed another table or view. A question about who opened or queried data belongs in audit records. The history and lineage checks here establish different facts.

**Sources** · [Workspace users](https://docs.databricks.com/aws/en/admin/users-groups/users) · [SQL warehouse access](https://docs.databricks.com/aws/en/admin/sql) · [Read access in Catalog](https://docs.databricks.com/aws/en/discover/database-objects) · [Delta history](https://docs.databricks.com/aws/en/tables/history) · [Unity Catalog lineage](https://docs.databricks.com/aws/en/data-governance/unity-catalog/data-lineage)

[Contents](../#contents) · [Field notes](#field-notes-10)

## Genie lab 06

### Trigger an update on arrival

*GENIE CODE · JOBS*

The 2009 file can arrive while nobody is looking at the inbox. A file-arrival trigger starts the job when there’s work to do. Have it refresh the tables before it checks their counts. If the check ran first, yesterday’s valid totals could pass while the new file was still waiting.

Start with the B05 pipeline and B02’s held file. First test the initial state; release 2009 only after the trigger is active.

#### Make a notebook to check the counts

Choose New • Notebook, name it verify_delivery and set the language to Python. In an empty cell select + Genie Code, paste the prompt, accept the cell and save. The job runs it. To test it interactively, open the notebook compute menu and choose Serverless. A SQL warehouse only runs SQL cells.

**Paste into Genie**

```text
Generate a Python cell that runs a Spark SQL query over workspace.palmer_lab.gold_counts, summing obs_n, weighed_n and missing_n in that order. Convert the result to a tuple of integers named actual. Assert that actual is either (224, 223, 1) or (344, 342, 2); include actual in the assertion error. Print actual.
```

#### Build and test the job

Open Jobs & Pipelines and Genie Code. Send the prompt. If Genie supplies a configuration plan, enter its two tasks via Create job, then Add trigger in Job details. Use Test connection and Save for the file trigger.

**Paste into Genie**

```text
Create or configure a job named palmer_delivery. Use a Pipeline task named refresh for existing palmer_etl with full refresh off. Add task verify with Type Notebook, Source Workspace, Path to the saved verify_delivery notebook, Compute Serverless and Depends on refresh. The notebook contains Python. If you need its path, ask me to select it. Set a file-arrival trigger on /Volumes/workspace/palmer_lab/landing/inbox. If job editing is unavailable here, give me the precise UI field values. Leave the held file untouched.
```

Click Run now. Open Runs • latest run • verify and look for (224, 223, 1). Once the file trigger is active, open stage_deliveries on notebook Serverless compute. Use + Genie Code for the next cell and run it once.

**Paste into Genie**

```text
Generate an independent Python cell using pathlib and shutil. Copy /Volumes/workspace/palmer_lab/landing/hold/last.csv to /Volumes/workspace/palmer_lab/landing/inbox/last.csv only if the destination does not exist. Print whether the new file was released.
```

**Checkpoint**

In Runs find a file-triggered run with verify = (344, 342, 2). A second Run now returns the same counts. Pause the trigger afterwards.

#### What if the task succeeded but I can’t see its printout?

The check accepts both deliveries, so either set of counts passes. Look at the run trigger for FILE_ARRIVAL. Then query Gold for 344 observed and 342 weighed, and Bronze for 120 rows from last.csv. Together those checks show that the second delivery landed.

**Sources** · [Configure a notebook task](https://docs.databricks.com/aws/en/jobs/tasks/notebook) · [Choose notebook compute](https://docs.databricks.com/aws/en/compute/serverless/notebooks) · [File arrival triggers](https://docs.databricks.com/aws/en/jobs/file-arrival-triggers) · [Generate notebook cells](https://docs.databricks.com/aws/en/notebooks/basic-editing)

[Contents](../#contents) · [Field notes](#field-notes-06)

## Genie lab 07

### Find out why the job failed

*GENIE CODE · JOB REPAIR*

A job can fail even when the field data is right. You’ll change the expected number of weights to 343 and see the check fail. Use the two missing weights in the original records to decide whether to repair the test or investigate ingestion. Changing the data just to pass the test would put a made-up measurement in the report.

Start with the completed B06 job, both deliveries present and its trigger paused. This deliberate failure changes only a test in your lab notebook.

#### Create the failure

Open verify_delivery, select its verification cell and ask Genie Code to edit it. Save the change. In palmer_delivery click Run now.

**Paste into Genie**

```text
In the selected cell keep the Gold query and the actual tuple. Change only the assertion to require actual == (344, 343, 1), reporting actual on failure. This intentionally creates a failing test for a repair exercise.
```

#### Diagnose from the data

In Jobs & Pipelines • palmer_delivery • Runs choose the failed run and open verify. Note the assertion output. In SQL Editor, ask Genie Code the next question and run its query.

**Paste into Genie**

```text
Show yr, island, species and source_file for each row of workspace.palmer_lab.silver_obs whose mass_g is NULL. Explain whether the observed Gold totals (344, 342, 2) or my expected totals (344, 343, 1) match those rows.
```

#### Repair the failed task

Return to the verify_delivery notebook, select the cell and send the repair prompt. Save it. In the failed run select Repair run, select verify and start the repair.

**Paste into Genie**

```text
Restore the assertion so it accepts only (224, 223, 1) or (344, 342, 2). Include actual in any error. Keep the dataset and pipeline unchanged.
```

**Checkpoint**

verify succeeds in the repaired run. The two missing values trace to first.csv on Torgersen and last.csv on Biscoe.

#### Do I need to process the files again?

Look at the two task results. If refresh succeeded and only verify failed, repair verify after correcting its expected count. Starting the whole job again would repeat a step that already succeeded.

**Sources** · [Jobs quickstart](https://docs.databricks.com/aws/en/jobs/jobs-quickstart) · [Repair failed runs](https://docs.databricks.com/aws/en/jobs/repair-job-failures) · [ZeroOps requirements](https://docs.databricks.com/aws/en/zeroops/get-started)

[Contents](../#contents) · [Field notes](#field-notes-07)

## Genie lab 08

### Check the island totals in Designer

*GENIE CODE · DESIGNER*

Putting the Gold totals on two screens would repeat any mistake in the Gold query. Instead, start from the individual Silver rows and count them by island using the Designer canvas. If both methods give 168 observations and 167 weights for Biscoe, you’ve checked how the totals were built.

Start with Silver and the full two-delivery Gold result from B06. Create a blank visual data prep with New • Visual data prep and name it island_check.

**Paste into Genie**

```text
Build a visual data prep on this canvas using workspace.palmer_lab.silver_obs. Add an Aggregate operator grouped by island. Count species as observations and non-null mass_g as weighed. Leave the aggregate as the final operator. Do not add an output table or schedule.
```

#### Verify independently

Inspect the operators and select Run all. Select Aggregate to see its three output rows. In SQL Editor with the serverless warehouse ask Genie Code for the independent check, then run it.

**Paste into Genie**

```text
From workspace.palmer_lab.gold_counts group by island and return SUM(obs_n) as observations, SUM(weighed_n) as weighed and SUM(missing_n) as missing. Sort by island.
```

**Checkpoint**

Compare the canvas observation and weight counts to the SQL results. Biscoe is 168 / 167, Dream 124 / 124 and Torgersen 52 / 51. The missing counts are 1, 0 and 1 respectively. Save the visual prep.

**The more you know**

Run all processes the complete input. A preview may only process a sample.

#### Why build the same summary a second time?

Designer starts with the individual Silver rows. Counting them here gives you a way to catch a mistake in the Gold query. If both screens used the same Gold result, agreement would tell you much less. Run the full input before comparing.

**Sources** · [Designer tutorial](https://docs.databricks.com/aws/en/designer/build-transformation) · [Genie on canvas](https://docs.databricks.com/aws/en/designer/genie-code) · [Designer availability](https://www.databricks.com/blog/whats-new-lakeflow-designer-october-2026-roundup)

[Contents](../#contents) · [Field notes](#field-notes-08)

## Genie lab 09

### Match the two names for each species

*GENIE CODE · SPECIES LOOKUP*

A biologist asks for Pygoscelis adeliae, while the field sheets say Adelie. Keep those names together in one reference table so a correction can reach every report that uses it. Join each observation to its species once. A left join keeps an unfamiliar label visible as a missing match, so you can correct the lookup without losing the observation.

Start with workspace.palmer_lab.silver_obs from B05 and its complete data from B06. Use a new SQL Editor query with the serverless warehouse.

**Paste into Genie**

```text
In workspace.palmer_lab create a Delta table named taxon with columns species and scientific_name. Insert exactly these three pairs: Adelie = Pygoscelis adeliae; Chinstrap = Pygoscelis antarcticus; Gentoo = Pygoscelis papua. Make the lab definition repeatable without duplicating rows. Create a view named field_obs that retains s.* from silver_obs and adds t.scientific_name using a LEFT JOIN to taxon on species. Comment that one row is one sampled observation including unweighed birds. Then check taxon for duplicate keys and report COUNT(*), COUNT(scientific_name), COUNT(mass_g) from field_obs.
```

#### Apply and inspect

Review the three name pairs and the join type. Use the generated SQL in the query editor, run it and inspect the duplicate report and counts. The lookup is created directly from the prompt in Databricks.

**Checkpoint**

No duplicated species keys; 344 joined rows, 344 mapped rows and 342 weights. Keep field_obs for B10 and B11.

#### Is this three-row lookup all Genie Ontology does?

Here you’ve built a SQL relationship. Genie Ontology has a wider job: it helps Genie interpret terms using definitions people publish and clues from queries, dashboards and agents. In lab 11 you publish a rule for “weighed”. If a popular query counts every observation, that query can be a clue, but the explicit metric definition carries more weight. For a Genie One answer with citations, open Genie Ontology under the response to inspect the sources it actually used. A new lookup may have no snippet to inspect yet.

**Sources** · [ITIS taxonomic record](https://www.itis.gov/servlet/SingleRpt/SingleRpt?search_topic=TSN&search_value=174444) · [Dataset and original studies](https://github.com/allisonhorst/palmerpenguins) · [Genie Ontology](https://docs.databricks.com/aws/en/genie/genie-ontology/)

[Contents](../#contents) · [Field notes](#field-notes-09)

## Genie lab 10

### Inspect lineage and grants

*GENIE CODE · GOVERNANCE*

The colleague reviewing a species name needs to know its source and whether the lookup table changed. Lineage traces the name back to taxon. Table history shows writes to taxon. Opening either as the owner tells you little about the colleague’s access, so you’ll also check history under their sign-in.

#### Inspect the owner’s view

Open Genie Code in SQL Editor on the workspace warehouse. Open Catalog • workspace • palmer_lab • field_obs • Lineage to follow scientific_name back to taxon. Open taxon • History for the write record.

**Paste into Genie**

```text
Show my session identity, the definition of workspace.palmer_lab.field_obs, the grants on workspace.palmer_lab.taxon and its latest Delta history entry. Identify the source of scientific_name. Run only read-only SQL.
```

#### Set up the reader

The workspace admin checks Settings • Identity and access • Users • Manage and assigns the colleague to this workspace if needed. Confirm they have Databricks SQL access. A warehouse manager checks Can use for the colleague or their group on the existing SQL warehouse. Replace READER_EMAIL in the prompt with the colleague’s actual Databricks sign-in.

**Paste into Genie**

```text
Draft SQL to give READER_EMAIL USE CATALOG on workspace, USE SCHEMA on workspace.palmer_lab and SELECT on workspace.palmer_lab.taxon. Include a way to inspect existing grants on each object. Show the SQL without executing it.
```

Check existing direct and group access. Have the owner or an authorized grantor for each object run only the missing statements. SELECT also lets the colleague read the table’s rows.

#### Test from the colleague’s own session

Have the colleague open this workspace URL, select the warehouse in SQL Editor and send the prompt to Genie Code in their own session. If Genie returns SQL, the colleague runs it there.

**Paste into Genie**

```text
Run SELECT session_user() and then DESCRIBE HISTORY workspace.palmer_lab.taxon LIMIT 1 as two separate statements. Show my identity and the returned version, timestamp and operation.
```

**Checkpoint**

Confirm the first result names the colleague and the second returns a history row. If the warehouse is unavailable, check SQL access and Can use. For a permission error naming an object, check its Unity Catalog grant. Remove only temporary grants added for this test.

#### Would the history show who read the table?

History lists writes, including creation or replacement. Lineage shows which data fed another table or view. A question about who opened or queried data belongs in audit records. The history and lineage checks here establish different facts.

**Sources** · [Workspace users](https://docs.databricks.com/aws/en/admin/users-groups/users) · [SQL warehouse access](https://docs.databricks.com/aws/en/admin/sql) · [Read access in Catalog](https://docs.databricks.com/aws/en/discover/database-objects) · [Delta history](https://docs.databricks.com/aws/en/tables/history) · [Unity Catalog lineage](https://docs.databricks.com/aws/en/data-governance/unity-catalog/data-lineage)

[Contents](../#contents) · [Field notes](#field-notes-10)

## Field notes 06

### Did the arriving file cause the update?

The station has three different questions. Did a file arrive? Did the pipeline process it? Did the output contain the right number of observations? Each question has its own evidence.

#### How a job fits around a pipeline

A job coordinates tasks and records their outcomes. The trigger asks Databricks to start a run when it detects a new file. The refresh task updates the pipeline tables. The verify task depends on refresh, so it checks the result after that task succeeds. Putting two tasks in the same job without a dependency leaves their order uncertain.

#### A trigger watches a path

Only inbox is watched. Creating hold/last.csv prepares the delivery, and placing last.csv in inbox makes it eligible to trigger a run. Overwriting an existing file under the same name does not trigger a file-arrival run. The trigger is a signal to look for work. Its run count is not a count of input files or survey rows.

#### What a green check can establish

The verification notebook accepts the valid totals both before and after the late file. A passing check alone cannot tell you which delivery it saw. Look at the run details for FILE_ARRIVAL, count the 120 Bronze rows attributed to last.csv, and check the final Gold totals. These observations connect the cause, the input and the outcome.

**The more you know**

A manual run gives 344 / 342 / 2. Can you sign off on the trigger? You can sign off on the data state. To establish how the job started, open the matching run and read its trigger.

#### Check your understanding

Refresh and verify both succeeded, but Bronze still has 224 rows. Which part of the delivery has yet to be demonstrated?

[Check your answer for lab 06](../appendix/#answers-for-labs-06-to-10)

**Sources** · [File arrival triggers](https://docs.databricks.com/aws/en/jobs/file-arrival-triggers) · [Configure a notebook task](https://docs.databricks.com/aws/en/jobs/tasks/notebook) · [Pipeline job task](https://docs.databricks.com/aws/en/jobs/tasks/pipeline)

[Contents](../#contents) · [Manual lab 06](#manual-lab-06) · [Genie lab 06](#genie-lab-06)

## Field notes 07

### What should you repair after a failure?

A red job run can mean that a perfectly good test caught bad data. It can also mean that the data are right and the test is wrong. The color alone gives you no way to choose the fix.

#### Locate the first failed task

Open the run and read the individual task outcomes. Here refresh succeeded. Verify expected 343 weighed observations and read 342. The assertion is useful because it shows the actual values. Compare those with a source independent of the failing test. The two original rows with NA explain the difference.

#### Repair has a scope

A repair can rerun selected unsuccessful tasks and their dependents as part of the original job run. Here only verify needs another attempt. A repair uses current task and job settings, so record the failing assertion before changing it. A green repaired attempt shows that the corrected check passed. The first failure is still part of the story.

#### When a retry can make things worse

Repeating a read-only count is inexpensive in this example. Repeating a task that inserts a review case, publishes a file or sends a notification may repeat the side effect. Before retrying, ask what was already committed and how the operation recognizes an earlier attempt. Lab 18 gives the review case a stable ID for precisely this reason.

**The more you know**

If Bronze contains only the first 224 rows, lowering the assertion to 224 hides the missing delivery. If Bronze contains both files and the archive confirms two missing weights, correct the test. The same failed assertion can lead to different actions.

#### Check your understanding

Verify fails after a successful refresh. What evidence would justify repairing only verify?

[Check your answer for lab 07](../appendix/#answers-for-labs-06-to-10)

**Sources** · [Repair failed runs](https://docs.databricks.com/aws/en/jobs/repair-job-failures) · [Jobs quickstart](https://docs.databricks.com/aws/en/jobs/jobs-quickstart) · [Delta history](https://docs.databricks.com/aws/en/tables/history)

[Contents](../#contents) · [Manual lab 07](#manual-lab-07) · [Genie lab 07](#genie-lab-07)

## Field notes 08

### How independent is a second calculation?

The Designer canvas gives you a second way to ask the counting question. Its value depends on where you start and which operation you choose.

#### Read the canvas as a series of operations

The Source node reads Silver at the observation level. Aggregate groups those rows by island and computes a value for each group. On large inputs, grouping can require Spark to move records with the same key to the same worker, a shuffle. An uneven key distribution can give one worker far more to do. After aggregation, Biscoe is one row. A value such as bill length for an individual bird no longer has a single meaning in that row unless you choose a calculation for it.

#### Decide what the comparison can detect

The canvas counts Silver rows directly. The SQL check adds the group counts already in Gold. Agreement is evidence that the Gold grouping and roll-up preserved the island counts. Both calculations still share Silver. A record lost during Bronze ingestion or Silver cleaning could be absent from both answers. Use the archive count from lab 03 to test that earlier part of the process.

#### Look closely at the column being counted

The canvas uses COUNT(species) as its observation counter. It works for this archive because every observation has a species label. If a future worker left species blank and the reader converted it to NULL, that observation would disappear from this count. COUNT(*) in SQL would keep it. This is exactly the kind of assumption a comparison can expose.

**The more you know**

On Biscoe, a preview shows a few records with weights and no missing ones. Run the full aggregation before drawing conclusions about completeness. The expected full result is 168 observations and 167 weights.

#### Check your understanding

The canvas and Gold both show 340 observations. What is a useful third comparison?

[Check your answer for lab 08](../appendix/#answers-for-labs-06-to-10)

**Sources** · [Designer tutorial](https://docs.databricks.com/aws/en/designer/build-transformation) · [Genie on canvas](https://docs.databricks.com/aws/en/designer/genie-code) · [Spark SQL performance](https://spark.apache.org/docs/latest/sql-performance-tuning.html)

[Contents](../#contents) · [Manual lab 08](#manual-lab-08) · [Genie lab 08](#genie-lab-08)

## Field notes 09

### How do names acquire meaning for Genie?

A biologist says “Pygoscelis adeliae.” A field sheet says “Adelie.” The team also uses the word “weighed.” The species names and the counting rule pose different problems, even though both matter when a colleague asks a question in ordinary language.

#### A lookup records an explicit relationship

The taxon table pairs a survey label with a scientific name. Many observations can refer to one species. The join can then attach the scientific name to each observation. A left join keeps an observation whose label has no match, giving you a chance to discover a spelling or coverage problem.

#### What Genie Ontology adds

Genie Ontology brings together modeled knowledge, such as published metric definitions, and knowledge learned from assets and their use, such as saved queries and dashboards. The lookup expresses a relationship for SQL users. The metric view in lab 11 expresses what the team counts as weighed. An agent can use that context when interpreting a question. An answer’s cited sources and generated query show which context actually informed it.

#### Why name mapping also needs a row-count test

Imagine accidentally adding a second mapping for Adelie. This is called join fan-out. Every one of the 152 Adelie observations would match twice. The joined view would grow from 344 to 496 rows, even if both mappings spell the scientific name correctly. Check uniqueness on the lookup key as well as coverage on the observations.

**The more you know**

A new survey label has no mapping. The joined row count can remain 344 while the number with a scientific name drops to 343. If you only checked the first number, you would miss the gap.

#### Check your understanding

There are 344 rows after the left join but only 343 populated scientific names. Where would you look first?

[Check your answer for lab 09](../appendix/#answers-for-labs-06-to-10)

**Sources** · [Genie Ontology](https://docs.databricks.com/aws/en/genie/genie-ontology/) · [Dataset and original studies](https://github.com/allisonhorst/palmerpenguins) · [ITIS taxonomic record](https://www.itis.gov/servlet/SingleRpt/SingleRpt?search_topic=TSN&search_value=174444) · [Define a metric view](https://docs.databricks.com/aws/en/uc-semantics/metric-views/create)

[Contents](../#contents) · [Manual lab 09](#manual-lab-09) · [Genie lab 09](#genie-lab-09)

## Field notes 10

### Who can see what happened to a table?

“Can my colleague see the history?” has several parts. Which person is signed in? Can they run SQL? Can they reach the table through its catalog and schema? What kind of history do they need?

#### Follow the access path

The account identifies the person. The workspace gives them a place to work. For the SQL route they need SQL access and permission to use the warehouse. Unity Catalog then checks the catalog, schema and table privileges. In this exercise the reader gets USE CATALOG, USE SCHEMA and SELECT on taxon. SELECT also lets them read the mapping rows.

#### Choose evidence that answers the question

DESCRIBE HISTORY lists Delta table versions and the operations that wrote them. Lineage describes inputs and outputs, such as taxon supplying scientific_name to field_obs. Audit records address access events. A saved version number can help a reproducibility check, but reading an old version also requires the underlying data files to remain available. A table version answers “what write occurred?” A lineage edge answers “what depends on this?” Neither establishes on its own that a particular colleague queried the rows.

#### Think in terms of effective permissions

The reader can receive permissions directly, through a group or from grants higher in the object hierarchy. Removing a temporary direct grant may leave group access intact. Test with the actual reader’s identity, and note which privileges existed before you began. A successful owner query tells you little about that reader.

**The more you know**

If the colleague can run SELECT session_user() but receives a permission error on taxon, the warehouse has already executed a query. Check which catalog, schema or table privilege the error names before changing compute settings.

#### Check your understanding

The reader can see a table in Catalog. Is that enough evidence that they can inspect its Delta history through SQL?

[Check your answer for lab 10](../appendix/#answers-for-labs-06-to-10)

**Sources** · [Unity Catalog privileges](https://docs.databricks.com/aws/en/data-governance/unity-catalog/best-practices) · [Delta history](https://docs.databricks.com/aws/en/tables/history) · [Unity Catalog lineage](https://docs.databricks.com/aws/en/data-governance/unity-catalog/data-lineage) · [SHOW GRANTS](https://docs.databricks.com/aws/en/sql/language-manual/security-show-grant)

[Contents](../#contents) · [Manual lab 10](#manual-lab-10) · [Genie lab 10](#genie-lab-10)

---

← [Labs 01–05](../labs-01-05/) · [Tutorial home](../) · [Labs 11–15](../labs-11-15/) →

---

## About the author

Mohammed Brückner is an enterprise architect in Munich, building production machine learning platforms on Azure and Databricks. More at [mohammed-brueckner.com](https://mohammed-brueckner.com) — and for talks or workshops on data platforms and enterprise AI, see [Speaking & Workshops](https://mohammed-brueckner.com/speaking/).

Books: [*Machine Learning Operations (MLOps) with Databricks on Azure End-to-End*](https://www.amazon.com/dp/B0FTSY78DR) — the production-scale continuation of this tutorial — and [Platform Economies](https://platformeconomies.com) (2026), on what happens when the API becomes the product.
