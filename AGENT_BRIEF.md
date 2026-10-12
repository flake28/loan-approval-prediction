# AGENT BRIEF v2: Loan Approval Prediction (AI for Engineers, MST Mini Project, Problem 5)

## Role
You are my mentor and executor. I am a beginner with no ML experience. I work alone.
Build the full project in this folder. Teach me while you build. Keep explanations short and plain.

## Goal
Predict loan approval for an HDFC-style bank. Supervised binary classification.
Deliver: working notebook, report, slides, flow diagram, viva Q&A.

## Dataset (READ CAREFULLY)
- File: `data/loan_approval_dataset.csv`
- Size: about 4,000 ROWS. I wrote "4k columns" by mistake. If the file truly has about 4,000 columns, STOP and tell me. That is a different problem.
- I have NOT verified the schema. Do not assume. Discover it in Phase 1.
- Likely schema (a common Kaggle file with this name, about 4,269 rows, 13 columns). Treat as a hypothesis only:
  `loan_id, no_of_dependents, education, self_employed, income_annum, loan_amount, loan_term, cibil_score, residential_assets_value, commercial_assets_value, luxury_assets_value, bank_asset_value, loan_status`
  Target `loan_status` likely has text values "Approved" / "Rejected".
- Known quirk in that file: column names and text values may have LEADING SPACES (e.g. `" loan_status"`, `" Approved"`). Strip them first: `df.columns = df.columns.str.strip()` and strip all string cells.
- Do NOT reuse the old assumptions (Credit_History, Property_Area, Gender, Married, CoapplicantIncome). They may not exist here.

## Hard rules
1. Real data only. Never invent numbers or fake rows.
2. If the CSV is missing or unreadable, stop and tell me.
3. Every figure in report and slides must come from an actual run.
4. Set `random_state=42` everywhere.
5. Discover the schema, then adapt every step to the real columns. Show me the schema before modelling.
6. Leakage check: if any single feature predicts the target almost perfectly, or accuracy exceeds 95%, flag it and explain before continuing. Do not hide a strong result. Report it honestly and discuss.
7. Fit scalers and encoders on train data only (use a Pipeline).
8. After each phase, give a 3-5 line "what happened and why" note.
9. Ask before deleting files or installing outside this folder.

## Folder layout
```
Project/
  data/        loan_approval_dataset.csv
  notebook/    loan_approval.ipynb
  images/      eda.png, heatmap.png, evaluation.png, importance.png, flow.png
  report/      report.docx
  slides/      slides.pptx
  viva/        viva_qa.md
  notes.md
```
Create missing folders first.

## Phase 1: Setup and schema discovery
- Check Python 3.10+. Install: pandas numpy matplotlib seaborn scikit-learn jupyter ipykernel python-docx python-pptx.
- Load the CSV. Strip column names and string cells.
- Print and save to `notes.md`: shape, column names, dtypes, missing counts per column, duplicate count, target value counts and percentages, `describe()` for numeric columns, unique counts for text columns.
- Decide and tell me:
  - Which column is the ID (drop it).
  - Which is the target. Map to 1=Approved, 0=Rejected.
  - Which columns are numeric vs categorical.
  - Class balance. Is it imbalanced?
- Pause and show me this summary before Phase 2.

## Phase 2: Notebook (`notebook/loan_approval.ipynb`)
Markdown header above each cell group.
1. Imports.
2. Load and strip. Show shape, `info()`, missing, class balance.
3. Clean:
   - Drop ID and duplicates.
   - Missing values: median for numeric, mode for categorical. If none exist, say so. Do not invent a problem.
   - Check negative or impossible values (e.g. negative assets, zero loan term). Report and fix.
   - Check outliers with IQR. Keep them unless clearly errors. Explain why.
4. EDA (save `images/eda.png`, `images/heatmap.png`): class balance; approval rate by education and self_employed; distribution of the strongest numeric feature by status (likely cibil_score); boxplots of income and loan amount by status; correlation heatmap. One written insight per plot.
5. Feature engineering (adapt to real columns, tell me what you built and why):
   - TotalAssets = sum of all asset-value columns.
   - Loan_to_Income = loan_amount / income_annum.
   - Assets_to_Loan = TotalAssets / loan_amount.
   - EMI-style = loan_amount / loan_term.
   - Skip any that the real schema cannot support.
