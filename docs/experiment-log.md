# Experiment Log

## Setup
- **Data:** NEPSE index daily history from the Nepal Market Data project (MIT licensed); not redistributed here. 6,731 sessions, 1997-07-20 to 2026-11-13.
- **Analysis window:** 2018-11-11 onward. Before late 2018 the open, high and low repeat the close, so no real daily range exists.
- **Target:** 1 if the next session's close is higher than today's, else 0 (the single flat session counts as 0).
- **Features (known at the close of session t):** today's return, returns of the two previous sessions, daily range as % of close.
- **Split:** chronological 80/20. Train: 2018-11-14 to 2025-03-30 (1,460 sessions). Test: 2025-04-01 to 2026-11-06 (365 sessions).
- **Class balance (test):** 53.2% down, 46.8% up.

## Data quality notes
- 27 sessions (about 1.5%) have an open or close outside the recorded high–low range; one (2019-01-09) has a high below its low. Rows kept to preserve the time sequence; daily range computed from the max and min of the four values.
- Our close-to-close returns agree with the source's `per_change` (mean absolute difference about 0.004 percentage points).
- The trading week changed from Sunday–Thursday to Monday–Friday in 2026 (last Sunday session: 2026-04-05). Features are shifted by rows, not calendar days.

## Experiment 1: Baseline (always predict the most common training class)
Accuracy 0.5315, precision 0, recall 0, F1 0. Useless at spotting up days despite 53% accuracy.

## Experiment 2: Logistic Regression
Accuracy 0.5452, precision 0.525, recall 0.310, F1 0.390. Confusion matrix [[146, 48], [118, 53]]. Coefficients tiny with inconsistent signs: little evidence of signal.

## Experiment 3: Decision Tree (max_depth=3) and Random Forest (200 trees, max_depth=5, min_samples_leaf=20)
| Model | Accuracy | Precision | Recall | F1 | Train acc |
|---|---|---|---|---|---|
| Decision Tree | 0.5589 | 0.539 | 0.404 | 0.462 | 0.612 |
| Random Forest | 0.5753 | 0.558 | 0.450 | 0.498 | 0.655 |

The gap between training and test accuracy shows mild overfitting. Hyperparameters were not tuned.

## Experiment 4: Expanding-window time-series cross-validation (5 folds)
| Model | F1 | F2 | F3 | F4 | F5 | Mean |
|---|---|---|---|---|---|---|
| Baseline | 0.408 | 0.421 | 0.546 | 0.516 | 0.523 | 0.483 |
| Logistic Regression | 0.487 | 0.549 | 0.526 | 0.507 | 0.536 | 0.521 |
| Decision Tree | 0.559 | 0.576 | 0.543 | 0.599 | 0.536 | 0.563 |
| Random Forest | 0.566 | 0.536 | 0.579 | 0.638 | 0.553 | 0.574 |

The Random Forest beats the baseline in all five folds. However, the baseline is hurt by shifts in the up/down mix between training and test periods (folds 1 and 2). Against the majority class within each test fold (0.592, 0.579, 0.546, 0.516, 0.523), the Forest wins folds 3, 4 and 5 by roughly 3, 12 and 3 points and loses folds 1 and 2.

## Conclusions
- Past returns and daily range give at most a small, inconsistent edge over a naive guess.
- More flexible models (Tree, Forest) did somewhat better than Logistic Regression, unlike the gold-price project where the simple baseline won.

## Limitations
- Accuracy differences of 2–4 points on 365 test days are close to what luck alone can produce (about ±2.6 points).
- Three models were compared on the same test set, which slightly flatters the best one.
- Not a trading strategy: no transaction costs, and the index cannot be bought directly.
- Only four features; no macro, news, or sector information.

## Possible next steps
- Add more lags and rolling averages (e.g. 5-day mean return, 5-day volatility) and test whether they help.
- Add turnover (`traded_amount`), which is real from 2016.
- Repeat the experiment on a single stock such as NABIL, once corporate actions are handled.