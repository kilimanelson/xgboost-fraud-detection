# Discovery plan

Status: **planned**. No sessions have been held. This document does not contain participants, quotations, or findings.

Audience for the sessions: fraud analysts who close transaction reviews, and one risk operations lead who owns queue capacity. Recruit a small set from a real program only with that organization’s permission. Until then, use the prototype as a discussion prop, and keep every example labeled illustrative.

Goal: learn whether a score-ordered transaction queue is a workable way to prioritize reviews, and what evidence would be required before testing a higher cutoff. The sessions do not validate the XGBoost model.

## How to run a session

Plan 45 to 60 minutes. One interviewer, one note-taker. Record only with consent. Start by saying the records on screen are fictional, the scores are not from the model, and nothing will block or release a payment.

Use the first half for the current job. Use the second half for the prototype tasks. Leave ten minutes to ask what would make the person distrust the queue.

## Current investigations and queue priority

Ask these in the analyst sessions. Follow answers with “what happened last time,” not with a pitch.

1. Walk through the last investigation you closed. What did you open first, and what did you ignore?
2. What arrives in the queue today: a transaction, a customer, a pair of transfer and cash-out legs, or something else?
3. How do you decide what is next when many items are waiting? Score, amount, age, customer risk, or a rule a colleague taught you?
4. Which fields do you trust at the moment you start a review? Which fields are misleading because the transaction was later cancelled or reversed?
5. When do you mark something inconclusive instead of fraud or legitimate? What information is usually missing?
6. How often is the label you would want unavailable until another team, a chargeback, or a later complaint?

Ask the operations lead:

1. How many reviews can the desk finish in a shift, and what share of flagged items age out?
2. Are two flags for the same customer counted as one investigation or two?
3. What miss rate, in transactions per day or per week, would be unacceptable even if false alerts fell?
4. Who is allowed to change a review cutoff, and what evidence do they require?

## Case grouping, capacity, and label maturity

Listen for three distinctions. Do not fill them in from the PaySim file.

- **Case grouping.** Analysts may need the matching cash-out and transfer together. A transaction-level flag count can overstate the work.
- **Capacity.** The benchmark’s drop from 3,335 to 2,152 test flags is a transaction count on a synthetic test set. It is not a staffing plan. Ask what the desk can actually clear.
- **Label maturity.** `isFraud` in PaySim is a simulator label. In a live program, ask when a review outcome becomes stable enough to train on, and which outcomes must stay out of training.

## Prototype tasks

Open `prototype/index.html`. Tell the participant the queue is fictional and the benchmark panel is a separate, already-computed experiment.

1. Find the unreviewed item with the highest illustrative score and open it. Note whether they use sort, search, or scanning.
2. Assign it to a demo analyst.
3. Try to record a disposition before writing a note. The prototype should block that action. Ask whether a forced note matches their job.
4. Write a note and close the case as inconclusive. Ask what “inconclusive” should do to the customer and to the model label.
5. Order the remaining queue by waiting time. Ask which item they would open next, and why that differs from score order.
6. Read the benchmark panel. Ask them to say, in their own words, whether those counts are the cases on screen. The expected answer is that they are not.

Stop the task if the person thinks the page can freeze a real account. Correct that immediately.

## Evidence log

Copy one row per observation. Leave the sheet empty until a session happens. Keep observation, interpretation, and product decision in different columns so a quote is not quietly turned into a roadmap item.

| Date | Session | Observation (what was said or done) | Interpretation (what we think it means) | Product decision (what changes) | Challenged hypothesis |
|---|---|---|---|---|---|
|  |  |  |  |  |  |

## Hypotheses and what would challenge them

These are starting hypotheses, not results.

| Hypothesis | Evidence that would challenge it |
|---|---|
| Analysts can prioritize from a transaction score if the case shows type, amount, balances, and wait time. | They refuse to work an item until a related transaction, device, or customer history is attached. |
| A required note before fraud, legitimate, or inconclusive is acceptable. | Notes are written after the close, or a forced note produces filler text they do not trust. |
| Ordering by waiting time will surface different work than ordering by score, and analysts want both. | The two orders pick the same cases in their real queue, or they will not use a score sort at all. |
| Reducing false alerts by moving from 0.50 to 0.90 is desirable if recall stays near 96 percent on a benchmark like this one. | The operations lead treats the extra misses (43 on this test set) as unacceptable, or the desk is not capacity-constrained. |
| Balance fields that update after a cancelled fraud are a poor fit for the review screen. | Analysts rely on those post-transaction balances and can explain a legitimate operational use. |
| Inconclusive must remain a first-class outcome. | Reviewers say policy forces a binary close, and an inconclusive state would be gamed or ignored. |

If a hypothesis is challenged, record that in the evidence log. Do not update the model or the cutoff from a single session.