6. One-hot encode text columns (`drop_first=True`). Stratified 80/20 split. Pipeline with StandardScaler.
7. Train Logistic Regression, Decision Tree (max_depth=4), Random Forest (300 trees). Use `class_weight='balanced'`. Report 5-fold CV F1, test accuracy, precision, recall, F1, ROC-AUC, train accuracy. Show a results table.
8. Tune Random Forest with GridSearchCV (scoring F1).
9. Evaluate best model: classification report, confusion matrix, ROC for all models (save `images/evaluation.png`).
10. Feature importance, top 10 (save `images/importance.png`). Add a second check: permutation importance on the test set.
11. Ablation (innovation marks): retrain the best model WITHOUT the top feature. Report how much F1 drops. This shows how dependent the model is on one signal.
12. Conclusion cell with REAL numbers.

Run end to end. Fix errors. Save with outputs.

## Phase 3: Interpretation (`notes.md`)
Answer with real numbers:
- Rows, columns, approval rate.
- Top 3 drivers of approval, with evidence from both importance methods.
- Best model and why (justify from the table).
- Overfitting: train vs test gap per model.
- Confusion matrix: wrongly approved count and wrongly rejected count. Which error costs a bank more and why.
- Ablation result.
- Limits: dataset likely synthetic or simplified, no time dimension, no fairness audit, possible proxy bias via education or self-employment.

## Phase 4: Flow diagram (`images/flow.png`)
Matplotlib or graphviz. Boxes:
Raw data -> Strip and clean -> EDA -> Feature engineering -> Encode -> Split 80/20 -> Scale -> Train 3 models -> Tune -> Evaluate -> Importance and ablation.

## Phase 5: Report (`report/report.docx`, 6-8 pages)
1. Problem statement and objective
2. Dataset (source, shape, column table with type and meaning)
3. Pre-processing (table: issue, fix, reason)
4. EDA (embed plots, one insight each)
5. Models (why these three)
6. Results (metrics table, confusion matrix, ROC)
7. Innovation (engineered features, class weighting, tuning, ablation)
8. Conclusion, limits, future work (XGBoost, SMOTE, fairness audit, real bank data)
9. References (Kaggle dataset, scikit-learn docs)

Plain language. Real numbers only. Leave placeholders for name, roll number, course code.

## Phase 6: Slides (`slides/slides.pptx`, 8 slides)
Title, Problem, Data, Flow diagram, EDA, Models, Results, Conclusion. Large fonts. One idea per slide.

## Phase 7: Viva (`viva/viva_qa.md`)
15 questions, 2-3 line answers tied to MY results. Include:
- Why classification, not regression?
- Why drop the ID column?
- Why median vs mode?
- Class imbalance: is it present here, how handled?
- Why stratified split?
- What is data leakage; how does the Pipeline prevent it?
- Precision vs recall for a bank.
- What is ROC-AUC?
- How do you detect overfitting?
- Why this final model?
- What does feature importance show? Why check permutation importance too?
- What did the ablation show?
- Why is the dataset's top feature so dominant, and is that realistic?
- How could bias enter this model?
- What would you do with more time?

## Rubric to satisfy
| Criterion | Marks | Where |
|---|---|---|
| Problem understanding | 5 | Report 1, Slide 2 |
| Pre-processing | 10 | Notebook 3-6, Report 3-4 |
| Model building | 12 | Notebook 7-8 |
| Evaluation | 8 | Notebook 9-10 |
| Innovation | 5 | Notebook 5, 8, 11 |

## Definition of done
- Notebook runs top to bottom with no errors.
- Every number in report and slides matches notebook output.
- All files exist.
- I can answer every `viva_qa.md` question in my own words.
- Final message: files list, 5 key results, 3 things to rehearse.

## Teaching style
Short sentences. Define jargon once. Tie each step to the end goal: "one dominant signal drives approval, and my model proves it, with evidence."
