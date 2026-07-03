# 📊 Statistics & Probability

[← Back to main README](../README.md)

---

### Q: What is the difference between Type I and Type II error?

**Answer:**
A **Type I error** (false positive) is rejecting a true null hypothesis — concluding there's an effect when there isn't one. Its probability is **α** (significance level). A **Type II error** (false negative) is failing to reject a false null hypothesis — missing a real effect. Its probability is **β**, and **1 − β** is statistical power. There's a trade-off: lowering α (stricter significance) generally increases β unless you increase sample size.

**Follow-up:** How would you decide which error is more costly in a fraud-detection system?

---

### Q: Explain p-value in plain language.

**Answer:**
The p-value is the probability of observing data at least as extreme as what you saw, **assuming the null hypothesis is true**. It is *not* the probability the null hypothesis is true, and it's not the probability your result is due to chance. A small p-value (typically < 0.05) suggests the observed data would be unlikely under the null, giving evidence to reject it — but statistical significance isn't the same as practical significance.

**Follow-up:** Why is p-hacking a problem, and how do you guard against it?

---

### Q: What is the Central Limit Theorem and why does it matter?

**Answer:**
The CLT states that the sampling distribution of the mean of a sufficiently large number of i.i.d. random samples (regardless of the population's original distribution) approaches a normal distribution, with mean equal to the population mean and variance σ²/n. It matters because it justifies using normal-based confidence intervals and hypothesis tests (like z-tests) even when the underlying data isn't normally distributed, as long as n is reasonably large (rule of thumb: n ≥ 30).

**Follow-up:** Does the CLT hold if the underlying distribution has infinite variance (e.g., Cauchy)?

---

### Q: What's the difference between correlation and causation? How do you test for causation?

**Answer:**
Correlation measures the strength/direction of a linear (or monotonic) relationship between two variables; it says nothing about one causing the other — confounders, reverse causation, or coincidence can produce correlation. To establish causation you typically need a **randomized controlled experiment** (A/B test), or, when experiments aren't feasible, quasi-experimental methods like **difference-in-differences, instrumental variables, regression discontinuity, or propensity score matching** — combined with a causal graph (DAG) to reason about confounders.

**Follow-up:** Give an example of a confounder that creates spurious correlation.

---

### Q: Explain the difference between a Poisson distribution and a Binomial distribution.

**Answer:**
**Binomial** models the number of successes in a *fixed* number of independent Bernoulli trials (n, p) — e.g., 10 coin flips. **Poisson** models the number of events occurring in a *fixed interval of time/space*, given events occur independently at a constant average rate λ — e.g., customer arrivals per hour. Poisson can be seen as the limiting case of Binomial as n → ∞, p → 0, with np = λ held constant (rare-event approximation).

**Follow-up:** When would you use Poisson vs. Negative Binomial for count data?

---

### Q: What is a confidence interval, and what does "95% confidence" actually mean?

**Answer:**
A confidence interval is a range constructed from sample data that is likely to contain the true population parameter. "95% confidence" means: if you repeated the sampling process many times and built an interval each time using the same method, **95% of those intervals would contain the true parameter** — it does *not* mean there's a 95% probability the true value lies in this one specific interval (a common misconception, since the true value is fixed, not random).

**Follow-up:** How does confidence interval width change as sample size increases?

---

### Q: What is the difference between variance and standard deviation, and why do we use both?

**Answer:**
Variance is the average of squared deviations from the mean (units are squared, e.g. dollars²); standard deviation is the square root of variance, bringing it back to the original units (dollars), which makes it more interpretable. Variance is mathematically convenient for algebra (e.g., additivity of independent variances), while standard deviation is preferred for reporting spread and for constructing intervals (e.g., "±1 SD").

---

### Q: Explain Bayes' Theorem and walk through a practical example.

**Answer:**
Bayes' theorem: **P(A|B) = P(B|A) · P(A) / P(B)**. It lets you update a prior belief P(A) given new evidence B. Classic example: a disease affects 1% of a population; a test is 99% sensitive and 95% specific. If someone tests positive, P(disease|positive) = (0.99 × 0.01) / (0.99×0.01 + 0.05×0.99) ≈ **16.7%** — much lower than intuition suggests, because the false positive rate dominates when the base rate (prior) is low.

**Follow-up:** Why is this base-rate fallacy dangerous in real-world screening/fraud systems?

---

### Q: What is the difference between a parametric and non-parametric test? Give examples.

**Answer:**
Parametric tests assume the data follows a specific distribution (usually normal) and estimate parameters (mean, variance) — e.g., **t-test, ANOVA, Pearson correlation**. Non-parametric tests make fewer distributional assumptions and often work on ranks — e.g., **Mann-Whitney U, Wilcoxon signed-rank, Kruskal-Wallis, Spearman correlation**. Non-parametric tests are more robust to outliers/non-normality but typically have less statistical power when parametric assumptions actually hold.

---

### Q: What is multicollinearity and how do you detect and fix it?

**Answer:**
Multicollinearity occurs when two or more predictors in a regression are highly linearly correlated, making it hard to isolate each variable's individual effect — coefficients become unstable and standard errors inflate. Detect it via **correlation matrices** or **Variance Inflation Factor (VIF)**; VIF > 5–10 is typically flagged as concerning. Fixes: drop/combine correlated features, use **PCA** to decorrelate, or apply **regularization (Ridge/Lasso)**, which shrinks unstable coefficients.

---

### Q: Explain the bias-variance tradeoff from a statistical standpoint.

**Answer:**
Expected test error decomposes as **Bias² + Variance + Irreducible error**. Bias is error from overly simplistic assumptions (underfitting); variance is error from sensitivity to the specific training sample (overfitting). Simple models (e.g., linear regression) tend to have high bias, low variance; complex models (e.g., deep decision trees) have low bias, high variance. The goal is to find model complexity that minimizes the sum, often via cross-validation.

---

### Q: What's the difference between a one-tailed and a two-tailed test? When would you use each?

**Answer:**
A **two-tailed test** checks for a difference in either direction (H1: μ ≠ μ₀), splitting α across both tails. A **one-tailed test** checks for a difference in a specific direction only (H1: μ > μ₀ or μ < μ₀), putting all of α in one tail — giving more power to detect an effect in that direction, but zero power to detect an effect in the opposite direction. Use one-tailed only when a change in the "wrong" direction is truly irrelevant to the decision, which is rare in practice — most A/B tests default to two-tailed to avoid bias.

---

### Q: What is the Law of Large Numbers, and how is it different from the Central Limit Theorem?

**Answer:**
The **Law of Large Numbers (LLN)** says the sample mean converges to the true population mean as sample size grows — it's about *convergence*. The **CLT** describes the *shape* of the distribution of the sample mean around that true value (approximately normal) as n grows, and gives you the rate/spread via the standard error. LLN tells you *where* you're headed; CLT tells you *how* you get there and how much uncertainty remains at finite n.

---

### Q: How do you handle imbalanced classes when computing evaluation metrics, statistically speaking?

**Answer:**
Accuracy is misleading on imbalanced data because a naive "always predict majority class" model scores high. Prefer **precision, recall, F1, and PR-AUC** (more informative than ROC-AUC under heavy imbalance), and consider **class-weighted loss functions**, **resampling (SMOTE, undersampling)**, or **stratified cross-validation** to keep class ratios consistent across folds. Also consider whether the business cost of false positives vs. false negatives is symmetric — if not, optimize a cost-weighted metric directly rather than a generic one.

---

### Q: What is Simpson's Paradox? Give an example.

**Answer:**
Simpson's Paradox occurs when a trend appears in several different groups of data but disappears or reverses when the groups are combined — usually due to a confounding/lurking variable with unequal group sizes. Classic example: a treatment can have a *lower* success rate than a control in every age subgroup individually, yet a *higher* overall success rate, because more severe cases (lower baseline success) were disproportionately assigned to the control group. It's a strong argument for always checking results within relevant subgroups, not just in aggregate.

**Follow-up:** How would you detect Simpson's Paradox in an A/B test result?

---

### Q: Explain the difference between covariance and correlation.

**Answer:**
Covariance measures the direction of the linear relationship between two variables but its magnitude is unbounded and scale-dependent (units = product of the two variables' units), making it hard to compare across variable pairs. Correlation (Pearson's r) is covariance normalized by the product of the standard deviations, bounding it to [-1, 1] and making it scale-invariant and interpretable — 0 means no linear relationship, ±1 means perfect linear relationship.

---
