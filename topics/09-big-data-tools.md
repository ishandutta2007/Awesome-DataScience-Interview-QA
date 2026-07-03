# ⚙️ Big Data & Tools (Spark, Airflow, Cloud)

[← Back to main README](../README.md)

---

### Q: Explain how Spark's architecture achieves distributed processing, at a high level.

**Answer:**
Spark distributes data across a cluster as **RDDs (Resilient Distributed Datasets)** or higher-level **DataFrames**, partitioned across worker nodes. A **driver program** builds a logical execution plan (a DAG of transformations), which the **cluster manager** (YARN, Kubernetes, or Spark's standalone manager) schedules as tasks across **executors** on worker nodes, each processing its own data partition in parallel. Spark keeps intermediate data **in memory** across operations where possible (unlike Hadoop MapReduce, which writes intermediate results to disk between each step), which is the main reason it's typically much faster for iterative workloads like ML training.

---

### Q: What is the difference between a `transformation` and an `action` in Spark, and why does this distinction matter?

**Answer:**
**Transformations** (e.g., `map`, `filter`, `join`) are **lazy** — they build up a logical execution plan (DAG) without actually computing anything. **Actions** (e.g., `collect`, `count`, `write`) **trigger execution** of the accumulated transformations. This matters because Spark can **optimize the entire DAG** before executing it (e.g., combining/reordering operations, pushing filters earlier — "predicate pushdown"), and it avoids wasted computation on transformations whose results are never actually used.

---

### Q: What is data skew in a distributed system like Spark, and how do you address it?

**Answer:**
Data skew occurs when data isn't evenly distributed across partitions — e.g., a `groupBy` or `join` key where one value (like a popular product ID) accounts for a disproportionate share of rows, causing one partition/task to take far longer than the rest and become a bottleneck (**stragglers**), while other executors sit idle. Mitigations: **salting** the skewed key (adding a random suffix to spread it across more partitions, then aggregating results afterward), using Spark's **adaptive query execution (AQE)** which can automatically detect and handle skewed joins, or repartitioning explicitly with a custom partitioner that accounts for the known skew.

---

### Q: What is a DAG (Directed Acyclic Graph) in the context of Airflow, and why can't it have cycles?

**Answer:**
An Airflow DAG defines a workflow as a set of tasks with explicit dependencies (edges) — e.g., "task B runs only after task A succeeds." It must be **acyclic** because a cycle (A depends on B, which depends on A) would create an undefined/infinite execution order with no valid starting point — the scheduler wouldn't know which task to run first. Airflow's scheduler uses **topological sorting** of the DAG to determine valid execution order, which is only well-defined for acyclic graphs.

---

### Q: What is the difference between a data warehouse and a data lake?

**Answer:**
A **data warehouse** (e.g., Snowflake, Redshift, BigQuery) stores **structured, schema-on-write** data optimized for fast SQL analytics — data is cleaned/transformed before loading. A **data lake** (e.g., S3/Delta Lake, ADLS) stores **raw data in its native format** (structured, semi-structured, unstructured) with **schema-on-read**, offering more flexibility and lower storage cost, but requiring more processing effort to query effectively. Modern "lakehouse" architectures (Delta Lake, Iceberg, Databricks) try to combine both — data lake storage/cost with warehouse-like query performance and ACID transaction guarantees.

---

### Q: How would you design a data pipeline to ingest, clean, and serve data for a daily ML model retraining job?

**Answer:**
Typical structure: (1) **Ingestion**: pull raw data from source systems (databases, event streams, APIs) into a landing zone (data lake) via scheduled batch jobs or streaming (Kafka). (2) **Transformation/cleaning**: an orchestrated job (Airflow/dbt) validates schema, deduplicates, handles missing values, and writes cleaned data to a curated layer, ideally with **data quality checks** (e.g., Great Expectations) that fail the pipeline loudly rather than silently propagating bad data. (3) **Feature computation**: compute and store features in a **feature store** for consistency between training and serving. (4) **Training**: a scheduled job pulls the latest curated data/features, retrains the model, and validates it against a holdout set before promoting it. (5) **Monitoring**: track data drift, pipeline SLAs, and model performance post-deployment.

---

### Q: What is partitioning in a data warehouse/lake, and how does it improve query performance?

**Answer:**
Partitioning physically splits a large table into smaller chunks based on a column's value (commonly date) so that queries filtering on the partition column can **skip scanning irrelevant partitions entirely** (partition pruning), dramatically reducing I/O and query cost/time. E.g., partitioning an events table by `event_date` means a query for "yesterday's events" only scans one day's partition instead of the entire table's history. Over-partitioning (too many small partitions) can hurt performance too, due to metadata overhead — so partition granularity should match common query patterns.

---

### Q: What's the difference between batch processing and stream processing, and how do you decide which one to use?

**Answer:**
**Batch processing** (e.g., a nightly Spark job) processes data in large, discrete chunks on a schedule — simpler to reason about and debug, appropriate when some latency (minutes to hours) is acceptable. **Stream processing** (e.g., Kafka Streams, Flink, Spark Structured Streaming) processes data continuously as it arrives, enabling near-real-time results — necessary for use cases like fraud detection or real-time dashboards where minutes of delay matters. The tradeoff is stream processing is architecturally more complex (handling out-of-order events, exactly-once semantics, state management) — so unless there's a genuine business need for low latency, batch is often the simpler, more maintainable choice.

---

### Q: How would you handle schema evolution in a data pipeline — e.g., an upstream source adds a new column?

**Answer:**
Use file formats/table formats that support schema evolution natively (**Parquet with a schema registry, Delta Lake, Apache Iceberg**), which can add/rename/reorder columns without breaking existing readers. Enforce **schema validation at ingestion** to catch unexpected/breaking changes (e.g., a column type change) early rather than letting them silently corrupt downstream tables. For consumers (dashboards, ML features), prefer **explicit column selection** over `SELECT *`-style ingestion so new upstream columns don't unexpectedly appear/break assumptions in downstream code, and version your schemas so breaking changes can be rolled out with a migration path.

---

### Q: What is the CAP theorem, and how does it apply to choosing a distributed database?

**Answer:**
CAP theorem states a distributed system can only guarantee **two of three** properties during a network partition: **Consistency** (every read gets the latest write), **Availability** (every request gets a response, even if not the latest data), and **Partition tolerance** (the system continues operating despite network splits). Since partitions are unavoidable in real distributed systems, the practical choice is between **CP** (e.g., HBase, MongoDB in certain configs — prioritize consistency, may reject requests during a partition) and **AP** (e.g., Cassandra, DynamoDB — prioritize availability, may serve stale data during a partition). The right choice depends on the use case: financial transactions typically need CP; a social media feed can tolerate AP's eventual consistency.

---

### Q: What are the tradeoffs between row-based and column-based (columnar) storage formats?

**Answer:**
**Row-based** formats (e.g., CSV, Avro) store all fields of a record together — efficient for **writing/reading full records** (OLTP-style workloads: insert/update a single row). **Columnar** formats (e.g., Parquet, ORC) store each column's values together — far more efficient for **analytical queries** that scan a few columns across many rows (OLAP-style), since you only read the columns you need, and similar values compress much better together (better compression ratios). This is why data warehouses and lakes overwhelmingly favor columnar formats for analytics, while transactional databases favor row-based storage.

---

### Q: How do you monitor and ensure the reliability (SLA) of a production data pipeline?

**Answer:**
Track: **freshness** (is data as recent as expected — alert if a pipeline hasn't run/completed on schedule), **volume** (row counts within expected ranges — a sudden drop or spike often signals an upstream issue), **schema/quality checks** (nulls, duplicates, value ranges within expected bounds — tools like Great Expectations or dbt tests), and **task-level success/failure alerting** in the orchestrator (Airflow's SLA misses, retries, and failure notifications). Beyond automated checks, maintain clear **runbooks** for common failure modes and **data lineage** tracking so when something breaks downstream, you can quickly trace back to the root cause upstream.

---
