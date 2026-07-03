# 💡 Case Studies & Product Sense

[← Back to main README](../README.md)

> These are open-ended "product sense" / case-study style questions common at companies like Meta, Google, Airbnb, Uber, and Amazon. There's no single "correct" answer — what matters is **structured thinking**. Each answer below models a strong structure/approach, not a memorized script.

---

### Q: "Daily Active Users (DAU) for our app dropped 10% last week. How would you investigate?"

**Answer:**
Structure the investigation top-down:
1. **Verify it's real**: rule out a logging/instrumentation bug or reporting pipeline issue before assuming it's a real user behavior change — check if the drop appears in raw event counts, not just a dashboard.
2. **Segment the drop**: by platform (iOS/Android/web), geography, new vs. returning users, acquisition channel. A drop concentrated in one segment points to a specific cause (e.g., an app store outage, a broken release in one region).
3. **Check external factors**: app store status, server uptime/error rates, marketing spend changes, seasonality (holidays), competitor launches.
4. **Check internal changes**: recent feature releases, A/B tests that shipped, pricing/policy changes.
5. **Quantify and prioritize**: once you find candidate causes, estimate how much of the 10% each one explains before proposing a fix.

**Follow-up:** How would you distinguish a genuine user behavior shift from a measurement artifact?

---

### Q: "How would you measure the success of a new feature, say, a 'save for later' button on an e-commerce app?"

