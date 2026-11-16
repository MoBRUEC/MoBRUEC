---
permalink: /databricks/free-edition-tutorial/appendix/
title: "Databricks Free Edition Tutorial: Answer Key and Appendices"
description: "The answer key for all twenty labs, acceptance checks, the offline archive route when your workspace cannot reach the publisher, how to diagnose a wrong Genie answer, and how to choose between a fixed report and an open question."
---

# Answer Key and Appendices

*Part of the [Databricks Free Edition tutorial](../) with Palmer Penguins — [all labs and routes](../#contents).*

## Acceptance checks

### Find where a count went wrong

If a number looks wrong, start at the top of this table. For example, if the inbox has 224 records but Bronze has 448, check which files Bronze read before changing the dashboard. Save the result you saw and the run ID so you can return to that run.

| Layer | Expected result | Where to investigate |
| --- | --- | --- |
| Volume | 344 source records; deliveries of 224 and 120. | Check the verified notebook fetch or the offline cells, file names, year filter and volume path. |
| First update | 224 observations, 223 weights. | Inspect source_file. Only first.csv belongs in inbox at this stage. |
| Second delivery | 344 observations, 342 weights; 15 Gold groups. | Check the trigger, job run and inbox contents. A renamed duplicate is a new file. |
| Ontology | 344 joined, 344 mapped, 342 weighed. | Check that each species maps to exactly one lookup row. |
| Analytics | Biscoe 168/167; Dream 124/124; Torgersen 52/51. | Compare Gold and metric view results before checking filters or agent SQL. |
| Advanced labs | MAE 710.46 / 360.66 g; 342 shape keys; no records for Ross. | Inspect split year, vector order, sync status, tool choice and arguments. |

#### If a resource is unavailable

If you hit a usage limit, wait for it to reset before running the step again. For a permission error, check which account you’re using and the name of the table or file. If your region has no suitable model, record which parts of the exercise you could run.

#### What counts as evidence

Keep the file-triggered run, repaired task output, published dashboard, benchmark result and review row. The job history shows what started a run. The chat’s tool details show where an answer came from.

**Sources** · [Publisher’s CSV](https://raw.githubusercontent.com/allisonhorst/palmerpenguins/main/inst/extdata/penguins.csv) · [File arrival triggers](https://docs.databricks.com/aws/en/jobs/file-arrival-triggers) · [Repair failed runs](https://docs.databricks.com/aws/en/jobs/repair-job-failures) · [Edition limits](https://docs.databricks.com/aws/en/getting-started/free-edition-limitations)

[Contents](../#contents)

## Answers for labs 01 to 05

### Check your reasoning

Compare your explanation with the worked answer. Each link returns to the field notes for that question.

#### 01   A scheduled query worked in your notebook yesterday but now reports that taxon cannot be found. What would you check before creating a new table?

Check the catalog and schema in the job’s session. Use workspace.palmer_lab.taxon to test the named object under the job identity. If access is denied, investigate grants.

[Revisit lab 01](../labs-01-05/#field-notes-01)

#### 02   A colleague asks whether matching SHA-256 fingerprints prove that all the weighing scales were calibrated. What can the fingerprint establish?

The normalized archive text matches the tested input. Calibration is a fact about collection in the field. The bytes could match perfectly and still contain a bad measurement.

[Revisit lab 02](../labs-01-05/#field-notes-02)

#### 03   If a technician accidentally enters a weight of −1, will COUNT(TRY_CAST(body_mass_g AS INT)) include it? Which check would detect it?

Yes. −1 is a valid integer. A positive-weight rule can flag it. Successful conversion proves that the value has the right type, not that it is physically plausible.

[Revisit lab 03](../labs-01-05/#field-notes-03)

#### 04   Two different filenames contain the same observation. Can a successful incremental pipeline update tell you which row to keep?

No. File progress can be correct while the data contains repeated observations. A stable observation ID or a defensible matching rule is needed to decide whether two records describe one event.

[Revisit lab 04](../labs-01-05/#field-notes-04)

#### 05   Gold tells you a weight is missing on Biscoe. Can Gold alone recover the original text the worker entered?

No. Aggregating into Gold removed the individual field value. Use Silver to find the affected observation, then the raw Bronze value or delivery file to see the original entry.

[Revisit lab 05](../labs-01-05/#field-notes-05)

**Sources** · [TRY_CAST](https://docs.databricks.com/aws/en/sql/language-manual/functions/try_cast) · [Auto Loader checkpoints](https://docs.databricks.com/aws/en/ingestion/cloud-object-storage/auto-loader/) · [Expectations](https://docs.databricks.com/aws/en/ldp/expectations)

[Contents](../#contents)

## Answers for labs 06 to 10

### Check your reasoning

Compare your explanation with the worked answer. Each link returns to the field notes for that question.

#### 06   Refresh and verify both succeeded, but Bronze still has 224 rows. Which part of the delivery has yet to be demonstrated?

The second delivery. Verify permits the first state. Check for a new inbox file, the recorded trigger and the 120 Bronze rows from last.csv before signing off on the final state.

[Revisit lab 06](../labs-06-10/#field-notes-06)

#### 07   Verify fails after a successful refresh. What evidence would justify repairing only verify?

An independent count agrees with the refreshed data, and the assertion is demonstrably wrong. Correct it and repair verify. If counts differ from the archive, investigate upstream first.

[Revisit lab 07](../labs-06-10/#field-notes-07)

#### 08   The canvas and Gold both show 340 observations. What is a useful third comparison?

Compare with the raw archive of 344 and check Bronze by filename, then Silver. Agreement between two summaries derived from Silver cannot rule out an earlier loss.

[Revisit lab 08](../labs-06-10/#field-notes-08)

#### 09   There are 344 rows after the left join but only 343 populated scientific names. Where would you look first?

Find the row whose scientific name is NULL and inspect its original species label and mapping. A left join preserves the input even when the reference key has no match.

[Revisit lab 09](../labs-06-10/#field-notes-09)

#### 10   The reader can see a table in Catalog. Is that enough evidence that they can inspect its Delta history through SQL?

No. Have the reader confirm their session identity and run DESCRIBE HISTORY using a warehouse they can use. Visibility in Catalog alone does not exercise that path.

[Revisit lab 10](../labs-06-10/#field-notes-10)

**Sources** · [File arrival triggers](https://docs.databricks.com/aws/en/jobs/file-arrival-triggers) · [Repair failed runs](https://docs.databricks.com/aws/en/jobs/repair-job-failures) · [Genie Ontology](https://docs.databricks.com/aws/en/genie/genie-ontology/) · [Delta history](https://docs.databricks.com/aws/en/tables/history)

[Contents](../#contents)

## Answers for labs 11 to 15

### Check your reasoning

Compare your explanation with the worked answer. Each link returns to the field notes for that question.

#### 11   If an island has no measured weights, what should its mean mass be? Why would reporting zero be misleading?

The mean of an empty set of measurements is NULL. Zero would claim a measured mass of zero grams. Keep the observation and weighed counts alongside the mean.

[Revisit lab 11](../labs-11-15/#field-notes-11)

#### 12   The lead sees the updated title but the old counts. What would you inspect first?

The published design reached the viewer. Check dataset freshness, underlying data and active filters. Re-publishing the title does not establish which rows the query returned.

[Revisit lab 12](../labs-11-15/#field-notes-12)

#### 13   Your agent passes the benchmark for “weighed by island.” What question would test a different risk?

Ask about the wild population on Dream. A correct response distinguishes the 124 sampled observations from the unknown island population. This tests the limit of the data.

[Revisit lab 13](../labs-11-15/#field-notes-13)

#### 14   A third prediction rule scores better after you choose its settings by repeatedly looking at 2009. What has happened to the test set?

The test results helped choose the rule. Treat 2009 as validation data for that choice. A genuinely unseen season would support a fresh estimate of performance.

[Revisit lab 14](../labs-11-15/#field-notes-14)

#### 15   You double the flipper divisor in the index but leave the divisor in the query unchanged. Can you interpret the new nearest match?

The stored and query points now use different coordinates. Apply the same transform to both, sync the changed index, and compare with an exact distance calculation.

[Revisit lab 15](../labs-11-15/#field-notes-15)

**Sources** · [Query measures](https://docs.databricks.com/aws/en/uc-semantics/metric-views/query) · [Agent benchmarks](https://docs.databricks.com/aws/en/genie-agents/monitor) · [MLflow runs and experiments](https://www.mlflow.org/docs/latest/tracking/) · [AI Search and L2 distance](https://docs.databricks.com/aws/en/ai-search/ai-search)

[Contents](../#contents)

## Answers for labs 16 to 20

### Check your reasoning

Compare your explanation with the worked answer. Each link returns to the field notes for that question.

#### 16   The tool returns “no observations” for Ross and the assistant reports “zero birds live there.” Which part would you change first?

The function reported the scope correctly. Inspect the model instruction and final wording. The survey has no records for Ross, so it cannot establish the wild population.

[Revisit lab 16](../labs-16-20/#field-notes-16)

#### 17   An agent gives the right count for Biscoe using a query filtered to another island. Would you mark the case as passed?

The number alone might coincide. Mark the evidence check as failed and correct the island selection. Keep both the query and reply so the error can be reproduced.

[Revisit lab 17](../labs-16-20/#field-notes-17)

#### 18   The Biscoe record reads reviewed. Can you tell which colleague reviewed it on Tuesday?

The lab table stores only the current state. Answering who and when requires additional evidence, such as an event recorded with the reviewer’s identity and time.

[Revisit lab 18](../labs-16-20/#field-notes-18)

#### 19   The lead opens the app and sees the title, followed by a SQL permission error. Which identity should you investigate?

The lab uses app authorization for SQL. Check that the app service principal can use the bound warehouse and read the catalog, schema and Gold table.

[Revisit lab 19](../labs-16-20/#field-notes-19)

#### 20   The missing Biscoe value is still visible after the case is marked reviewed. Should the station report be corrected?

The two records are consistent. The survey still lacks a measurement, and the operational register records the human review. Report both facts with their sources.

[Revisit lab 20](../labs-16-20/#field-notes-20)

**Sources** · [Structured retrieval tools](https://docs.databricks.com/aws/en/agents/custom-agents/structured-retrieval-tools) · [Agent benchmarks](https://docs.databricks.com/aws/en/genie-agents/monitor) · [PostgreSQL conflict handling](https://www.postgresql.org/docs/current/sql-insert.html) · [Application identity](https://docs.databricks.com/aws/en/dev-tools/databricks-apps/auth)

[Contents](../#contents)

## Offline archive step 1

### Restore the field archive

If the notebook can’t fetch the survey, you can still do the exercise. The text on these two sections holds a compressed copy of the original CSV. The first cell saves half of it in notebook memory. The second uses both halves to put the CSV in your volume.

**Workspace and commands**

In stage_deliveries create a new Python cell. Copy the entire code block below, including the opening and closing triple quotes, and run it. Keep this notebook session active for the [next section](#offline-archive-step-2).

```python
part_one = """
/Td6WFoAAATm1rRGAgAhARYAAAB0L+Wj4DuICcFdADmcCM2vgJxm/2zcXLP1U0wQAB58kXaca9o1COyCT3lG
2URif2xxVFq01Cqk5t1q5KY1iemCx22Mf1uYLpX0xSyTPH38D+NbccW6cwbUPNl7ofPfzvxqHx8i3UAl1vWY
YPqZTr1D2SKRrbQTHWk0W6hpv741Hu0wGufC3Yex98QUnDullJoBD8mIY47WbPzuIkj40j8HLNbrEcAAo9SR
hsdlj6WgJVTxtULig4GTTqfgNsw5/DnoHvYE+GcCuKFFoctQIM9Mtq2GaXQ3sfTVDgD7nmFPuALUsjUT4yQ2
OGp3K4gq4vgNOscOnxyv70vhGswVn6djb2EcGSmERjb2Qa59JJrPCZO21dYNiq+cefZTrtXhhDhmkluMhlev
QSgSM4nAODsXNuE3uOU+wftw65vEEIr7z3KQP9sv/To0idYibcLt3eraJQIThjPEmrQvful28FXbxgUlnRRA
GRUyu905YXvzWt/NCXFV69FwZjIDtQWKx6WFHffcHnF4nZMNNTA1t39a1Z1QXwFnn3d6f/Q+uO8tVuuAnEu/
BH2sAr0SVOPyvs39/67d8/sIvUDL0CtGDEkRIZ54MjLTnTxYhJoxD8Eav0g/lJq/LiOXamAl8McBfcX0NpSx
E8rtqvCn9fnQFwJYfT4nfEWGVnKPzKrcpbOpI4zeJkjDkFi+LxB/tqPeSW4l/V9tayIjECiBFUr+zd9PA0bd
pZxSa1/mk+Ca+flRyReuWIeSt001uGmXBeLP5uBesiTnhQeNjrOKXx1YhvxfTHy/IxjO03oKj2tFESUGJQRP
yQjYdsNSOkLXb+GIUrl8u8Vp5nAMctL6NHLiLDyR8JJefXcQhS7ElZeUpw9iDnH3Y7W//CPNu9YLA05LH8QL
9awu60SVOwloTezbjElCR9Kw0F8bgjYX9FUpucGdxNDRh6ZL1wXlhCLmBfOQMN3y2IP8yajIrMWX12fq358d
BoW5XhEhl6stxVSK9NXCQ7uDUb6FLU3oPMQCp2zjkjlO7pTIEKLVC7BG1hGNL4tEFbtBdLvVG5UAMK2kilhP
4YZOklDckTwGG8YBFMBE0Mrphxjab5Ig7rixvbPHLyX86TRn+vP8IdIGuj+7+Itro+j0GDrvNfakKmJMaZEW
+/N5UCQvEHRlk0Gw3laIeWvWd3hZQwzt80RWb0rj/Lfpt1L/iNUesJD8+773RMujCUcZpSmMGAqDacNHr2FH
I3k3hrMJmlMLfvlFxQEPKW7L3u7fpUOBNLrn+ekXy2rFj2qyh1QbU3Po9xrGB2amE+/FtODsib5FlKwES2XN
Ti1GsKf3TWi1qzZdEt6Bj5A3e6rNlGyrWwf341XVWXBgxGOYrTPKCmK+y/jNBjNjTVqfQf7+4KWcqbNubPuF
ahMCWj/nyI5oHUZPwUHdlVxj4aQYJB9PxrDZFH+s65XWhtR+O5UqF0x+ES2teEEhVLLXGDVnidUR8upj1Nus
hvb4EswXjpFI90FDbYn+DXAaXtTRcp+Y257ROPMmiWeVM7aKm4ua3nY5CMmhGw/AF2yva6mnkK+geAa6viL+
mhai7H9DtSi3o8XWF0s/xUA3lCF74bqqa5PSXzPBAZGo+qWFW9BkYJQz96KQKIpv9iaYsUP6UVAkS45elHSq
KPvJpn+KvMUig3+qCrqhR3r3fX5zl7KF7/MO7PHE1GOu5OJ35AlEyZGYb04GDgvNt2fgT4WovsYXA6Q9cXgn
"""
```

**Checkpoint**

This cell stores the first half of the archive in memory. Continue directly to the [next section](#offline-archive-step-2).

**Sources** · [Dataset and original studies](https://github.com/allisonhorst/palmerpenguins) · [CC0 licence](https://allisonhorst.github.io/palmerpenguins/LICENSE.html) · [Work with volume files](https://docs.databricks.com/aws/en/volumes/volume-files)

[Contents](../#contents)

## Offline archive step 2

### Write the verified CSV

The first cell left half the survey file in memory. Run this second cell in the same notebook. It joins the halves, turns them back into a CSV and checks that the contents match the publisher’s file before saving it in landing.

```python
import base64, lzma, hashlib
from pathlib import Path
part_two = """
81EVyGyhTF0rzEZPcmGqFjUvAB1E6BF4/VEN4eI0VjLlmraODVMuCinCZcUZs3pddkGVLTINf5JXVjk4tKDU
7le1cylOmAZuTNfxfYJX4EaFP9zwnR7qsuJabIMo9b/hsQ9W9QU1ktMSTC64U4mmdt00goolWnfNE3SVHYdj
gSiQQZ2QLdRKG4vHobG/p5hEI16leFK1aS2vCSOINuxlUeuzyTRHL4xzXfBu84zN0QY5Ri5ljMQii4VuDQkE
ZTAP0RbSebbQuydNpvbFjxIuXbDfUOzWwUq2UX/+exlqEkHOdAlm1qNPvzpfBlsgyNh35kWspsr2x7hGx9WW
cZ7tyIuSLR6zE7EWLm2Jip8V/s9qthn/sqgb0LVbhHrOXJ7zY1yXoIQ9D3iPEIDiHMMftMkKDcvnRzy6uXlg
q8GEdQsXvxaq2KiENlqrwoLY7m5klsEKp9TWTny0XPQ9y0sZXi1dz41i33sCRMEfyiPjp9pYhhz9Z7hrHIpl
TY4lfFCq2KQXtyYy70quZ951Xf7G+qdebVnXVHshJ2k4h89pM0CDb86+wJuXnOmMLV6BLubOra0w5fWoeuHd
jPhDc5akea7/n3vxWf7aQRQQgouWyTbsqDOvGx6AG3Bp5WLzU0HXehhA6EYuiKaG2ZhGQGiPHypBAGiyOZS6
jv+Hf1JcFtPBPLXw+7+X9zs3wR9H0lY9ypr7tKoi0dXsS3BWaM8BzXbn7The9YtSiZJMqr81+dLBqDKZFsSw
QVL4ONTJluHPXGzOex0aB2+9hu7zCdN4N4GAtCheAFdU9Dn5lis2KH3OVVWuMLip6NE4K4PDhgiokXKAK0aR
w0UYYT8rEd21YX5nqYsw58r0N5bHZhMXMz0PMPD3hIHLxPfLAL5n5h22acFKcY6fTQOQjY/ggmORhqMR4Hs+
HwAmAQ4jWEtpDqxb9NyEaRygQ44nUOgvHucv4d/KmXzOGnJfxe44olYFM/xyDyIBBUx52aHtGxt1dvRC9a7h
9zXt/9uLbzlpb//dB84SisQysiicjBmYmnJn/GoO4QkdBEsoz/GGsbePtb5vtmq455vQD5oXL4oGEQjpMGwv
PexahI8YlXlwf2GV9bW2m5iDdxxgoagSicIepCjwju1IBdFwR/FwJeez6lfDq+nlMBlYQuk8t+SXpGZ0tz8c
FL3312xFmoSmEYLbyh+Yq3sxlpbo6A3tVDoWYap5Uv11p50kmf/BS3Cz4lFFPoEyH5itJy83GnEAySuUemzU
9PKlaUmZGpINnikSw2ZnjmAcw/NfM404IwdGjZeI+zMJYCyZSfNpOeF/htWm2IkVdwSXCtGu+h+ewWcPyrzF
R/mzSGFxAUkOjj0lr7Rv4kNLgfxumEzTWENi9DJ+HYmc6XEYmV7geZ3cK9FeXMrDplV/CYOvdhB9RRiUQR2E
J7cc5yCbvcpRnVhsCkQCobj2lPEWEkhFXu5gzEM0WyvhFx4X9CIXW9axp731REYsHRofZIW4g8Rb7HJ+i0C7
9WBBi6DkBUKBnFDDd1vgunBgdw7Iv4ngiYzdXRbfzFoE+/aymG0oth9Y77fpYRFwuo6cS9lkSf5AtkUb2GxT
9zGjEKMB2o4AAAAAdrgEwgMYBnUAAd0TiXcAABqxWouxxGf7AgAAAAAEWVo=
"""
data = lzma.decompress(base64.b64decode(part_one + part_two))
digest = "f204db2c753b0937caac3cb35258562c14f073e4bbc76be24b4c51ce22767a93"
assert hashlib.sha256(data).hexdigest() == digest
root = Path("/Volumes/workspace/palmer_lab/landing")
(root / "penguins.csv").write_bytes(data)
print(len(data.splitlines()) - 1, "observations")
```

**Checkpoint**

Expect 344 observations. Return to [Manual lab 02](../labs-01-05/#manual-lab-02) and run cell 2 to create the 224-row first delivery and the 120-row held delivery. If part_one is undefined, rerun the [preceding cell](#offline-archive-step-1) in this notebook.

**Sources** · [Publisher’s CSV](https://raw.githubusercontent.com/allisonhorst/palmerpenguins/main/inst/extdata/penguins.csv) · [CC0 licence](https://allisonhorst.github.io/palmerpenguins/LICENSE.html) · [Work with volume files](https://docs.databricks.com/aws/en/volumes/volume-files)

[Contents](../#contents)

## Genie offline archive

### Stage the source without internet

*OFFLINE SOURCE*

If Genie’s notebook cell can’t reach the publisher, use this copy of the survey instead. The block below contains the actual file in compressed form. Give the notebook those bytes, then ask Genie to turn them back into a CSV. You can do both steps in Databricks.

#### Supply the original records in the notebook

In stage_deliveries add a Python code cell. Paste and run this data-only assignment. The text represents the published CC0 records, compressed with LZMA and encoded as Base64.

```python
payload = """
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
WouxxGf7AgAAAAAEWVo=
"""
```

#### Generate the restore cell

Below the data cell choose + Genie Code. Paste the whole prompt below, accept the generated cell and run it in the same session.

**Paste into Genie**

```text
Write a Python cell that reads the string variable payload already set in this notebook, decodes it with base64.b64decode and decompresses with lzma.decompress. Verify SHA-256 equals f204db2c753b0937caac3cb35258562c14f073e4bbc76be24b4c51ce22767a93 before writing the verified bytes to /Volumes/workspace/palmer_lab/landing/penguins.csv. Print the number of data rows excluding the header. Use no network access.
```

**Checkpoint**

Expect 344 records. Return to the second prompt of B02 in [Genie lab 02](../labs-01-05/#genie-lab-02). Run its generated cell and check 224 and 120.

**Sources** · [Dataset and original studies](https://github.com/allisonhorst/palmerpenguins) · [CC0 licence](https://allisonhorst.github.io/palmerpenguins/LICENSE.html) · [Work with volume files](https://docs.databricks.com/aws/en/volumes/volume-files) · [Generate notebook cells](https://docs.databricks.com/aws/en/notebooks/basic-editing)

[Contents](../#contents)

## Diagnosing a Genie answer

### When the agent gives the wrong count

Suppose your colleague asks for the number weighed on Biscoe and gets 168. That is a real number in the survey. It answers a different question. Here’s how to find out what happened and decide whether the agent is ready to share.

#### Where did the question come from?

The clickable suggestion in the chat comes from a sample question you configure or one Genie suggests. Your colleague can also type freely. A saved benchmark runs separately, in a fresh conversation. To offer a question to readers and test it regularly, save it in both places.

#### Where did the answer come from?

Open Show generated code. MEASURE(observations) counts all 168 Biscoe entries. MEASURE(weighed) counts the 167 entries with weights. If Genie chose the first measure, clarify “weighed” in its instructions. In Configure • Examples you can also give Genie a worked SQL example to guide future queries.

#### What if the benchmark says 168 is correct?

Run the SQL Answer yourself. If it counts all rows, you’ve taught the test to accept the wrong result. Correct the reference SQL first. Then test both “How many were weighed on each island?” and “How many entries have a recorded weight, by island?” as separate benchmarks using the same reference query.

#### Who decides whether a failure stops sharing?

In Chat mode, results from the reference SQL are the test for accuracy. Agent mode uses a model to grade a multi-step response and can take an evaluation note. For this counts exercise, use Chat mode. If you require both phrasings to pass before sharing, that is your quality gate. After any fix, rerun both and inspect Evaluation.

#### Will my colleague see the same thing?

To use the shared survey desk, a colleague needs workspace consumer or Databricks SQL access, permission to run the agent and permission to read survey_metrics in Unity Catalog. The agent uses its author’s warehouse compute credentials and checks data access as the person asking. The direct SQL Editor test in lab 10 also required the reader’s own warehouse access.

**The more you know**

Where does Genie One fit? It can find a relevant agent for a business question. Opening Palmer survey desk directly selects the agent you set up. Genie Code is the assistant you used to build and edit the project.

**Sources** · [Agent benchmarks](https://docs.databricks.com/aws/en/genie-agents/monitor) · [Guide agent answers](https://docs.databricks.com/aws/en/genie-agents/tune-quality) · [Create a Genie Agent](https://docs.databricks.com/aws/en/genie-agents/set-up) · [Genie Ontology](https://docs.databricks.com/aws/en/genie/genie-ontology/) · [Genie One chat](https://docs.databricks.com/aws/en/genie-one/chat)

[Contents](../#contents)

## Choosing an agent

### A fixed report or an open question?

The lead starts with “How many weights are missing on Biscoe?” and follows up with “Which species had the missing weight?” Those two questions are enough to see why you might choose a function, a Genie Agent, or a combination.

| Decision | Playground with island_report | Palmer survey desk (Genie Agent) |
| --- | --- | --- |
| Who defines the calculation? | You write and test one SQL function. At run time the model supplies an island. | Genie uses the metric view and instructions to generate SQL suited to the wording and available data. |
| What can the user vary? | Ask about another island or call the function twice to compare islands. Each call returns the same three kinds of count. | Ask for a new grouping, filter or sort. Island and species are available in this lab’s metric view. |
| Where could it go wrong? | Wrong island, skipped call, faulty function, stale Gold data, or a reply that misquotes the tool. Fixed SQL can still cost compute. | Wrong measure, filter or grouping. Read the generated SQL and compare against a known result. |
| When would I choose it? | A recurring workflow where the same reviewed calculation feeds every report. Test again when its SQL changes. | A colleague exploring a new question. Give them clear metric definitions and benchmark the questions they rely on. |

#### Try the follow-up after labs 13 and 16

In Playground ask for the Biscoe counts, then ask which species had the missing weight. The tool returns 168 sampled, 167 weighed and one missing. Its result contains no species, so it cannot support an answer to the follow-up. An honest reply should explain what is missing.

In Palmer survey desk ask “For Biscoe, show observations, weighed and missing by species.” Inspect the island filter and grouping in Show generated code. Check Adelie (44 observed, 44 weighed) and Gentoo (124 observed, 123 weighed). The missing weight was a Gentoo observation.

#### Can I use both approaches in one Genie Agent?

Yes. Genie can also use a registered Unity Catalog function or a parameterized SQL example as a trusted asset. That lets you curate the common island report while keeping other questions open for exploration. “Trusted asset” describes how a query was supplied and used. You still need to verify its inputs, data and answer.

**Sources** · [Structured retrieval tools](https://docs.databricks.com/aws/en/agents/custom-agents/structured-retrieval-tools) · [Playground agent tutorial](https://docs.databricks.com/aws/en/getting-started/gen-ai-llm-agent) · [Create a Genie Agent](https://docs.databricks.com/aws/en/genie-agents/set-up) · [Guide agent answers](https://docs.databricks.com/aws/en/genie-agents/tune-quality) · [Agent benchmarks](https://docs.databricks.com/aws/en/genie-agents/monitor) · [Query measures](https://docs.databricks.com/aws/en/uc-semantics/metric-views/query)

[Contents](../#contents)

---

← [Labs 16–20](../labs-16-20/) · [Tutorial home](../)

---

## About the author

Mohammed Brückner is an enterprise architect in Munich, building production machine learning platforms on Azure and Databricks. More at [mohammed-brueckner.com](https://mohammed-brueckner.com) — and for talks or workshops on data platforms and enterprise AI, see [Speaking & Workshops](https://mohammed-brueckner.com/speaking/).

Books: [*Machine Learning Operations (MLOps) with Databricks on Azure End-to-End*](https://www.amazon.com/dp/B0FTSY78DR) — the production-scale continuation of this tutorial — and [Platform Economies](https://platformeconomies.com) (2026), on what happens when the API becomes the product.
