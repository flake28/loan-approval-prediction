# Phase 1: Schema Discovery

Shape: 4269 rows, 13 columns

## Columns & dtypes

|                          | dtype   |
|:-------------------------|:--------|
| loan_id                  | int64   |
| no_of_dependents         | int64   |
| education                | str     |
| self_employed            | str     |
| income_annum             | int64   |
| loan_amount              | int64   |
| loan_term                | int64   |
| cibil_score              | int64   |
| residential_assets_value | int64   |
| commercial_assets_value  | int64   |
| luxury_assets_value      | int64   |
| bank_asset_value         | int64   |
| loan_status              | str     |


## Missing values per column

|                          |   missing |
|:-------------------------|----------:|
| loan_id                  |         0 |
| no_of_dependents         |         0 |
| education                |         0 |
| self_employed            |         0 |
| income_annum             |         0 |
| loan_amount              |         0 |
| loan_term                |         0 |
| cibil_score              |         0 |
| residential_assets_value |         0 |
| commercial_assets_value  |         0 |
| luxury_assets_value      |         0 |
| bank_asset_value         |         0 |
| loan_status              |         0 |


Duplicate rows: 0


## Target value counts (loan_status)

| loan_status   |   count |   pct |
|:--------------|--------:|------:|
| Approved      |    2656 | 62.22 |
| Rejected      |    1613 | 37.78 |


## Numeric describe()

|       |   loan_id |   no_of_dependents |     income_annum |      loan_amount |   loan_term |   cibil_score |   residential_assets_value |   commercial_assets_value |   luxury_assets_value |   bank_asset_value |
|:------|----------:|-------------------:|-----------------:|-----------------:|------------:|--------------:|---------------------------:|--------------------------:|----------------------:|-------------------:|
| count |    4269   |         4269       |   4269           |   4269           |  4269       |      4269     |             4269           |            4269           |        4269           |     4269           |
| mean  |    2135   |            2.49871 |      5.05912e+06 |      1.51335e+07 |    10.9004  |       599.936 |                7.47262e+06 |               4.97316e+06 |           1.51263e+07 |        4.97669e+06 |
| std   |    1232.5 |            1.69591 |      2.80684e+06 |      9.04336e+06 |     5.70919 |       172.43  |                6.50364e+06 |               4.38897e+06 |           9.10375e+06 |        3.25019e+06 |
| min   |       1   |            0       | 200000           | 300000           |     2       |       300     |          -100000           |               0           |      300000           |        0           |
| 25%   |    1068   |            1       |      2.7e+06     |      7.7e+06     |     6       |       453     |                2.2e+06     |               1.3e+06     |           7.5e+06     |        2.3e+06     |
| 50%   |    2135   |            3       |      5.1e+06     |      1.45e+07    |    10       |       600     |                5.6e+06     |               3.7e+06     |           1.46e+07    |        4.6e+06     |
| 75%   |    3202   |            4       |      7.5e+06     |      2.15e+07    |    16       |       748     |                1.13e+07    |               7.6e+06     |           2.17e+07    |        7.1e+06     |
| max   |    4269   |            5       |      9.9e+06     |      3.95e+07    |    20       |       900     |                2.91e+07    |               1.94e+07    |           3.92e+07    |        1.47e+07    |


## Unique counts for text columns

- education: ['Graduate', 'Not Graduate']

- self_employed: ['No', 'Yes']


---

# Phase 2: Cleaning summary

- Dropped `loan_id` (not predictive, just a row label).
- 0 duplicate rows found.
- Mapped target: Approved=1, Rejected=0.
- Found 28 rows with impossible negative `residential_assets_value` (min -100000). Fixed by clipping to 0 (negative asset value cannot exist in reality).
- IQR outlier scan: some asset columns show outliers (e.g. 52 in residential_assets_value, 37 in commercial_assets_value) but these are plausible high-net-worth applicants, not data errors. Kept as-is.
- Engineered 4 features: TotalAssets, Loan_to_Income, Assets_to_Loan, EMI_style. All supported by the real schema.

# Phase 3: Interpretation (real numbers from the executed notebook)

**Rows / columns / approval rate:** 4,269 rows, 13 raw columns (+4 engineered + 1 one-hot). Approval rate: 62.22% Approved, 37.78% Rejected.

