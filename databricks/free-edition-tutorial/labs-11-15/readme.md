---
permalink: /databricks/free-edition-tutorial/labs-11-15/
title: "Databricks Labs 11-15: Metric Views, Genie Agents, ML and Vector Search"
description: "Define a metric once and reuse it everywhere, publish a brief, teach a Genie agent your definitions, test a weight prediction with ML, and find observations by vector similarity — manual and Genie routes for every lab."
---

# Labs 11–15: Shared Definitions, Agents and ML

*Part of the [Databricks Free Edition tutorial](../) with Palmer Penguins — [all labs and routes](../#contents).*

## Manual lab 11

### Define “weighed” once

*SEMANTIC METRICS*

Two reports can both say “weighed” and silently count different things. On Biscoe, counting all observations gives 168 while counting recorded weights gives 167. Put the agreed calculation in a metric view. Then a dashboard author and an agent can both ask for weighed by island or by species without rewriting that rule.

**Workspace and commands**

Run the following in SQL Editor on the serverless warehouse. Keep YAML indentation and the double dollar delimiters as shown.

```sql
CREATE OR REPLACE VIEW workspace.palmer_lab.survey_metrics
WITH METRICS LANGUAGE YAML AS
$$
version: 1.1
source: workspace.palmer_lab.field_obs
dimensions:
  - name: island
    expr: source.island
  - name: species
    expr: source.species
measures:
  - name: observations
    expr: COUNT(1)
    comment: All sampled records including missing weights.
  - name: weighed
    expr: COUNT(source.mass_g)
    comment: Records with measured body mass in grams.
  - name: mean_mass_g
    expr: AVG(source.mass_g)
$$;
SELECT species, MEASURE(observations), MEASURE(weighed),
       ROUND(MEASURE(mean_mass_g), 1) AS mean_g
FROM workspace.palmer_lab.survey_metrics
GROUP BY species ORDER BY species;
```

[Open the equivalent Genie route](#genie-lab-11)

**Checkpoint**

Adelie 152 / 151 / 3700.7 g. Chinstrap 68 / 68 / 3733.1 g. Gentoo 124 / 123 / 5076.0 g. The mean across the 342 weighed records is 4201.8 g.

#### What changes when I ask “by island” instead of “by species”?

Island and species are dimensions: they decide how the rows are grouped. Weighed is a measure: it counts recorded weights inside each group. Leave out the grouping to get 342 overall. This metric is modeled knowledge in Genie Ontology. Writing “weighed” only in one agent’s instructions would leave dashboard authors to recreate the rule. A metric view gives both the agent and SQL users a calculation they can call.

**Sources** · [Define a metric view](https://docs.databricks.com/aws/en/uc-semantics/metric-views/create) · [Query measures](https://docs.databricks.com/aws/en/uc-semantics/metric-views/query) · [Genie Ontology](https://docs.databricks.com/aws/en/genie/genie-ontology/)

[Contents](../#contents) · [Field notes](#field-notes-11)

## Manual lab 12

### Publish the station brief

*AI/BI DASHBOARDS*

The station lead wants the same overview every week. A chart makes the busiest island easy to spot, while a table lets the reader check the one-record gap between 168 observed and 167 weighed on Biscoe. Build both from the metric view so one display can’t silently adopt a different definition of weighed.

**Workspace and commands**

New • Dashboard. Rename it Palmer survey brief. In Data • Add SQL dataset, run the query below using the Free Edition warehouse. Rename the dataset Island coverage.

```sql
SELECT island, MEASURE(observations) AS observed,
       MEASURE(weighed) AS weighed,
       MEASURE(observations) - MEASURE(weighed) AS missing
FROM workspace.palmer_lab.survey_metrics
GROUP BY island ORDER BY island;
```

On the canvas select Add a visualization. Choose Bar and Island coverage. Put island on X and observed on Y. Title it “Sampled observations by island”. Add a Table widget showing all four columns, titled “Weight coverage”. Publish to your private workspace and open the published version.

[Open the equivalent Genie route](#genie-lab-12)

**Checkpoint**

Biscoe has the tallest bar at 168 observations. Biscoe and Torgersen each have one missing weight. Dream has none. The published view has both widgets.

**Gotcha**

Clear filters and run the dataset query again if the counts differ. A filtered dashboard can omit perfectly valid rows.

#### Why can I see my chart edit while a colleague sees the old chart?

Your edit is in the draft. Publish again, then open the published dashboard and compare its title and widgets with what your colleague sees. If the design matches but the counts differ, check the filters and data permissions.

**Sampled observations by island**

| Island | Observations |
| --- | ---: |
| Biscoe | 168 |
| Dream | 124 |
| Torgersen | 52 |

**Sources** · [Dashboard tutorial](https://docs.databricks.com/aws/en/dashboards/tutorials/create-dashboard) · [Dashboard authoring](https://docs.databricks.com/aws/en/dashboards/manage/dashboard-agent) · [Query measures](https://docs.databricks.com/aws/en/uc-semantics/metric-views/query) · [Dashboard drafts](https://docs.databricks.com/aws/en/dashboards/manage)

[Contents](../#contents) · [Field notes](#field-notes-12)

## Manual lab 13

### Teach the analyst’s agent

*GENIE AGENTS*

The dashboard answers the questions you chose when you built it. A colleague may ask a new one tomorrow, such as “Which species had missing weights?” Give a Genie Agent the metric view so it can write a query for the question that arrives. Use a known island count to test whether it picks the intended measure before you trust it with a new question.

**Workspace and commands**

Genie Agents • New. Select only workspace.palmer_lab.survey_metrics and create the agent. Name it Palmer survey desk. Choose the existing SQL warehouse. Add instructions: “Observation means one sampled record. Weighed means a non-null body mass. Missing equals observations minus weighed. These data cannot establish wild population size.”

#### Give the reader a starting point

In the agent settings, add “How many observations were weighed on each island?” as a sample question. Open a new chat and click it. In a second new chat type “By island, how many records have a weight?” Open Show generated code in both answers and look for the weighed measure.

#### Give the author a repeatable test

In Benchmarks • Add benchmark, enter the sample question and paste the query below into SQL Answer. Save it, choose Chat mode, run the benchmark and open Evaluation. The reference query tells Databricks which results to expect.

```sql
SELECT island, MEASURE(weighed) AS weighed
FROM workspace.palmer_lab.survey_metrics
GROUP BY island ORDER BY island;
```

[Open the equivalent Genie route](#genie-lab-13)

**Checkpoint**

The numerical result is Biscoe 167, Dream 124 and Torgersen 51. For the population question, the supported statement is that Dream has 124 sampled records. Its wild population is unknown from these data.

**The more you know**

A benchmark becomes a quality gate if you decide to require a pass before sharing. In a fresh chat ask “How many penguins live on Dream Island?” Check that the reply explains the limits of the survey. See [Diagnosing a Genie answer](../appendix/#diagnosing-a-genie-answer) for a failed benchmark and how to fix it.

**Sources** · [Create a Genie Agent](https://docs.databricks.com/aws/en/genie-agents/set-up) · [Agent benchmarks](https://docs.databricks.com/aws/en/genie-agents/monitor) · [Query measures](https://docs.databricks.com/aws/en/uc-semantics/metric-views/query) · [Genie execution boundaries](https://docs.databricks.com/aws/en/databricks-ai/databricks-ai-trust) · [Genie One chat](https://docs.databricks.com/aws/en/genie-one/chat) · [Guide agent answers](https://docs.databricks.com/aws/en/genie-agents/tune-quality)

[Contents](../#contents) · [Field notes](#field-notes-13)

## Manual lab 14

### Test a weight prediction

*ML AND MLFLOW*

If you had to estimate a missing weight, would knowing the species improve the guess? The overall average is the simplest baseline. Compare it with an average for each species, using exactly the same 2009 birds to judge both. Keeping the two scores in MLflow lets a colleague see how much accuracy the extra detail bought.

**Workspace and commands**

New • Notebook • Python • Serverless. Name it mass_baseline. This cell learns the global mean and one mean per species on 2007–08. It predicts each held-out 2009 weight and averages the absolute error in grams. Smaller error means closer predictions. The two experiment runs retain those scores. Run the cell.

```python
import mlflow
from statistics import mean
data = spark.sql("""
    SELECT yr, species, mass_g FROM workspace.palmer_lab.silver_obs
    WHERE mass_g IS NOT NULL
""").collect()
train = [r for r in data if r.yr < 2009]
test = [r for r in data if r.yr == 2009]
overall = mean(r.mass_g for r in train)
by_species = {s: mean(r.mass_g for r in train if r.species == s)
              for s in {r.species for r in train}}
for name in ("overall_mean", "species_mean"):
    predictions = [overall if name == "overall_mean"
                   else by_species[r.species] for r in test]
    mae = mean(abs(r.mass_g - p) for r, p in zip(test, predictions))
    with mlflow.start_run(run_name=name):
        mlflow.log_param("test_year", 2009)
        mlflow.log_metric("mae_g", mae)
        mlflow.log_metric("test_n", len(test))
    print(name, len(train), len(test), round(mae, 2))
```

Open the notebook experiment or Experiments and compare mae_g across the two runs. If the tracking client is missing, add mlflow-skinny in the notebook’s Environment dependencies, apply and rerun. The environment handles installation.

[Open the equivalent Genie route](#genie-lab-14)

**Checkpoint**

The split has 223 training and 119 test weights. The global mean gives 710.46 g MAE; the species model gives 360.66 g. There are still 342 genuinely weighed records.

#### Why keep the known 2009 weights out of training?

They give you answers to check after the predictions are made. Letting those weights influence the averages would give the model an unfair advantage. MLflow saves each run’s error in grams. A smaller error can help you choose a method, but an estimated weight is still a guess.

**Sources** · [MLflow tracking](https://docs.databricks.com/aws/en/mlflow/tracking-server-configuration) · [Free Edition overview](https://docs.databricks.com/aws/en/getting-started/free-edition) · [Publisher’s CSV](https://raw.githubusercontent.com/allisonhorst/palmerpenguins/main/inst/extdata/penguins.csv)

[Contents](../#contents) · [Field notes](#field-notes-14)

## Manual lab 15

### Find an observation by similarity

*AI SEARCH*

A field note gives a 39.1 mm bill and a 181 mm flipper. SQL equality can find that exact pair. AI Search earns its place when a later note has slightly different readings and you want the closest records. Start with the known pair so you can check the index and the order of the measurements.

**Gotcha**

Allow time for the first index. In this workspace the sync pipeline was still idle after about 25 minutes while Databricks allocated serving compute. Genie estimated up to an hour for that run. You can start lab 16 while you wait: it uses Gold and works independently of this index.

**Workspace and commands**

In SQL Editor create a Delta source with Change Data Feed. Check that the six-field key is unique on these measurements.

```sql
CREATE OR REPLACE TABLE workspace.palmer_lab.shape_points
USING DELTA TBLPROPERTIES ('delta.enableChangeDataFeed'='true') AS
SELECT CONCAT_WS('|', species, island, CAST(yr AS STRING),
         CAST(bill_mm AS STRING), CAST(flipper_mm AS STRING),
         CAST(mass_g AS STRING)) AS record_key,
       species, island, yr,
       ARRAY(CAST(bill_mm / 50 AS FLOAT),
             CAST(flipper_mm / 200 AS FLOAT)) AS shape
FROM workspace.palmer_lab.silver_obs
WHERE bill_mm IS NOT NULL AND flipper_mm IS NOT NULL;
SELECT COUNT(*), COUNT(DISTINCT record_key)
FROM workspace.palmer_lab.shape_points;
```

Compute • AI Search • Create endpoint. Use Standard, name it palmer-search, or reuse your one standard endpoint. Catalog • shape_points • Create • Vector search index. Name it workspace.palmer_lab.shape_idx. Set key record_key, Use existing embeddings, shape, dimension 2, Triggered sync. Select the endpoint and create the index.

#### When can I run the lookup?

Save the SQL below as shape_note_lookup. Open Compute • AI Search • palmer-search and select workspace.palmer_lab.shape_idx, or open that index in Catalog Explorer. If its status says Provisioning, check again in 10–15 minutes. Even an Online endpoint can have an index still starting up. Run the saved query once the index shows Ready. If it reports a failure, open the status details.

```sql
SELECT species, island, yr FROM vector_search(
  index => 'workspace.palmer_lab.shape_idx',
  query_vector => ARRAY(CAST(39.1 / 50 AS FLOAT),
                        CAST(181 / 200 AS FLOAT)), num_results => 1
);
```

[Open the equivalent Genie route](#genie-lab-15)

**Checkpoint**

Source check: 342 rows and 342 distinct keys. Search check: the live index query returns Adelie, Torgersen, 2007. Keep the search check pending until the query returns that row.

#### Does the closest match identify an individual penguin?

Scaling sets how much each measurement counts toward distance. Use the same scales for the index and the query. Similar readings can come from different birds. After editing the source, sync the triggered index before searching for new readings.

**Sources** · [Delta Sync indexes](https://docs.databricks.com/aws/en/ai-search/create-ai-search) · [Vector search SQL](https://docs.databricks.com/aws/en/sql/language-manual/functions/vector_search) · [Edition limits](https://docs.databricks.com/aws/en/getting-started/free-edition-limitations) · [Endpoint and index readiness](https://docs.databricks.com/aws/en/ai-search/high-qps)

[Contents](../#contents) · [Field notes](#field-notes-15)

## Genie lab 11

### Give “weighed” a shared definition

*GENIE CODE · METRIC VIEW*

Two reports can both say “weighed” and silently count different things. On Biscoe, counting all observations gives 168 while counting recorded weights gives 167. Put the agreed calculation in a metric view. Then a dashboard author and an agent can both ask for weighed by island or by species without rewriting that rule.

Start with the B09 field_obs view. Open a new SQL Editor query, select the serverless warehouse and open Genie Code.

**Paste into Genie**

```text
Create or replace the Unity Catalog metric view workspace.palmer_lab.survey_metrics over workspace.palmer_lab.field_obs. Use WITH METRICS LANGUAGE YAML, version 1.1. Dimensions: island and species. Measures: observations = COUNT(1) including unweighed records; weighed = COUNT(source.mass_g); mean_mass_g = AVG(source.mass_g). Add clear comments giving the grain and unit. Also generate a separate query grouped by species and ordered by species, calling MEASURE for each measure and rounding mean_mass_g to one decimal. Add an overall mean query.
```

#### Run and inspect

Review the source, indentation and definition of weighed. Apply and run the definition in SQL Editor. Run the two generated queries.

**Checkpoint**

Adelie: 152 observations, 151 weighed, mean 3700.7 g. Chinstrap: 68, 68, 3733.1 g. Gentoo: 124, 123, 5076.0 g. Overall mean: 4201.8 g. This metric view is the source for the dashboard and analyst’s agent.

**Gotcha**

An average of the three species means gives each species equal weight. The overall query should evaluate the measure without a species grouping.

#### What changes when I ask “by island” instead of “by species”?

Island and species are dimensions: they decide how the rows are grouped. Weighed is a measure: it counts recorded weights inside each group. Leave out the grouping to get 342 overall. This metric is modeled knowledge in Genie Ontology. Writing “weighed” only in one agent’s instructions would leave dashboard authors to recreate the rule. A metric view gives both the agent and SQL users a calculation they can call.

**Sources** · [Define a metric view](https://docs.databricks.com/aws/en/uc-semantics/metric-views/create) · [Query measures](https://docs.databricks.com/aws/en/uc-semantics/metric-views/query) · [Genie Ontology](https://docs.databricks.com/aws/en/genie/genie-ontology/)

[Contents](../#contents) · [Field notes](#field-notes-11)

## Genie lab 12

### Publish the island brief

*GENIE CODE · DASHBOARD*

The station lead wants the same overview every week. A chart makes the busiest island easy to spot, while a table lets the reader check the one-record gap between 168 observed and 167 weighed on Biscoe. Build both from the metric view so one display can’t silently adopt a different definition of weighed.

Start with B11’s survey_metrics. Open New • Dashboard, name the draft Palmer survey brief and open Genie Code in the dashboard.

**Paste into Genie**

```text
Build a dashboard named Palmer survey brief from workspace.palmer_lab.survey_metrics on my serverless SQL warehouse. Add one SQL dataset named Island coverage grouped by island. Use MEASURE(observations) as observed, MEASURE(weighed) as weighed, and their difference as missing. Add a bar chart titled Sampled observations by island with island on X and observed on Y. Add a table titled Weight coverage showing island, observed, weighed and missing. Use the same dataset for both widgets. Leave the dashboard in draft for inspection.
```

#### Publish and check

Check the Data tab, run the dataset, then inspect the chart and table on the canvas. If the authoring assistant proposes changes, apply them. If it returns only SQL, use Data • Add SQL dataset, name it Island coverage, then add a Bar and a Table widget with the fields named in the prompt. Click Publish with private workspace access and open the published dashboard.

**Checkpoint**

The tallest bar is Biscoe at 168. The table shows Biscoe 168 / 167 / 1, Dream 124 / 124 / 0 and Torgersen 52 / 51 / 1. Clear filters if the displayed counts differ.

#### Why can I see my chart edit while a colleague sees the old chart?

Your edit is in the draft. Publish again, then open the published dashboard and compare its title and widgets with what your colleague sees. If the design matches but the counts differ, check the filters and data permissions.

**Sources** · [Dashboard tutorial](https://docs.databricks.com/aws/en/dashboards/tutorials/create-dashboard) · [Dashboard authoring](https://docs.databricks.com/aws/en/dashboards/manage/dashboard-agent) · [Query measures](https://docs.databricks.com/aws/en/uc-semantics/metric-views/query) · [Dashboard drafts](https://docs.databricks.com/aws/en/dashboards/manage)

[Contents](../#contents) · [Field notes](#field-notes-12)

## Genie lab 13

### Create the analyst’s agent

*GENIE AGENTS · CONFIGURATION*

The dashboard answers the questions you chose when you built it. A colleague may ask a new one tomorrow, such as “Which species had missing weights?” Give a Genie Agent the metric view so it can write a query for the question that arrives. Use a known island count to test whether it picks the intended measure before you trust it with a new question.

Start with the metric view workspace.palmer_lab.survey_metrics from B11.

#### Prepare the configuration with Genie Code

Open Genie Code in SQL Editor and send this prompt. Keep the proposed settings and SQL.

**Paste into Genie**

```text
Prepare configuration text for a Genie Agent named Palmer survey desk whose sole source is workspace.palmer_lab.survey_metrics. Define observation as one sampled record, weighed as non-null measured mass, and missing as observations minus weighed. Instruct it to distinguish sample counts from wild populations. Suggest a clickable sample question, an everyday paraphrase for testing and a benchmark SQL answer to “How many observations were weighed on each island?” using island and MEASURE(weighed), grouped and sorted by island.
```

#### Create and use the agent

Genie Agents • New. Attach workspace.palmer_lab.survey_metrics and click Create. Name it Palmer survey desk, select the warehouse and save the instructions. In agent settings save the island question as a sample question. Open a new chat and click it. Under Benchmarks • Add benchmark, add the island question separately. Check the proposed SQL uses MEASURE(weighed), save it as SQL Answer, run in Chat mode and open Evaluation.

#### Test different wording in a fresh agent chat

**Paste into Genie**

```text
By island, how many survey entries have a recorded weight?
```

Compare Show generated code with the result of clicking the sample. Next ask the population question in a fresh chat. If it guesses, clarify the instructions and retest.

**Paste into Genie**

```text
How many penguins live on Dream Island? Explain what the survey supports.
```

**Checkpoint**

The numerical benchmark returns Biscoe 167, Dream 124 and Torgersen 51. The agent can identify 124 sampled Dream records, but the wild population is unknown.

**The more you know**

The sample helps a colleague start. The benchmark scores a fresh answer. You choose whether a failed test holds up sharing. [Diagnosing a Genie answer](../appendix/#diagnosing-a-genie-answer) shows how to diagnose a failure.

**Sources** · [Create a Genie Agent](https://docs.databricks.com/aws/en/genie-agents/set-up) · [Agent benchmarks](https://docs.databricks.com/aws/en/genie-agents/monitor) · [Query measures](https://docs.databricks.com/aws/en/uc-semantics/metric-views/query) · [Genie One chat](https://docs.databricks.com/aws/en/genie-one/chat) · [Guide agent answers](https://docs.databricks.com/aws/en/genie-agents/tune-quality)

[Contents](../#contents) · [Field notes](#field-notes-13)

## Genie lab 14

### Evaluate weight predictors

*GENIE CODE · MLFLOW*

If you had to estimate a missing weight, would knowing the species improve the guess? The overall average is the simplest baseline. Compare it with an average for each species, using exactly the same 2009 birds to judge both. Keeping the two scores in MLflow lets a colleague see how much accuracy the extra detail bought.

Start with the complete B06 Silver data. New • Notebook • Python • Serverless. Name the blank notebook mass_baseline. Use + Genie Code to create one cell. It estimates mass from the earlier seasons and scores predictions against later measurements. MAE means the average absolute difference in grams; lower is better.

**Paste into Genie**

```text
Generate a self-contained Python notebook cell using Spark SQL, statistics.mean and MLflow. Load yr, species and mass_g from workspace.palmer_lab.silver_obs where mass_g is not null. Use 2007–08 as training data and 2009 as held-out test data. Fit one global mean mass predictor and one mean per species, using training data only. Evaluate each on the same test rows with mean absolute error in grams. Log two MLflow runs to the notebook experiment, named overall_mean and species_mean. Log test_year=2009, test_n and mae_g. Print training count, test count and both MAEs to two decimals. Leave the source table unchanged.
```

#### Run and compare

Accept the generated cell, inspect the split and run it. Open the notebook’s experiment pane or Experiments to compare the two runs. If the import fails because the client is missing, add mlflow-skinny under notebook Environment dependencies, apply and rerun.

**Checkpoint**

Training: 223 measured observations. Testing: 119. Overall mean MAE: 710.46 g. Species mean MAE: 360.66 g. The results should appear both in the notebook and as two MLflow runs.

#### Why keep the known 2009 weights out of training?

They give you answers to check after the predictions are made. Letting those weights influence the averages would give the model an unfair advantage. MLflow saves each run’s error in grams. A smaller error can help you choose a method, but an estimated weight is still a guess.

**Sources** · [MLflow tracking](https://docs.databricks.com/aws/en/mlflow/tracking-server-configuration) · [Publisher’s CSV](https://raw.githubusercontent.com/allisonhorst/palmerpenguins/main/inst/extdata/penguins.csv) · [Generate notebook cells](https://docs.databricks.com/aws/en/notebooks/basic-editing)

[Contents](../#contents) · [Field notes](#field-notes-14)

## Genie lab 15

### Retrieve a known measurement

*GENIE CODE · AI SEARCH*

A field note gives a 39.1 mm bill and a 181 mm flipper. SQL equality can find that exact pair. AI Search earns its place when a later note has slightly different readings and you want the closest records. Start with the known pair so you can check the index and the order of the measurements.

**Gotcha**

Allow time for the first index. In this workspace the sync pipeline was still idle after about 25 minutes while Databricks allocated serving compute. Genie estimated up to an hour for that run. You can start lab 16 while you wait: it uses Gold and works independently of this index.

Start with workspace.palmer_lab.silver_obs after B06. Use SQL Editor and the serverless warehouse. You’ll put each pair of measurements in a table and build an index so AI Search can find the nearest pair.

**Paste into Genie**

```text
Create or replace the Delta table workspace.palmer_lab.shape_points over workspace.palmer_lab.silver_obs, with delta.enableChangeDataFeed=true. Filter out null bill_mm and flipper_mm. Include species, island, yr, record_key and shape. Make record_key by joining species, island, yr, bill_mm, flipper_mm and mass_g in this order with '|', casting numeric fields to strings. Set shape to ARRAY(CAST(bill_mm / 50 AS FLOAT), CAST(flipper_mm / 200 AS FLOAT)). Generate a count and distinct-key check. Prepare a separate vector_search SQL query on index 'workspace.palmer_lab.shape_idx' using query_vector = [39.1 / 50, 181 / 200] as FLOATs and num_results = 1. Return species, island and yr.
```

#### Create, index and query

Run the table definition and key check first. In Compute • AI Search, create Standard endpoint palmer-search or reuse your existing standard endpoint. Catalog • workspace • palmer_lab • shape_points • Create • Vector search index. Name the index workspace.palmer_lab.shape_idx. Select key record_key, Use existing embeddings, column shape, dimension 2, and Triggered sync. Select the endpoint and create the index. Save the search query as shape_note_lookup.

#### Check the index before running the query

Open Compute • AI Search • palmer-search and select workspace.palmer_lab.shape_idx, or open the index in Catalog Explorer. If it says Provisioning and the sync pipeline is waiting for capacity, leave it in place and check again in 10–15 minutes. If it reports a failure, open its status details.

**Paste into Genie**

```text
Check whether the index workspace.palmer_lab.shape_idx on palmer-search is ready to query. If it is Ready, run shape_note_lookup and show the returned row. If it is still provisioning, report its status. If you cannot inspect it here, tell me where to check it in Catalog Explorer.
```

**Checkpoint**

The source has 342 rows and 342 distinct keys. The expected live result is Adelie, Torgersen, 2007. Record it as verified only after the index query returns it.

#### Does the closest match identify an individual penguin?

Scaling sets how much each measurement counts toward distance. Use the same scales for the index and the query. Similar readings can come from different birds. After editing the source, sync the triggered index before searching for new readings.

**Sources** · [Delta Sync indexes](https://docs.databricks.com/aws/en/ai-search/create-ai-search) · [Vector search SQL](https://docs.databricks.com/aws/en/sql/language-manual/functions/vector_search) · [Edition limits](https://docs.databricks.com/aws/en/getting-started/free-edition-limitations) · [Endpoint and index readiness](https://docs.databricks.com/aws/en/ai-search/high-qps)

[Contents](../#contents) · [Field notes](#field-notes-15)

## Field notes 11

### Which numbers can you safely add?

The team asks for the average penguin weight across all three species. Someone takes the mean of the three species averages and gets about 4,170 g. The tutorial’s metric returns 4,201.8 g. Both calculations used the same survey.

#### Measures and dimensions

A dimension, such as island or species, says how to divide the observations. A measure says what calculation to perform within each group. The metric view gives the calculation a name and makes it available to SQL, dashboards and Genie. Asking for weighed by island uses the same rule as weighed by species. Only the groups change.

#### Why averages need their denominators

The species contributed 151, 68 and 123 usable weights. Averaging the three species means gives each species equal influence. To find the mean for a weighed observation, add the individual masses and divide by 342. Gold also stores the pieces for that calculation: SUM(total_mass_g) divided by SUM(weighed_n). The same reasoning applies to percentages. An island with 52 sightings should not receive the same weight as an island with 168 when reporting overall coverage.

#### The metric name carries a promise

A dashboard and a Genie Agent can call the same measure and still disagree if one filters the data differently. Before editing the definition, compare the population being counted, the dimensions and the filters. A change to the shared definition can affect every consumer using it.

**The more you know**

Which answer should you give to “What does the average species weigh?” Ask whether each species or each measured observation should count equally. The two figures above answer two different, reasonable questions.

#### Check your understanding

If an island has no measured weights, what should its mean mass be? Why would reporting zero be misleading?

[Check your answer for lab 11](../appendix/#answers-for-labs-11-to-15)

**Sources** · [Define a metric view](https://docs.databricks.com/aws/en/uc-semantics/metric-views/create) · [Query measures](https://docs.databricks.com/aws/en/uc-semantics/metric-views/query) · [Genie Ontology](https://docs.databricks.com/aws/en/genie/genie-ontology/)

[Contents](../#contents) · [Manual lab 11](#manual-lab-11) · [Genie lab 11](#genie-lab-11)

## Field notes 12

### What does a published dashboard promise?

The station lead opens yesterday’s link. A title has changed in your draft, a late delivery has changed Gold, and the lead may have a filter selected. All three can change what someone sees in different ways.

#### Follow one visual back to its data

The dashboard dataset is a query that returns an island, an observation count, a weighed count and their difference. Both widgets use that result. The bar chart makes relative size easy to see. The table shows the one-record gaps that a glance at bar heights could miss. Biscoe’s tall bar measures sampling activity. It does not estimate the island’s population.

#### Publishing and refreshing answer different questions

Publishing makes the current dashboard design available to its viewers. Running or refreshing the dataset obtains results from the underlying data. A draft title can remain invisible to a colleague until you publish. A published title can be correct while the displayed counts still reflect an earlier refresh or an active filter.

#### Check whose access the dashboard uses

Databricks dashboards have publishing options for data access. With shared data permissions, viewers can see results using the publisher’s data permissions. With individual data permissions, the viewer’s own access to the underlying data matters. Check the chosen option before sharing and test as an intended viewer. A privately published dashboard is enough for the lab.

**The more you know**

The lead sees 168 observations in total. That number matches Biscoe alone. First inspect the selected islands and the rows in the dataset. Rebuilding the chart or changing the metric before checking the filter could conceal a perfectly valid view of one island.

#### Check your understanding

The lead sees the updated title but the old counts. What would you inspect first?

[Check your answer for lab 12](../appendix/#answers-for-labs-11-to-15)

**Sources** · [Dashboard tutorial](https://docs.databricks.com/aws/en/dashboards/tutorials/create-dashboard) · [Dashboard drafts](https://docs.databricks.com/aws/en/dashboards/manage) · [Dashboard sharing](https://docs.databricks.com/aws/en/dashboards/share/share) · [Query measures](https://docs.databricks.com/aws/en/uc-semantics/metric-views/query)

[Contents](../#contents) · [Manual lab 12](#manual-lab-12) · [Genie lab 12](#genie-lab-12)

## Field notes 13

### What does an analyst’s Genie Agent learn?

A colleague clicks “How many observations were weighed on each island?” Later they type “Where do we have a weight on file?” You want both to reach the same metric even though the sentences are different.

#### From a question to a query

The curated agent has the data assets and instructions you gave it. It interprets the question, generates SQL for the selected data and uses the result to answer. Show generated code makes that decision inspectable. Look for the weighed measure, a grouping by island, and any filters that narrow the question. A plausible sentence gives you little insight into those choices.

#### Three places to teach three different things

A sample question is a visible starting point for the reader. Instructions explain local language and limits, such as what the field team calls an observation. A benchmark is an author’s test question with an expected answer, here supplied by reference SQL. Running it checks the result. A caller does not have to type the benchmark wording to activate a special response.

#### Keep a few questions out of the examples

If you tune an agent using only one sentence, a pass on that sentence tells you little about paraphrases. Try a held-back wording in a fresh chat. Also include a question the data cannot answer, such as population. Keep the reference query under review: an incorrect reference would reward the wrong calculation.

**The more you know**

Genie Code helps you build or change a workspace asset. Palmer survey desk is the configured agent answering questions about survey data. Genie One can help someone find and consult an agent. When diagnosing a bad answer, record which surface and which agent produced it.

#### Check your understanding

Your agent passes the benchmark for “weighed by island.” What question would test a different risk?

[Check your answer for lab 13](../appendix/#answers-for-labs-11-to-15)

**Sources** · [Create a Genie Agent](https://docs.databricks.com/aws/en/genie-agents/set-up) · [Guide agent answers](https://docs.databricks.com/aws/en/genie-agents/tune-quality) · [Agent benchmarks](https://docs.databricks.com/aws/en/genie-agents/monitor) · [Genie One chat](https://docs.databricks.com/aws/en/genie-one/chat) · [Genie Ontology](https://docs.databricks.com/aws/en/genie/genie-ontology/)

[Contents](../#contents) · [Manual lab 13](#manual-lab-13) · [Genie lab 13](#genie-lab-13)

## Field notes 14

### How good is a simple prediction?

An error of 361 grams sounds large until you know what a simpler guess would have cost. This lab compares two ways to estimate mass, using exactly the same observed weights to judge them.

#### Draw a line between learning and judging

Use 223 measured observations from 2007–08 to learn the averages. Set aside the 119 measured observations from 2009 for evaluation. For each test row, predict its mass before looking at its known weight. The overall model always predicts the same training average. The species model looks up the training average for that species.

#### Read the unit as well as the score

Mean absolute error takes the size of each prediction error in grams and averages those sizes. Predictions 100 g too high and 100 g too low contribute 100 g each. Their errors cannot cancel. The species approach reduces this error from 710.46 g to 360.66 g on the held-out season. That score describes the average error across the test records. It does not give an uncertainty interval for any one penguin.

#### What MLflow saves, and what you may need later

An experiment groups runs. A run can record parameters, metrics and artifacts. This notebook logs the test year, number of test records and error. To reproduce a comparison on changing data, also identify the input snapshot and code used. The notebook uses collect() to bring 342 rows into Python. On a large survey, compute the grouped means and errors in Spark and return the small summary. The scoring idea transfers even when the implementation must change.

**The more you know**

The two records with no measured mass can receive estimates, but you cannot use them to calculate prediction error. Keep estimates separate from measured values, or a later report will incorrectly claim that every penguin was weighed.

#### Check your understanding

A third prediction rule scores better after you choose its settings by repeatedly looking at 2009. What has happened to the test set?

[Check your answer for lab 14](../appendix/#answers-for-labs-11-to-15)

**Sources** · [MLflow tracking](https://docs.databricks.com/aws/en/mlflow/tracking-server-configuration) · [MLflow runs and experiments](https://www.mlflow.org/docs/latest/tracking/) · [Dataset and original studies](https://github.com/allisonhorst/palmerpenguins)

[Contents](../#contents) · [Manual lab 14](#manual-lab-14) · [Genie lab 14](#genie-lab-14)

## Field notes 15

### What does “similar” mean to a search index?

The field note gives two measurements. An ordinary SQL query can check for an exact pair. A search index lets you ask which observations are near a slightly different pair. The definition of “near” is a design choice you make.

#### A vector is just an ordered list here

The first number represents bill length divided by 50. The second represents flipper length divided by 200. Both were measured in millimeters, but their typical sizes differ. Apply the same ordering and scales to the stored records and the query. This example uses measurements you can inspect by eye. It needs no language model to produce embeddings.

#### Scaling changes which neighbor wins

With this scaling, a difference of 1 mm in bill length and a difference of 4 mm in flipper length each move the point by 0.02 along one axis. Change either divisor and the ranking can change even if the survey measurements stay fixed. Databricks uses an approximate nearest-neighbor search with L2 distance. On a small sample you can sort all points by squared distance to get an exact reference for the index.

#### The source, the index and its serving compute

The Delta table holds the vectors. Its change data feed records row changes for downstream processing. The index is a searchable copy that a triggered sync updates. The endpoint supplies compute for queries. Count and inspect the source while the endpoint provisions. After a source update, an older index can return an older nearest neighbor until it syncs.

**The more you know**

For a note of 39.2 mm and 181 mm, an exact distance calculation on this archive ranks the 39.1 mm, 181 mm Torgersen record first. The original 39.1 mm note checks a known match. The altered note tests the reason to use similarity search at all.

#### Check your understanding

You double the flipper divisor in the index but leave the divisor in the query unchanged. Can you interpret the new nearest match?

[Check your answer for lab 15](../appendix/#answers-for-labs-11-to-15)

**Sources** · [AI Search and L2 distance](https://docs.databricks.com/aws/en/ai-search/ai-search) · [Delta Sync indexes](https://docs.databricks.com/aws/en/ai-search/create-ai-search) · [Vector search SQL](https://docs.databricks.com/aws/en/sql/language-manual/functions/vector_search) · [Endpoint and index readiness](https://docs.databricks.com/aws/en/ai-search/high-qps)

[Contents](../#contents) · [Manual lab 15](#manual-lab-15) · [Genie lab 15](#genie-lab-15)

---

← [Labs 06–10](../labs-06-10/) · [Tutorial home](../) · [Labs 16–20](../labs-16-20/) →

---

## About the author

Mohammed Brückner is an enterprise architect in Munich, building production machine learning platforms on Azure and Databricks. More at [mohammed-brueckner.com](https://mohammed-brueckner.com) — and for talks or workshops on data platforms and enterprise AI, see [Speaking & Workshops](https://mohammed-brueckner.com/speaking/).

Books: [*Machine Learning Operations (MLOps) with Databricks on Azure End-to-End*](https://www.amazon.com/dp/B0FTSY78DR) — the production-scale continuation of this tutorial — and [Platform Economies](https://platformeconomies.com) (2026), on what happens when the API becomes the product.
