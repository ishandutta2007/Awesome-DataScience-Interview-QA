# 🧪 A/B Testing & Experimentation

[← Back to main README](../README.md)

---

### Q: Walk through how you'd design an A/B test end-to-end for a new checkout button color.

**Answer:**
1. **Define the hypothesis and primary metric** (e.g., "changing the button to green increases checkout conversion rate").
2. **Choose guardrail metrics** (e.g., revenue per user, page load time) to catch unintended harm.
3. **Compute required sample size** via power analysis, given baseline conversion rate, minimum detectable effect (MDE), significance level (α), and power (1−β).
4. **Randomize** users (typically at the user or session level, ensuring consistent experience) into control/treatment.
5. **Run for a predetermined duration** (covering full weekly cycles to account for day-of-week effects) — don't peek and stop early informally.
6. **Analyze** using the pre-specified test (e.g., two-proportion z-test) and check for statistical *and* practical significance, plus segment-level sanity checks (Simpson's Paradox risk).

---

### Q: How do you calculate the required sample size for an A/B test?

**Answer:**
Sample size depends on: baseline conversion rate (**p**), minimum detectable effect (**MDE**, the smallest lift worth detecting), significance level (**α**, usually 0.05), and desired power (**1−β**, usually 0.8). For a two-proportion test, the formula is approximately:

```
n ≈ 2 × (z_{α/2} + z_β)² × p(1−p) / MDE²
```

Smaller MDEs and higher power requirements both increase required sample size substantially — halving the MDE roughly **quadruples** the required sample size, which is a key intuition to communicate to stakeholders who want to detect tiny effects quickly.

---

### Q: What is "peeking" in A/B testing, and why is it a statistical problem?

**Answer:**
Peeking is repeatedly checking test results before the pre-determined sample size/duration is reached and stopping as soon as the result looks significant. This inflates the **false positive rate** far beyond the nominal α, because each additional look is another chance for random noise to cross the significance threshold — with enough peeks, you're almost guaranteed to see a "significant" result eventually even with no true effect. Fixes: commit to a fixed sample size/duration in advance, or use **sequential testing methods** (e.g., mSPRT, Bayesian sequential testing) explicitly designed to allow valid continuous monitoring.

---

### Q: What is a guardrail metric, and why do you need them alongside your primary success metric?

**Answer:**
A guardrail metric monitors for **unintended negative side effects** of a change, even if the primary metric improves — e.g., a UI change might boost click-through rate (primary metric) while degrading page load time or increasing customer support tickets (guardrails). Without guardrails, you risk shipping a change that "wins" on the metric you optimized for while quietly harming the broader user experience or business health.

---

### Q: How would you handle network effects / interference between treatment and control groups (e.g., in a social network or marketplace)?

**Answer:**
Standard A/B testing assumes the **Stable Unit Treatment Value Assumption (SUTVA)** — one user's treatment doesn't affect another's outcome. This breaks in social networks (a treated user's friends are indirectly affected) and marketplaces (treated sellers compete for the same limited buyer demand as control sellers, causing spillover). Solutions include **cluster/graph-based randomization** (randomize whole clusters of connected users together), **geo-based experiments** (randomize by region), or **switchback experiments** (randomize by time period rather than by unit) for marketplace-style interference.

---

### Q: Explain the difference between statistical significance and practical significance, with an example.

**Answer:**
Statistical significance means the observed effect is unlikely to be due to random chance (low p-value). Practical significance means the effect size is **large enough to matter for the business**. With a very large sample size, even a tiny, practically meaningless lift (e.g., +0.01% conversion) can be statistically significant. Always report the **effect size with a confidence interval**, not just a p-value, and evaluate whether the lower bound of that interval still represents a meaningful business impact before deciding to ship.

---

### Q: What is a multiple comparisons problem, and how do you correct for it when testing many metrics/variants at once?

**Answer:**
If you test many metrics (or many variants) simultaneously, each at α = 0.05, the probability of at least one **false positive** across all tests rises well above 5% — e.g., testing 20 independent metrics gives roughly a 64% chance of at least one spurious "significant" result. Corrections include the conservative **Bonferroni correction** (divide α by the number of tests) or the less conservative, more powerful **Benjamini-Hochberg (False Discovery Rate) procedure**, which controls the expected proportion of false positives among significant results rather than the probability of any false positive at all.

---

### Q: How would you run an A/B test when you can't randomize at the individual user level (e.g., a pricing change visible to all users in a region)?

**Answer:**
Use **cluster randomization** at whatever level prevents contamination — e.g., randomize by geographic region, city, or store, rather than by individual user. Analysis must then account for the **reduced effective sample size** (clusters, not individual users, are the true unit of randomization) using cluster-robust standard errors or mixed-effects models, otherwise you'll understate variance and overstate significance. When true randomization isn't feasible at all, consider quasi-experimental designs like **difference-in-differences** with a comparable control region.

---

### Q: A test shows a statistically significant lift in your primary metric, but your CFO asks "are you sure this isn't just noise?" How do you respond?

**Answer:**
Walk through: (1) the **pre-registered hypothesis and metric** (not something found via post-hoc data mining, which would invalidate the p-value's interpretation), (2) the **sample size and power** of the test relative to what was planned, (3) the **confidence interval** on the effect size, not just the point estimate — a tight, clearly-positive interval is much more convincing than a wide one that barely excludes zero, (4) whether the effect is **consistent across relevant segments** (not driven by one anomalous subgroup, i.e., no Simpson's Paradox), and (5) whether guardrail metrics moved in a concerning direction.

---

### Q: What is the difference between a frequentist and Bayesian approach to A/B testing?

**Answer:**
**Frequentist** testing treats the true effect as a fixed unknown value and asks "how extreme is my observed data, assuming no effect?" (p-values, confidence intervals) — it doesn't naturally support continuous peeking without correction. **Bayesian** testing treats the effect as a random variable with a prior distribution, updates to a posterior distribution given the data, and directly answers "what's the probability treatment beats control, given what I've observed?" — this framing is often more intuitive for stakeholders and naturally supports continuous monitoring, though it requires justifying prior choices and is more computationally involved.

---

### Q: How would you detect and handle a "novelty effect" or "primacy effect" skewing your A/B test results?

**Answer:**
A **novelty effect** is a temporary lift in engagement simply because something changed (users are curious/exploring the new UI), which fades over time; a **primacy effect** is the opposite — users initially resist a change out of familiarity bias, but engagement recovers as they adapt. Detect these by plotting the treatment effect **over time** (day-by-day or week-by-week) instead of only looking at the aggregate — a declining or rising trend over the test duration is a red flag. Mitigate by running the test **long enough** to let the effect stabilize, and by segmenting new users (no prior exposure/bias) from existing users.

---

### Q: What's a common pitfall when randomizing users into A/B test groups, and how do you validate randomization worked correctly?

**Answer:**
Common pitfalls: non-uniform hashing causing unequal group sizes, re-randomizing the same user across sessions (breaking the consistent-experience assumption), or randomization correlating accidentally with a confounder (e.g., time-of-day-based bucketing correlating with user type). Validate with an **A/A test** (split traffic but give both groups the same experience) before or alongside the real test — if a proper A/A test shows a "significant" difference, something is wrong with your randomization or measurement pipeline, and you should not trust results from the corresponding A/B test until it's fixed.

---
