# 🤖 Machine Learning (Classical)

[← Back to main README](../README.md)

---

### Q: Explain the bias-variance tradeoff and how it shows up in model selection.

**Answer:**
High-bias models (e.g., linear regression on non-linear data) underfit — they miss real patterns. High-variance models (e.g., unpruned decision trees) overfit — they memorize noise in the training set and generalize poorly. The tradeoff means you can't minimize both simultaneously by increasing complexity alone; instead, you tune complexity (depth, regularization strength, number of features) using a validation set or cross-validation to find the sweet spot that minimizes total generalization error.

---

### Q: How does a Random Forest differ from a single Decision Tree, and why does it usually generalize better?

**Answer:**
A Random Forest is an ensemble of decision trees trained on **bootstrapped samples** of the data, with each split considering only a **random subset of features**. This decorrelates the trees so their errors don't all point the same direction; averaging (regression) or majority voting (classification) across many decorrelated trees reduces variance substantially compared to one deep tree, without much increase in bias — this is the core idea of **bagging**.

**Follow-up:** How is Gradient Boosting different from Random Forest in terms of how it reduces error?

---

### Q: Explain Gradient Boosting at a conceptual level.

**Answer:**
Gradient Boosting builds an ensemble **sequentially**: each new weak learner (typically a shallow tree) is trained to predict the **residual errors (negative gradient of the loss)** of the current ensemble, and its predictions are added with a shrinkage factor (learning rate). Unlike bagging, which reduces variance via averaging independent models, boosting reduces bias by iteratively correcting mistakes — making it powerful but more prone to overfitting if not regularized (via learning rate, tree depth, early stopping, subsampling).

---

### Q: What's the difference between L1 (Lasso) and L2 (Ridge) regularization?

**Answer:**
Both add a penalty to the loss to shrink coefficients and reduce overfitting. **L1 (Lasso)** adds the sum of absolute values of coefficients (|w|), which tends to push some coefficients exactly to **zero**, producing sparse models useful for feature selection. **L2 (Ridge)** adds the sum of squared coefficients (w²), shrinking all coefficients smoothly toward zero but rarely to exactly zero. Elastic Net combines both when you want sparsity plus the stability L2 provides with correlated features.

---

### Q: How do you handle missing data before training a model?

**Answer:**
First diagnose the **mechanism**: MCAR (missing completely at random), MAR (missing at random, related to observed data), or MNAR (missingness related to the unobserved value itself) — this affects whether imputation is even valid. Common approaches: drop rows/columns if missingness is minimal, **mean/median/mode imputation** for simplicity, **model-based imputation** (KNN, MICE, regression) for better accuracy, or use models that handle missingness natively (XGBoost, LightGBM). Often it also helps to add a binary "was_missing" indicator feature, since missingness itself can be informative.

---

### Q: Explain the difference between bagging and boosting.

**Answer:**
**Bagging** (e.g., Random Forest) trains models **independently in parallel** on bootstrapped samples and averages/votes their predictions — primarily reduces **variance**. **Boosting** (e.g., XGBoost, AdaBoost) trains models **sequentially**, each focusing on the previous model's errors — primarily reduces **bias**, though modern implementations also control variance via regularization. Bagging is more robust to overfitting out of the box; boosting typically achieves higher accuracy but needs more careful tuning.

---

### Q: What is cross-validation, and why is k-fold preferred over a single train/test split?

**Answer:**
Cross-validation splits data into k folds, training on k−1 folds and validating on the remaining fold, rotating through all k combinations and averaging the results. This gives a more robust estimate of generalization performance than a single split, because it reduces the variance introduced by an unlucky/lucky split, and every data point gets used for both training and validation. **Stratified k-fold** is preferred for classification with imbalanced classes; **time-series/rolling-window CV** is required when data has temporal structure (to avoid leakage from the future).

---

### Q: How do you choose between precision and recall for a given business problem?

**Answer:**
It depends on the relative cost of false positives vs. false negatives. **Prioritize recall** when missing a positive is costly (e.g., cancer screening, fraud detection where fraud slipping through is expensive). **Prioritize precision** when acting on a false positive is costly (e.g., flagging a legitimate transaction as fraud and blocking a customer, or spam filters wrongly blocking important email). When both matter, use **F1-score** or optimize a threshold against a business cost function directly rather than a symmetric metric.

---

### Q: What is the curse of dimensionality, and how does it affect distance-based algorithms like KNN?

**Answer:**
As the number of features grows, the volume of the feature space grows exponentially, so data points become **sparse** and the ratio between the nearest and farthest neighbor distances tends toward 1 — meaning "nearest neighbor" becomes less meaningful. This hurts algorithms that rely on distance metrics (KNN, K-Means, clustering) and increases the risk of overfitting for any model given fixed sample size. Mitigations: dimensionality reduction (PCA, feature selection), or using tree-based/regularized models that are less distance-sensitive.

---

### Q: Explain how you would evaluate a classification model beyond accuracy.

**Answer:**
Use a **confusion matrix** to see the breakdown of TP/FP/TN/FN, then compute **precision, recall, F1**, and **ROC-AUC/PR-AUC** for threshold-independent performance. For imbalanced data prefer **PR-AUC** over ROC-AUC, since ROC can look overly optimistic when negatives vastly outnumber positives. Also examine **calibration** (do predicted probabilities match observed frequencies?) if downstream decisions depend on probability estimates, not just class labels.

