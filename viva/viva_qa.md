# Viva Q&A — Loan Approval Prediction

Answers use real numbers from `notebook/loan_approval.ipynb`. Learn these in your own words.

---

**1. Why classification, not regression?**
The target (`loan_status`) is one of two categories — Approved or Rejected — not a continuous number. Regression predicts a number (like a price); classification predicts a category. We mapped Approved=1, Rejected=0.

**2. Why drop the ID column (`loan_id`)?**
It's just a sequential row label (1, 2, 3...). It has no real relationship to creditworthiness, so including it would add noise, not signal.

**3. Why median for numeric, mode for categorical missing values?**
Trick question for this dataset: there were **zero missing values**, so no imputation was needed. The rule is there in case a real-world version of this data had gaps — median resists outliers better than mean for numeric data; mode is the only sensible "average" for categories.

**4. Class imbalance: is it present here, how handled?**
Mildly — 62.2% Approved vs 37.8% Rejected. Not extreme, but we still used `class_weight='balanced'` on every model so the majority class doesn't dominate training.

**5. Why stratified split?**
`train_test_split(..., stratify=y)` keeps the same Approved/Rejected ratio in both train and test sets. Without it, a random split could accidentally put too many of one class in the test set, making evaluation unreliable.

**6. What is data leakage; how does the Pipeline prevent it?**
Leakage is when information from outside the training data (often the test set) sneaks into training, making results look better than they'd be in the real world. We wrapped `StandardScaler` + the classifier in one `sklearn.Pipeline`, and only called `.fit()` on training data — so the scaler's mean/std come only from train, never peek at test.

**7. Precision vs recall — which matters more for a bank?**
Precision: of everyone we approved, how many actually deserved it? Recall: of everyone who deserved approval, how many did we catch? For a bank, a **false approval** (approving someone who shouldn't be) risks losing money — so precision on the "Approved" class matters a lot. A false rejection just loses potential business, a cheaper mistake.

**8. What is ROC-AUC?**
A single number (0 to 1) summarizing how well the model separates the two classes across all possible decision thresholds, not just one cutoff. 1.0 = perfect separation, 0.5 = random guessing. Our best models scored very close to 1.0.

**9. How do you detect overfitting?**
Compare train accuracy vs test accuracy. A big gap (model memorized train data, fails on new data) signals overfitting. Here: Decision Tree train=1.0000 vs test=0.9988 (gap 0.0012), Random Forest train=1.0000 vs test=0.9977 (gap 0.0023) — both tiny. Not overfitting in the usual sense; the problem is just close to linearly separable.

**10. Why this final model?**
By raw test F1, the plain Decision Tree (0.9991) slightly beats even the tuned Random Forest (0.9972). Tuning the ensemble added almost nothing because `cibil_score` alone nearly solves the problem — a simple model captures it as well as a complex one. We report both honestly rather than picking whichever looks most "advanced."

**11. What does feature importance show? Why check permutation importance too?**
Built-in (Gini) importance says `cibil_score` = 0.7959 of total importance — by far the largest. But Gini importance can be biased toward numeric/high-cardinality features. So we cross-checked with **permutation importance** (shuffle one column, see how much F1 drops) on the test set — it agrees: `cibil_score` = 0.3438, still the clear #1. Two independent methods agreeing is stronger evidence than one.

**12. What did the ablation show?**
We removed `cibil_score` and retrained from scratch. F1 dropped from 0.9972 to 0.6616 — a 33.7% relative drop. This proves the model isn't using a broad mix of signals; it's leaning almost entirely on one column.

**13. Why is the dataset's top feature so dominant, and is that realistic?**
A simple rule, "cibil_score ≥ 550 → Approved," already gets 95.4% accuracy alone. Approved applicants average a 703 credit score; Rejected average 429 — almost no overlap. This strongly suggests the dataset is synthetic or built around a near-deterministic rule. Real-world bank underwriting weighs many softer factors together, so this level of single-feature dominance is **not fully realistic** — a genuine limitation to flag, not hide.

**14. How could bias enter this model?**
Even though `education` and `self_employed` show flat approval rates here (not predictive), features like income, assets, or employment type could act as **proxies** for protected characteristics (e.g. gender, region, caste, religion) not present in this dataset. No formal fairness audit was done — that's a real gap before any real-world deployment.

**15. What would you do with more time?**
Try gradient-boosted models (XGBoost/LightGBM) for comparison, run a formal fairness/disparate-impact audit, test on real (non-synthetic) bank data, and explore SMOTE-style resampling even though imbalance here was mild.