**Answer:**
Start with the **goal of the feature**: presumably to reduce cart abandonment and increase eventual conversion by giving users a lower-commitment way to bookmark items. Define a **primary metric** (e.g., conversion rate of "saved" items within N days, or overall purchase conversion rate for users who used the feature vs. didn't) and **guardrail metrics** (e.g., does it cannibalize "add to cart," does overall revenue per user change). Run as an **A/B test** rather than just observational analysis, since users who choose to use a new feature are self-selected and not directly comparable to those who don't (selection bias). Also track qualitative signals (support tickets, user feedback) for issues a metric alone might miss.

---

### Q: "You're asked to build a model to predict customer churn. Walk through your approach."

**Answer:**
1. **Define churn precisely** with the business (e.g., no purchase/login in 30/60/90 days — the definition materially changes the problem).
2. **Assemble features**: usage frequency/recency, support ticket history, billing issues, tenure, engagement trend (declining usage is often more predictive than absolute usage level).
3. **Handle class imbalance** (churners are usually the minority class) via appropriate metrics (precision/recall, PR-AUC) rather than accuracy.
4. **Avoid label leakage**: make sure features are only using information that would genuinely be available *before* the churn event (e.g., don't use "cancellation reason" as a feature to predict cancellation).
5. **Choose a model** balancing interpretability (stakeholders often want to know *why* someone is likely to churn, favoring logistic regression/tree-based models with SHAP) against raw predictive power.
6. **Tie to action**: a churn score is only useful if paired with an intervention (e.g., targeted retention offers) — evaluate the model's business impact via a held-out intervention test, not just offline AUC.

---

### Q: "How would you evaluate whether a recommendation system is actually improving user experience, beyond click-through rate?"

**Answer:**
CTR alone can reward clickbait-y recommendations that don't lead to genuine satisfaction. Layer in: **downstream engagement metrics** (watch time / read time / completion rate after the click, not just the click itself), **diversity metrics** (avoiding filter bubbles / repetitive recommendations), **long-term retention** (does exposure to recommendations correlate with continued platform use over weeks, not just immediate engagement), and **negative feedback signals** (explicit "not interested," hides, unsubscribes). Ideally validate causally via A/B test on the recommendation algorithm itself, tracking both short-term engagement and longer-term retention as a guardrail against short-term-optimized-but-long-term-harmful recommendations.

---

### Q: "A stakeholder wants to launch a feature you believe is not ready, based on your data analysis. How do you handle it?"

**Answer:**
This is testing communication and stakeholder management, not just analytics. A strong answer: present the **data clearly and objectively** (what metric moved, by how much, with what confidence), acknowledge the **business context and pressure** the stakeholder is facing, and frame the disagreement around **shared goals** rather than "I'm right, you're wrong" — e.g., propose a smaller-scope launch, a staged rollout with monitoring, or a follow-up experiment to resolve the uncertainty rather than an outright block. Ultimately, if the decision is made to launch despite your concerns, document the risk clearly and monitor the relevant guardrail metrics closely post-launch.

---

### Q: "How would you design a metric to measure 'quality' of search results for an e-commerce search bar?"

**Answer:**
Break "quality" into measurable proxies: **relevance** (does the top result match query intent — can be measured via human relevance ratings or click position of purchased items), **engagement** (click-through rate on results, but weighted toward top positions since users rarely scroll far), **conversion** (search-to-purchase rate), and **zero-result rate** (queries returning nothing, a strong negative signal). Combine a few into a composite metric or track them as a dashboard rather than a single number, since over-optimizing one proxy (e.g., CTR) in isolation can degrade others (e.g., showing popular-but-irrelevant items to farm clicks).

---

### Q: "Our conversion rate is higher on mobile than desktop, but revenue per user is lower on mobile. How would you interpret this?"

**Answer:**
This isn't necessarily contradictory — it likely reflects **basket size or average order value differences** rather than a data error: mobile users might convert more easily on cheap, quick-decision purchases (impulse buys, smaller basket) while desktop users engage in more considered, larger purchases (bigger baskets, especially for high-ticket items where a bigger screen for comparison-shopping helps). I'd verify by breaking down **average order value and units per order** by platform, and check whether this pattern holds across product categories or is concentrated in specific ones — the interpretation (and any recommendation) changes a lot depending on which explanation the data supports.

---

### Q: "How would you decide whether to build a personalized pricing model, and what risks would you flag to leadership?"

**Answer:**
Frame both the opportunity and the risk. Opportunity: potential revenue uplift by better matching price to willingness-to-pay. Risks to flag: **legal/regulatory exposure** (personalized pricing based on protected characteristics, or even proxies for them, can be illegal in many jurisdictions), **fairness/PR risk** (customers who discover they're paying more than others for the same product often react very negatively, causing reputational damage disproportionate to the revenue gain), and **measurement difficulty** (isolating true willingness-to-pay from confounded signals is hard, and errors compound directly into a trust-sensitive area). I'd recommend starting with less controversial forms — promotional targeting, dynamic pricing based on inventory/demand rather than individual user attributes — before individual-level price personalization.

---

### Q: "How would you approach forecasting next quarter's revenue for a subscription business?"

**Answer:**
Decompose revenue into components rather than forecasting one aggregate number: **existing subscriber retention/churn rate**, **new subscriber acquisition** (informed by marketing spend plans and historical CAC/conversion trends), **expansion revenue** (upsells/upgrades among existing customers), and **pricing changes** already planned. Build a **cohort-based model** (track retention curves by signup cohort) rather than a single time-series extrapolation, since cohort behavior is usually more stable and interpretable than aggregate revenue, which can mask offsetting trends (e.g., stable revenue hiding rising churn offset by rising acquisition). Present a **range** (best/base/worst case) rather than a single point forecast, and clearly state key assumptions driving the range.

---

### Q: "You launch a new feature and short-term engagement metrics improve, but you're worried about long-term effects you can't yet measure. How do you proceed?"

**Answer:**
Acknowledge the fundamental tension: **short-term metrics are measurable now; long-term effects (retention, trust, brand health) take longer to observe** and may only show up after the feature has already been broadly shipped. Mitigate by: running a **long-duration holdout** (keep a small % of users on the old experience for months, even after broader rollout, to observe delayed effects), tracking **leading indicators** that historically correlate with long-term harm (e.g., increased complaint rate, session quality metrics, not just session count), and being explicit with stakeholders that the initial "win" is a short-term read, with a plan to revisit before declaring full success.

---
