# Prioritizing mobile-money fraud review

## The decision

Before any pilot, the model inputs should be rebuilt and checked against what a risk system can know when a transaction is still open. The reproduced XGBoost model stays a benchmark. In parallel, the analyst workflow can be tested: search, ordering, assignment, notes, and a close that includes an inconclusive outcome. That is the decision this project recommends.

## The user problem

A mobile-money risk team has to choose which transactions a person opens first. Fraud in this file is rare: 8,213 of 6,362,620 rows, about 0.13 percent. Accuracy is a poor headline. Always calling a transaction legitimate is 99.87 percent accurate and catches no fraud.

The product question is how a team could prioritize reviews and still keep an acceptable level of fraud coverage. Two hypotheses are untested with users. Analysts may be able to work a score-ordered queue if the case shows its context and allows an inconclusive close. A higher cutoff, 0.90 instead of 0.50, may be an acceptable way to cut false alerts. Neither idea has been checked with fraud analysts or a risk operations lead.

## Role and approach

Nelson Kilima prepared this as an independent portfolio study for product management roles in payments, fintech, and machine learning. The work reproduces a published XGBoost workflow on synthetic PaySim data, compares two fixed cutoffs on one tuned model, and separates that benchmark from a clickable review desk. Implementation was AI-assisted. There was no customer discovery, no deployment, and no measurement of prevented fraud or money saved.

The notebook follows Stuart Walker’s article as the benchmark: the article’s feature set, an unstratified 80/20 split with seed 42, class weighting from the training labels, and a 0.90 comparison cutoff. The notebook computes its own counts. The article’s reported 74 percent precision and 96 percent recall are a reference point, not this run.

## The measured trade-off

The tuned model was scored once on 1,272,524 held-out transactions, including 1,620 fraud labels.

At 0.50 it flagged 3,335 transactions: 1,597 true positives and 1,738 false positives, and it missed 23 fraud records. Precision was 47.89 percent and recall was 98.58 percent.

At 0.90 it flagged 2,152 transactions: 1,554 true positives and 598 false positives, and it missed 66 fraud records. Precision was 72.21 percent and recall was 95.93 percent.

Moving from 0.50 to 0.90 removed 1,140 false alerts, about 65.6 percent, and 1,183 flags in total, about 35.5 percent. The cost was 43 additional missed fraud records. These flags are transactions, not unique customers and not deduplicated investigations. The 0.90 cutoff was fixed so the run could be compared with the article. It is not a threshold a business has optimized, and the scores have not been shown to be calibrated probabilities.

## A workflow that can be tested now

The prototype is a separate queue of fictional records. A reviewer can search, filter by status, and order by illustrative score or waiting time. Opening a case shows transaction context, assignment to a demo analyst, a note, and three closes: fraud, legitimate, or inconclusive. The note is required before a close. Measured 0.50 and 0.90 counts sit in another panel so they are not mixed into the sample queue. The page does not run XGBoost, label real payments, or move money.

## What changed the recommendation

In the executed run, gain importance concentrates on `newbalanceOrig`. The PaySim author warns against using `oldbalanceOrg`, `newbalanceOrig`, `oldbalanceDest`, and `newbalanceDest` for fraud detection, because detected fraudulent transactions are cancelled. This reproduction kept those fields. Preprocessing was also fitted on the full dataset before the split. Locally, every fraud label is a transfer or a cash-out: 4,097 and 4,116. That pattern belongs to this generator. It does not show that other fraud types would be caught. A random split says nothing about the next week of activity.

Those limits are why a pilot should wait for a rebuilt feature set. A valid-feature model and a chronological evaluation are described as next work. They have not been run.

## Success and the next step

The next validation step is exploratory sessions with fraud analysts and a risk operations lead, using the discovery plan and the prototype. That step succeeds if the team learns how cases are grouped, what the queue can hold, when labels become reliable, and whether 43 additional misses per this test volume would be tolerated. A later model step succeeds only after a new evaluation that avoids the warned balance fields and checks later time periods. Until then this repository is a decision record and a benchmark.
