# Lab 18 Reflection — Lakehouse Anti-Patterns

**Student:** Pham Trung Hieu

Of the "Top 5 Lakehouse Anti-Patterns," the one our team would most likely fall
into is **the small-file problem**. In a real LLM observability pipeline, requests
arrive continuously from many microservices, each writing its own micro-batch to
the Bronze layer every few seconds. Without a scheduled OPTIMIZE job, this rapidly
produces thousands of tiny Parquet files. As NB2 demonstrated, 200 such files
raised a single user_id point-query from ~18 ms to ~222 ms — a 12× slowdown.

The root cause is cultural rather than technical: teams focus on the ingestion
path ("make sure nothing is lost") and neglect the compaction path ("make sure it
stays queryable"). In production this compounds quickly — a week of micro-batches
from five services can generate tens of thousands of files, causing downstream
dashboards to time out and engineers to blame the query engine rather than the
storage layout.

The fix (scheduled `OPTIMIZE + ZORDER` on each Silver/Gold table) is
straightforward once the problem is understood, but adding it retroactively to
an existing pipeline requires a maintenance window. The lesson is to bake
compaction into the pipeline design from day one.