---

### Q: What is feature scaling and which algorithms actually need it?

**Answer:**
Feature scaling (standardization/normalization) rescales features to comparable ranges. It matters for algorithms sensitive to feature magnitude/distance: **KNN, K-Means, SVM, PCA, and gradient-descent-based models (logistic regression, neural nets)** — unscaled features can dominate distance calculations or slow/destabilize convergence. It's generally **not required** for tree-based models (Decision Trees, Random Forest, Gradient Boosting) since splits are based on feature order, not magnitude.

---

### Q: What is the difference between generative and discriminative models?

**Answer:**
**Discriminative models** learn the decision boundary directly, modeling P(y|x) — e.g., logistic regression, SVM, most neural net classifiers. **Generative models** learn the joint distribution P(x, y) (or P(x|y) and P(y)), which lets them generate new samples and also derive P(y|x) via Bayes' rule — e.g., Naive Bayes, Gaussian Mixture Models, GANs, VAEs. Discriminative models usually perform better when you only need classification; generative models are needed when you also want to sample/generate data or handle missing features naturally.

---

### Q: How does K-Means clustering work, and what are its main limitations?

**Answer:**
K-Means randomly initializes k centroids, assigns each point to its nearest centroid, recomputes centroids as the mean of assigned points, and repeats until convergence (minimizing within-cluster sum of squares). Limitations: you must **choose k in advance** (use elbow method or silhouette score), it assumes **roughly spherical, equally-sized clusters**, it's sensitive to **initialization** (mitigated by k-means++), and it's sensitive to **outliers and feature scale**.

**Follow-up:** How would you choose k objectively rather than eyeballing an elbow plot?

---

### Q: What is the difference between a Type A (parametric) model like Logistic Regression and a non-parametric model like KNN, in terms of assumptions and scalability?

**Answer:**
Logistic Regression assumes a specific functional form (log-odds linear in features) and learns a fixed number of parameters regardless of dataset size — training is fast, prediction is O(1) per point, and it's interpretable via coefficients, but it can underfit non-linear relationships. KNN makes no assumption about functional form (non-parametric) and effectively "memorizes" the training set — it can model arbitrarily complex boundaries but scales poorly at inference time (must compare against all training points, unless using approximate structures like KD-trees) and needs careful feature scaling.

---

### Q: What is SHAP and why is it useful for model interpretability?

**Answer:**
SHAP (SHapley Additive exPlanations) assigns each feature a contribution value for a specific prediction, based on Shapley values from cooperative game theory — it fairly distributes the "credit" for a prediction across features by averaging the marginal contribution of each feature across all possible feature orderings/coalitions. It's useful because it provides both **local explanations** (why did the model predict this for this row?) and, aggregated, **global feature importance**, and it's more theoretically grounded and consistent than simpler importance metrics like Gini importance.

---

### Q: Explain overfitting and list five concrete ways to prevent it.

**Answer:**
Overfitting is when a model captures noise/idiosyncrasies of the training set rather than the true underlying pattern, leading to strong training performance but poor generalization. Mitigations: (1) **more training data**, (2) **regularization** (L1/L2, dropout), (3) **cross-validation** with early stopping, (4) **reducing model complexity** (fewer features/parameters, shallower trees), (5) **ensembling** (bagging averages out variance) and/or data augmentation.

---

### Q: What's the difference between one-hot encoding and label/ordinal encoding, and when would you use each?

**Answer:**
**One-hot encoding** creates a binary column per category — appropriate for **nominal** (unordered) categorical variables, since it avoids implying a false ordinal relationship, but it increases dimensionality significantly with high-cardinality features. **Label/ordinal encoding** maps categories to integers — appropriate for **ordinal** variables where order is meaningful (e.g., "low/medium/high"), or for tree-based models which can handle arbitrary integer encodings reasonably well without implying a linear relationship the way linear models would.

---

### Q: How would you detect and handle outliers in a dataset before modeling?

**Answer:**
Detection: visualize with **box plots/scatter plots**, use statistical rules like **IQR (1.5×IQR beyond Q1/Q3)** or **z-score thresholds (|z| > 3)**, or model-based methods like **Isolation Forest / DBSCAN** for multivariate outliers. Handling depends on cause: if it's a data error, **fix or remove** it; if it's a genuine extreme value, consider **capping/winsorizing**, **transforming** (log transform to reduce skew), or using **robust models** (tree-based methods, robust regression) that are naturally less sensitive to outliers rather than blindly deleting valid data.

---

### Q: What is the ROC curve, and what does AUC actually represent?

**Answer:**
The ROC curve plots **True Positive Rate (recall)** against **False Positive Rate** at every possible classification threshold. **AUC (Area Under the Curve)** represents the probability that a randomly chosen positive example is ranked higher (by the model's score) than a randomly chosen negative example — an AUC of 0.5 means random guessing, 1.0 means perfect ranking. AUC is threshold-independent, which makes it useful for comparing models, but it can be misleadingly optimistic on heavily imbalanced datasets where PR-AUC is more informative.

---