**Top driver of approval:** `cibil_score`, by a wide margin.
- Built-in (Gini) importance: 0.7959 (next closest is loan_term at 0.0437).
- Permutation importance: 0.3438 (next closest is loan_term at 0.0444).
- Both methods agree: cibil_score dominates; loan_term and the engineered Loan_to_Income ratio are distant secondary signals.
- A single-feature rule "cibil_score >= 550 -> Approved" alone gets 95.36% accuracy on the raw data. Approved applicants average cibil_score 703.5 (std 125.2); Rejected average 429.5 (std 78.4) -- almost no overlap between the two groups.

**Model comparison (test set, 20% holdout, stratified):**

| Model | CV F1 (5-fold) | Train Acc | Test Acc | Precision | Recall | F1 | ROC-AUC |
|---|---|---|---|---|---|---|---|
| Logistic Regression | 0.9396 | 0.9283 | 0.9356 | 0.9704 | 0.9247 | 0.9470 | 0.9785 |
| Decision Tree (max_depth=4) | 0.9993 | 1.0000 | 0.9988 | 0.9981 | 1.0000 | 0.9991 | 0.9985 |
| Random Forest (300 trees) | 0.9981 | 1.0000 | 0.9977 | 0.9962 | 1.0000 | 0.9981 | 1.0000 |
| Random Forest (tuned, GridSearchCV) | 0.9981 | 1.0000 | 0.9965 | 0.9944 | 1.0000 | 0.9972 | 1.0000 |

Best tuned params: max_depth=None, min_samples_split=2, n_estimators=200.

**Best model and why:** By raw test F1, the plain Decision Tree (0.9991) is actually the single best model -- it edges out even the tuned Random Forest (0.9972). This makes sense: the problem is nearly linearly separable by cibil_score alone, so a shallow tree captures almost all the signal, and ensembling/tuning adds little. For the feature-importance and ablation analysis we used the tuned Random Forest (per the project's evaluation pipeline), since it gives both Gini and permutation importance naturally and is the more standard choice to present, while the report/viva will note the Decision Tree's competitive result as a genuine finding, not an error.

**Overfitting (train vs test gap):**
- Logistic Regression: train 0.9283 vs test 0.9356 -- essentially no gap (test even slightly higher, fine given dataset size).
- Decision Tree: train 1.0000 vs test 0.9988 -- gap 0.0012. Negligible.
- Random Forest: train 1.0000 vs test 0.9977 -- gap 0.0023. Negligible.
- Random Forest (tuned): train 1.0000 vs test 0.9965 -- gap 0.0035. Negligible.
All models fit training data perfectly or near-perfectly, but test performance stays close -- the small gaps say this isn't classic overfitting, it's a near-separable dataset.

**Confusion matrix (tuned Random Forest, test set, n=854):**
- True Negatives (correctly Rejected): 320
- False Positives (wrongly Approved): 3
- False Negatives (wrongly Rejected): 0
- True Positives (correctly Approved): 531

Wrongly-approved (3 cases) is the costlier error for a bank: approving someone who should be rejected risks the bank's money not being repaid. Wrongly-rejected (0 cases here) only costs a missed business opportunity, not a loss. The model happens to make zero of the costly-direction errors on this test set and only 3 of the cheaper-direction, but with a sample this small (n=854) that is not something to over-claim robustness from.

**Ablation result:** Removing `cibil_score` (the top feature) and retraining the same tuned Random Forest dropped F1 from 0.9972 to 0.6616 -- a 0.3356 absolute drop (33.7% relative). This is strong, direct evidence that the model's near-perfect accuracy rests almost entirely on one column.

**Limits:**
- Dataset is very likely synthetic or heavily simplified -- a near-deterministic cibil_score threshold rule is not how real-world bank underwriting behaves (real approvals weigh many soft factors together).
- No time dimension -- can't tell if this reflects one bank's policy at one point in time.
- No fairness audit performed -- education and self_employed show flat approval rates across groups here, which is reassuring, but a full fairness audit (disparate impact testing, subgroup calibration) was out of scope.
- Possible proxy bias: income, assets, and employment status may correlate with protected characteristics (e.g. caste, religion, region, gender) not present in this dataset -- real deployment would need that audit before use.
