# 🏗️ System Design & MLOps

[← Back to main README](../README.md)

---

### Q: Design a machine learning system to detect fraudulent transactions in real time. Walk through the key components.

**Answer:**
1. **Requirements**: low-latency scoring (typically < 100-200ms) at transaction time, high recall on fraud (with an acceptable false positive rate given customer friction), and the ability to update quickly as fraud patterns evolve.
2. **Feature pipeline**: a mix of **real-time features** (transaction amount, velocity of transactions in the last N minutes, device/location anomalies) computed via a **streaming system (Kafka + Flink/Spark Streaming)**, and **precomputed batch features** (historical user behavior aggregates) served from a low-latency **feature store**.
3. **Model serving**: a lightweight, low-latency model (often gradient-boosted trees rather than deep learning, for latency/interpretability) deployed behind a real-time inference API, with a **fallback rules engine** for cases the model is uncertain about or for known hard-coded fraud patterns.
4. **Feedback loop**: labels arrive with delay (confirmed fraud/chargebacks come in days/weeks later), so the system needs a mechanism to incorporate delayed labels into retraining without leaking future information.
5. **Monitoring**: track prediction distribution drift, feature drift, and false positive rate (customer complaints) as leading indicators of model degradation, since fraud patterns adapt adversarially over time.

---

### Q: What is the difference between online and offline (batch) model inference, and when would you use each?

**Answer:**
**Batch inference** scores a large set of examples on a schedule (e.g., nightly churn scores for all users) and stores results for later lookup — simpler infrastructure, no strict latency requirement, but predictions can be stale by the time they're used. **Online inference** scores individual requests in real time as they arrive (e.g., a fraud check at transaction time) — requires a low-latency serving infrastructure (model loaded in memory, feature retrieval must also be fast) but gives fresh, request-specific predictions. Choose batch when the use case tolerates staleness and you can precompute for a bounded set of entities; choose online when predictions must reflect the current request context or when the space of possible inputs is too large to precompute.

---

### Q: What is model drift, and how would you detect it in production?

**Answer:**
**Model drift** occurs when a model's performance degrades over time because the real-world data distribution has shifted from what it was trained on. Two main types: **data/covariate drift** (input feature distributions change, e.g., user demographics shift) and **concept drift** (the relationship between features and the target changes, e.g., what predicts churn changes after a pricing change). Detect via: monitoring **input feature distributions** over time (statistical tests like KS-test/PSI comparing current vs. training distribution), tracking **prediction distribution** shifts, and — where ground truth eventually arrives — monitoring actual **model performance metrics** over time, with alerting thresholds tied to acceptable degradation.

---

### Q: How would you design a feature store, and what problem does it solve?

**Answer:**
A feature store centralizes feature computation and storage so features are **defined once and reused consistently** across training and serving, solving the common problem of **training-serving skew** — where features computed differently in an offline training pipeline vs. an online serving pipeline cause silent model degradation in production. Key components: an **offline store** (batch-computed features for training, often a data warehouse/lake table) and an **online store** (low-latency key-value store like Redis/DynamoDB for real-time serving), with a shared **feature definition layer** ensuring the same transformation logic generates both, plus **point-in-time correctness** guarantees (ensuring training data only uses features as they existed at the time of the historical event, avoiding label leakage).

---

### Q: What is A/B testing a model in production called, and what's the difference between that and shadow deployment / canary deployment?

**Answer:**
**A/B testing (champion/challenger)** routes a portion of live traffic to the new model and compares business/model metrics against the current production model directly. **Shadow deployment** runs the new model alongside the production model on live traffic, but its predictions are only **logged, not acted on** — useful for validating a new model's behavior/latency against real traffic without any user-facing risk, before trusting it enough to run a real A/B test. **Canary deployment** gradually rolls out the new model to a small percentage of traffic (e.g., 1% → 5% → 25% → 100%), monitoring closely at each stage, primarily to catch **operational failures** (crashes, latency spikes, errors) early and limit blast radius, rather than to compare business metrics per se.

