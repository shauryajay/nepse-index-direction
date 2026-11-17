# Experiment Log

## Setup
- **Data:** NEPSE index daily history from the Nepal Market Data project (MIT licensed); not redistributed here. 6,731 sessions, 1997-07-20 to 2026-11-13.
- **Analysis window:** 2018-11-11 onward. Before late 2018 the open, high and low repeat the close, so no real daily range exists.
- **Target:** 1 if the next session's close is higher than today's, else 0 (the single flat session counts as 0).
- **Features (known at the close of session t):** today's return, returns of the two previous sessions, daily range as % of close.
- **Split:** chronological 80/20. Train: 2018-11-14 to 2025-03-30 (1,460 sessions). Test: 2025-04-01 to 2026-11-06 (365 sessions).
- **Class balance (test):** 53.2% down, 46.8% up.
- **Environment:** Kaggle notebook; Python 3.13, pandas 2.3.3, numpy 2.1.3, matplotlib 3.10.0, scikit-learn 1.6.1.

## Data quality notes
- 27 sessions (about 1.5%) have an open or close outside the recorded high–low range; one (2019-01-09) has a high below its low. Rows kept to preserve the time sequence; daily range computed from the max and min
