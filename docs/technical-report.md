# Technical report

This report describes the executed notebook `XGBoost_Fraud_Detection.ipynb`. Counts below come from that notebook’s saved outputs. They match the reference table for the tuned model at cutoffs 0.50 and 0.90. No training run was repeated for this write-up.

The notebook is a reproduction of the workflow in Stuart Walker’s article, [Fraud detection using XGBoost](https://www.linkedin.com/pulse/fraud-detection-using-xgboost-machine-learning-approach-stuart-walker-apeie/). The article’s final reported precision and recall, 74 percent and 96 percent at threshold 0.9, are a comparison target. This notebook calculates its own scores. The article’s threshold wording and confusion-matrix counts are inconsistent, so the tables in this repository use the notebook.

## Decision the experiment informs

The product question is how a mobile-money risk team could prioritize transaction reviews while keeping an acceptable level of fraud coverage. The measured comparison is one tuned model at two fixed cutoffs. It does not choose a launch threshold.

## Data

Source file: `data/PS_20174392719_1491204439457_log.csv`, from the [PaySim1 Kaggle distribution](https://www.kaggle.com/datasets/ealaxi/paysim1) of the PaySim simulator (Lopez-Rojas, Elmir, and Axelsson, 2016). The file is synthetic.

| Quantity | Value |
|---|---:|
| Data rows loaded | 6,362,620 |
| Fraud rows | 8,213 (0.129%) |
| Legitimate rows | 6,354,407 |
| Training rows | 5,090,096 |
| Training fraud rows | 6,593 |
| Test rows | 1,272,524 |
| Test fraud rows | 1,620 |
| Random seed | 42 |
| Test fraction | 0.20, unstratified |

A direct scan of the CSV found fraud labels only in `TRANSFER` (4,097) and `CASH_OUT` (4,116). Those two counts sum to 8,213. `PAYMENT`, `CASH_IN`, and `DEBIT` contributed no fraud labels in this file. That is a property of the synthetic generator, not a finding about other real-world fraud patterns.

Columns present in the file are `step`, `type`, `amount`, `nameOrig`, `oldbalanceOrg`, `newbalanceOrig`, `nameDest`, `oldbalanceDest`, `newbalanceDest`, `isFraud`, and `isFlaggedFraud`. The model uses nine inputs: `step`, `type`, `amount`, the four balance fields, and `isFlaggedFraud`. `nameOrig` and `nameDest` are excluded at read time. `isFraud` is the label and is not an input. Zero balances are kept. The notebook checks that numeric values are finite and nonnegative and that `type` is one of the five PaySim types.

## Preprocessing

`CASH_IN` is the reference category. The other four types become indicators. `step` and `isFlaggedFraud` are passed through. `amount` and the four balance columns are standardized with `StandardScaler`. The prepared matrix has 6,362,620 rows and 11 columns.

The scaler and encoder are fitted on the full feature table **before** `train_test_split`. That follows the article and means the scaler sees statistics from rows that later sit in the test set. Tree splits do not require standardized inputs; scaling was kept so the reproduction would match the article’s order of operations. A later evaluation should fit preprocessing inside training data only. That refit has not been done.

## Split and class weight

`train_test_split` uses `test_size=0.20` and `random_state=42` on row indices. The split is not stratified and not ordered by `step`. Training fraud count is 6,593. Test fraud count is 1,620. Those sum to 8,213.

`scale_pos_weight` is legitimate training rows divided by fraud training rows: 5,083,503 / 6,593 = 771.04550281. It is computed from training labels only.

## Models

Baseline `XGBClassifier` settings:

| Setting | Value |
|---|---|
| objective | `binary:logistic` |
| eval_metric | `logloss` |
| n_estimators | 100 |
| max_depth | 6 |
| learning_rate | 0.3 |
| scale_pos_weight | 771.04550281 |
| random_state | 42 |
| base_score | 0.5 |
| tree_method | `hist` |
| device | `cpu` |
| max_bin | 256 |
| n_jobs | up to 4 |
| importance_type | `gain` |

The baseline fit in 6.7 seconds in the saved run.

Hyperparameter search uses `RandomizedSearchCV` with 10 iterations, 3-fold cross-validation, `n_jobs=1`, and fraud precision (`precision_score`, `zero_division=0`) as the score. The test set is not passed to the search. The search space is:

- `max_depth`: 4, 6, 8
- `learning_rate`: 0.01, 0.1, 0.2
- `n_estimators`: 50, 100, 200
- `scale_pos_weight`: the training ratio only

The saved search took 163.8 seconds, refit the winner on the full training partition, and selected `max_depth=8`, `learning_rate=0.1`, `n_estimators=200`. Best cross-validation fraud precision was 0.527224. That number ranks training folds. It is not the test precision.

If `RUN_PARAMETER_SEARCH` is false, the notebook fits that same published configuration and says the search was skipped. `SAMPLE_ROWS` must stay `None` for a full-data comparison. A 100,000-row sample is an installation check only.

## Evaluation

A transaction is flagged when its score is greater than or equal to the cutoff. Confusion-matrix orientation is `[[TN, FP], [FN, TP]]`.

### Baseline at 0.50

|  | Predicted legitimate | Predicted fraud |
|---|---:|---:|
| Actual legitimate | 1,269,056 | 1,848 |
| Actual fraud | 17 | 1,603 |

Fraud precision 0.4645. Fraud recall 0.9895. Flags: 3,451.

### Tuned model at 0.50 and 0.90

| Metric | Cutoff 0.50 | Cutoff 0.90 |
|---|---:|---:|
| True positives | 1,597 | 1,554 |
| False positives | 1,738 | 598 |
| False negatives | 23 | 66 |
| True negatives | 1,269,166 | 1,270,306 |
| Flags (TP + FP) | 3,335 | 2,152 |
| Precision | 47.89% | 72.21% |
| Recall | 98.58% | 95.93% |
| F1 | 0.6446 | 0.8240 |

Arithmetic checks on the tuned model:

- 1,597 + 1,738 + 23 + 1,269,166 = 1,272,524
- 1,554 + 598 + 66 + 1,270,306 = 1,272,524
- 1,597 + 23 = 1,620 and 1,554 + 66 = 1,620
- False-alert reduction: 1,738 − 598 = 1,140, which is 1,140 / 1,738 = 65.6%
- Flag reduction: 3,335 − 2,152 = 1,183, which is 1,183 / 3,335 = 35.5%
- Additional misses: 66 − 23 = 43

Average precision on the tuned scores was 0.963761. ROC-AUC was 0.999106. Both summarize ranking across all test rows. They do not replace the cutoff counts.

Against the article’s reported 0.90 figures, this run was 72.21 percent precision and 95.93 percent recall, about 1.8 and 0.1 percentage points lower. Package versions and settings the article does not specify can move that gap. The notebook’s counts are the ones to cite.

Thresholds from 0.10 to 0.95 are tabulated in the notebook. They describe the curve. They were not used to pick a winner. `FINAL_THRESHOLD = 0.90` was set before that table was read.

## Feature importance

Gain importance on the tuned model, from the saved table:

| Feature | Gain share |
|---|---:|
| newbalanceOrig | 0.739 |
| type_PAYMENT | 0.157 |
| oldbalanceOrg | 0.035 |
| type_TRANSFER | 0.028 |
| amount | 0.016 |
| newbalanceDest | 0.009 |
| isFlaggedFraud | 0.005 |
| step | 0.004 |
| type_CASH_OUT | 0.004 |
| type_DEBIT | 0.002 |
| oldbalanceDest | 0.001 |

Gain is not a causal explanation. `newbalanceOrig` dominating the model is a reason to treat the benchmark as retrospective. The PaySim author warns against the four balance fields for fraud detection because detected fraudulent transactions are cancelled. This run kept them.

`isFlaggedFraud` is an existing detection flag, not a raw payment attribute. Class-weighted `predict_proba` scores are not calibrated fraud probabilities. The notebook states that limit beside the scoring function.

## What this run does not establish

- Live-system performance. The data are synthetic.
- Coverage of fraud types other than the transfers and cash-outs this generator labels.
- Fitness of the balance fields for authorization-time scoring.
- An unbiased estimate of test performance. Preprocessing was fitted before the split.
- Performance on later activity. The split is random, not chronological.
- A business-optimized threshold. 0.90 is a reference comparison.
- Calibrated probabilities, unique-customer workload, or prevented loss.
- A completed valid-feature rebuild or a chronological holdout. Both are future work.

## Environment

The saved run used Python 3.12 and the pins in `requirements.txt`: NumPy 2.3.5, pandas 2.2.3, SciPy 1.17.0, scikit-learn 1.7.2, XGBoost 3.0.5, Plotly 5.24.1, joblib 1.5.3, and nbformat 5.10.4. PyArrow, numexpr, and bottleneck are pinned because this NumPy 2 environment needs them for pandas to import.

Optional export (`SAVE_RESULTS = False` in the saved run) would write the model, preprocessor, and tables under `xgboost_results/`. Those artifacts are gitignored. The notebook remains the implementation.