---

### Q: How would you design a recommendation system for a large e-commerce platform, considering scale?

**Answer:**
Typical two-stage architecture: (1) **Candidate generation** — quickly narrow millions of items down to a few hundred plausible candidates per user using cheap, scalable methods (e.g., collaborative filtering via embeddings + approximate nearest neighbor search like FAISS/HNSW, or simple co-occurrence/popularity-based heuristics), since scoring every item with a complex model is computationally infeasible at scale. (2) **Ranking** — apply a more expensive, higher-precision model (e.g., a gradient-boosted tree or deep ranking model using richer features: user history, item metadata, context) to rank just the shortlisted candidates. This two-stage design balances **scalability (candidate generation)** with **precision (ranking)**, which is the standard pattern used by most large-scale recommendation systems (YouTube, Amazon, etc.).

---

### Q: What are the key considerations when deciding whether to retrain a model on a schedule vs. trigger-based retraining?

**Answer:**
**Scheduled retraining** (e.g., weekly/monthly) is simpler to operate and predictable, appropriate when data/concept drift happens gradually and the cost of a slightly stale model is low. **Trigger-based retraining** (kicked off when monitored drift/performance metrics cross a threshold) responds faster to sudden shifts (e.g., a major behavior change from an external event) but requires robust automated monitoring and can be riskier if triggers fire on noise rather than genuine drift. Many production systems use a hybrid: a baseline schedule plus alerting that can trigger an off-cycle retrain when performance degrades unexpectedly. Also weigh the **cost of retraining** (compute, data pipeline complexity, validation/approval process) against the expected benefit of freshness.

---

### Q: How would you ensure reproducibility in an ML pipeline?

**Answer:**
Key practices: **version everything** — code (git), data (data versioning tools like DVC, or immutable dataset snapshots), model artifacts, and the exact library/dependency versions (containerization via Docker, pinned requirements). **Set and log random seeds** for any stochastic process (train/test split, model initialization, shuffling). Use **experiment tracking** (MLflow, Weights & Biases) to log hyperparameters, metrics, and artifacts for every run, so any past result can be traced back to the exact code/data/config that produced it. Ideally, the whole pipeline (data prep → training → evaluation) is expressed as **declarative, orchestrated steps** (Airflow, Kubeflow) rather than manual notebook execution, so it can be re-run deterministically.

---

### Q: What is the difference between model interpretability and model explainability, and why does this distinction matter for production ML systems?

**Answer:**
**Interpretability** refers to models that are inherently understandable by design — you can directly trace how inputs map to outputs (e.g., linear regression coefficients, shallow decision trees). **Explainability** refers to techniques applied *after the fact* to approximate or explain the behavior of an inherently complex/black-box model (e.g., SHAP, LIME on a deep neural network or large ensemble) — these are approximations, not ground truth, and can sometimes be misleading if not used carefully. The distinction matters in regulated domains (credit, healthcare, hiring) where regulators or stakeholders may require genuinely interpretable models, versus lower-stakes domains where a more accurate black-box model plus a good explainability layer is an acceptable tradeoff.

---

### Q: How would you design monitoring/alerting for a production ML system beyond just tracking accuracy?

**Answer:**
Layer monitoring across several concerns: **operational health** (latency, error rate, uptime of the serving infrastructure — standard SRE practices), **data quality** (input feature nulls/out-of-range values, schema changes from upstream sources), **data/prediction drift** (statistical distribution shifts in inputs or outputs vs. training baseline), **model performance** (where ground truth is available, even with delay — track precision/recall/relevant business metrics over time, not just at deployment), and **business impact metrics** (the actual downstream KPI the model is meant to improve, since a model can look statistically fine while failing to move the metric it exists to serve). Alerting thresholds should be tuned to avoid alert fatigue, and dashboards should make it easy to distinguish "the model needs retraining" from "the pipeline is broken" from "there's a genuine, non-model-related business shift."

---
