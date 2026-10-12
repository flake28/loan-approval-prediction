# CLAUDE.md

Persistent rules for this project. Read first. Every session.

## Project
- Course: AI for Engineers (UCS321), MST mini project.
- Problem 5: Loan approval prediction, HDFC-style bank.
- Type: supervised binary classification. Approved=1, Rejected=0.
- Dataset: `data/loan_approval_dataset.csv`, about 4,000 rows. Schema unverified. Discover it first. Strip leading spaces in names and values.
- Team: me alone.
- Full task spec: `AGENT_BRIEF.md`. Follow it phase by phase.

## Who I am
- Complete beginner in ML. Engineering student.
- I must explain every step in a viva. Teach as you build.
- End goal: I can say "I trained three classifiers, picked the best by F1, and showed which features drive approval, with evidence."

## How to talk to me
- Be objective and challenging. Question weak choices.
- Back claims with evidence: output, numbers, docs.
- Short sentences. 3-6 words where possible.
- No filler. No preamble. No pleasantries.
- Run tools first. Show result. Then stop. Minimal narration.
- After each phase: 3-5 lines on what happened and why.
- Define jargon in one line the first time.
- Connect each step to the end goal.

## Hard rules
1. Real data only. Never fabricate or substitute data silently.
2. Every number in report and slides must come from an actual run.
3. If CSV missing, stop and tell me how to get it.
4. Use `random_state=42` everywhere.
5. Fit scalers and encoders on train data only. No leakage.
6. Accuracy above 95% or one near-perfect feature: flag as possible leakage. Report honestly. Never hide it.
7. Never assume columns. Inspect the real schema before each step.
8. Ask before deleting files or installing outside this folder.
9. Never edit files outside this project folder.
10. Don't overwrite my edits. Read a file before changing it.

## Layout
```
data/      loan_approval_dataset.csv
notebook/  loan_approval.ipynb
images/    plots and flow diagram
report/    report.docx
slides/    slides.pptx
viva/      viva_qa.md
notes.md   my results summary
```

## Stack
- Python 3.10+. pandas, numpy, matplotlib, seaborn, scikit-learn.
- Docs: python-docx, python-pptx.
- Notebook runs top to bottom with no errors.

## Marking rubric
| Criterion | Marks |
|---|---|
| Problem understanding | 5 |
| Pre-processing | 10 |
| Model building | 12 |
| Evaluation | 8 |
| Innovation | 5 |

## Done means
- Notebook runs clean with saved outputs.
- Report matches notebook numbers.
- Slides, flow diagram, viva Q&A exist.
- I can answer every viva question in my own words.
- Final message: files list, 5 key results, 3 things to rehearse.
