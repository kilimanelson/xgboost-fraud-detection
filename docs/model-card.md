# Model card

## Model

XGBoost binary classifier (`binary:logistic`) trained in `XGBoost_Fraud_Detection.ipynb` on synthetic PaySim transactions. The tuned configuration selected by the saved search is `max_depth=8`, `learning_rate=0.1`, `n_estimators=200`, and `scale_pos_weight=771.04550281`. The comparison cutoff is 0.90. A 0.50 cutoff is reported on the same scores.

Owner: Nelson Kilima. Independent portfolio study. AI-assisted implementation.

## Intended use

- Explain a benchmark reproduction of a published fraud-detection workflow.
- Compare how two fixed cutoffs change transaction flags, false alerts, and missed fraud on this test split.
- Support a product discussion about review priority. The discussion artifact is `prototype/index.html`, which does not execute this model.

## Unsuitable uses

- Authorizing, blocking, or releasing live payments.
- Estimating fraud probability for a customer. Scores are not calibrated probabilities.
- Staffing a fraud desk from the flag counts. Counts are transactions in a synthetic test split, not people and not deduplicated cases.
- Claiming coverage of fraud types absent from this file. Labels occur in transfers and cash-outs only.
- Treating accuracy near 99.9 percent as evidence the classifier is safe. A no-fraud rule is already 99.87 percent accurate.
- Publishing prevented-loss or savings figures. None were measured.

## Data

PaySim synthetic mobile-money log, file `PS_20174392719_1491204439457_log.csv`, as distributed on [Kaggle](https://www.kaggle.com/datasets/ealaxi/paysim1). Simulator citation: Lopez-Rojas, Elmir, and Axelsson (2016).

6,362,620 rows. 8,213 fraud labels. Training 5,090,096 rows (6,593 fraud). Test 1,272,524 rows (1,620 fraud). Seed 42. Unstratified random 80/20 split.

Inputs: `step`, `type`, `amount`, `oldbalanceOrg`, `newbalanceOrig`, `oldbalanceDest`, `newbalanceDest`, `isFlaggedFraud`. Account names are excluded. The label is `isFraud`.

## Evaluation

Held-out test metrics for the tuned model, from the executed notebook:

| Metric | Cutoff 0.50 | Cutoff 0.90 |
|---|---:|---:|
| True positives | 1,597 | 1,554 |
| False positives | 1,738 | 598 |
| False negatives | 23 | 66 |
| True negatives | 1,269,166 | 1,270,306 |
| Flags | 3,335 | 2,152 |
| Precision | 47.89% | 72.21% |
| Recall | 98.58% | 95.93% |

Average precision 0.963761. ROC-AUC 0.999106. Cross-validation selected the configuration with training-fold fraud precision 0.527224. That figure is not the test result.

Moving from 0.50 to 0.90 on these scores: 1,140 fewer false alerts (65.6 percent), 1,183 fewer flags (35.5 percent), 43 additional missed fraud records.

## Feature warnings

- The PaySim author warns against `oldbalanceOrg`, `newbalanceOrig`, `oldbalanceDest`, and `newbalanceDest` for fraud detection, because detected fraudulent transactions are cancelled. This model uses them. Gain importance is dominated by `newbalanceOrig` (0.739).
- `isFlaggedFraud` is an existing detection flag.
- Preprocessing was fitted on all rows before the split, so scaler statistics include the test rows.
- The random split does not measure later `step` values.
- The 0.90 cutoff compares this run with the source article. It is not an optimized operating point.

## Validation still required

- Retrain without the warned balance fields, using only inputs available before completion, and fit preprocessing on training data. Not run.
- Evaluate on a later time period rather than a random split. Not run.
- Check calibration before anyone reads a score as a probability. Not run.
- Exploratory research with analysts and a risk operations lead, using `docs/discovery-plan.md`. Not run.

## Ethical and reporting notes

Synthetic data still describes a fraud-control workflow that, if copied into production without the checks above, could mis-order human review. Report the error counts beside any rate. Do not imply customer impact, fairness across demographic groups, or regulatory approval. Those analyses were not done. No code license has been selected; see the README.
