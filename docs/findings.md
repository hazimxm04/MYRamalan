# Other findings

_Moved out of the main [README](../README.md) to keep it short._

**Outcome leakage, found and fixed (second leak).** Separately from the sentiment leak below, some structural features
(majority change, turnout change, and similar) were computed from the election being predicted, so the model was partly
shown the answer. Now every feature uses only the last election before the predicted one, and the one before that
(`backend/core/features.py`), elections before 2004 are excluded because the seat counts differed, and seats are matched
on their number (N.07) rather than their name. A test (`tests/test_no_outcome_leakage.py`) rewrites the predicted
election and fails if any feature changes. The evaluation numbers in the [README](../README.md) come from the fixed pipeline.

**Why accuracy is no better than the simple rules while AUC is high.** The model ranks seats well (AUC 0.93 in Johor,
0.99 in Negeri Sembilan), but its accuracy is no better than the simple rules "never opposition" and "same as last
time". Accuracy depends on the 0.5 probability cut-off, and because the training years include the 2018 opposition
wave, the model over-predicts opposition wins in 2026: 12 of the 13 errors were false alarms. Next, I will choose the
cut-off using only earlier elections, never the 2026 results, and add a per-seat feature for whether both BN and PN
field candidates.

**Pooled model across states, inconclusive.** One model trained on all five states (with a time-ordered test that never
uses the test year or later) got more seats right in Selangor, Perak and Negeri Sembilan, fewer in Melaka, and about the
same in Johor (`backend/models/pooled_results.json`). Seats in one election swing together, so there are far fewer
independent observations than seats; no improvement is claimed.

**Temporal leakage, found and fixed.** An earlier version of this
pipeline included prediction-period (Aug 2026) sentiment scores as a
*training* feature for transitions ending in 2013, 2018, and 2022 —
using information that did not exist yet at the time of those
elections. Fixed by splitting the feature set: training uses only
structural + ethnicity features; sentiment/economic features are
reserved for prediction time, where they are temporally valid.

**3-class coalition split investigated, and reverted.** Tested
splitting the target into BN / Pakatan / PN instead of binary. Found
it underperforms: PN only emerged as a coalition in 2020, so no
training transition has PN as a "previous" winner, and PN's seat
count in its one possible training transition varies sharply by
state (2 seats in Melaka vs. 26 in Perak) — a genuine data
limitation, not a tuning problem (confirmed via GridSearchCV, which
improved cross-validation scores but produced zero change in real
validation accuracy). Reverted to binary; full investigation
documented rather than discarded.

**Voter-roll demographics provide genuine seat-level signal.**
Ethnicity and age composition (`chinese_pct`, `young_malay_pct`,
`youth_pct`, etc.), sourced from anonymised voter rolls, are
consistently among the top features by importance across all 5
states — this is real, seat-varying signal that structural features
alone (majority change, turnout, incumbency) don't capture.

**Sentiment/narrative features are national, weighted by
demographics.** State-level sentiment scores are constant across all
seats in a state (zero tree importance on their own). Weighting
national narrative themes (e.g., "cost of living," "Islam threat")
by each seat's demographic composition produces genuine seat-level
variation — different seats respond differently to the same national
narrative. This is an idea that is **not in the current models**: it was
kept out of training to avoid the sentiment leakage described below, and
the live predictor currently holds these inputs at 0. It needs a retrain
with real seat-level values and a fair held-out test before any claim. A separate, assumption-based version that does not
touch the models is described under *Experimental sentiment layer* in the [README](../README.md).
