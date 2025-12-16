Can We Predict NYC Subway Delays?
Modeling Additional Platform Time (APT) with MTA Open Data

Authors: Will Wu, John Yue
Date: Dec 16, 2025

This project predicts next month’s NYC subway delay level using Additional Platform Time (APT) at the subway-line × month level. We merge two MTA open datasets (monthly APT metrics + monthly delay incident counts), engineer simple time-series features (lags/rolling means/calendar), and compare several regression baselines using a chronological train/test split.

Problem statement

Can we predict next month’s APT for a given subway line using recent delay history and incident patterns?

APT is a useful target because it compresses “delay” into one interpretable number: extra minutes spent waiting on the platform.

Data sources

We use two public CSV exports (Socrata / NY State Open Data):

Customer journey metrics (Beginning 2023): monthly APT by subway line (target).

Subway delay incidents (Beginning 2020): monthly incident counts by line (signals).

Feature engineering

All time-series features are computed within each subway line and shifted to avoid leakage.

APT history

apt_lag1, apt_lag2

apt_roll3_mean (3-month rolling mean, shifted)

Incidents

Monthly incident totals aggregated to line × month

Lagged incident features (e.g., last month / two months ago)

Calendar + categorical

Month / quarter / season indicators

One-hot encoded line labels

Modeling approach

Train/test split: chronological — first 80% of months for training, last 20% held out (deployment-like evaluation).

Shared preprocessing (all models)

Numeric: median imputation

Categorical: most-frequent imputation + one-hot encoding (handle_unknown="ignore")

Models compared

Baseline: predict apt_lag1

Ridge Regression

Random Forest

Histogram-based Gradient Boosting (HistGB)

Results (held-out future months)
Model	MAE	RMSE	R²
Baseline (lag1)	0.201	0.285	0.313
Ridge	0.260	0.337	0.041
Random Forest	0.197	0.265	0.408
HistGB (best)	0.193	0.263	0.418

Takeaways

The baseline is already strong ⇒ APT is highly autocorrelated (last month is a good guess).

Tree-based models outperform the baseline; HistGB is best overall.

Ridge underperforms, suggesting relationships/interactions are not purely linear.

EDA highlights (from the report figures)

APT distribution: concentrated ~1.0–1.5 minutes with a right tail up to ~3.1 → extreme months inflate RMSE.

Seasonality: average APT dips Jan→May, peaks in July, then declines (month/season features help).

Line differences: large differences in typical APT by service → line ID matters.

Error behavior: HistGB tracks typical months well; high-APT spikes are hardest to predict.

Feature importance check (RF): recent APT history dominates; calendar and line indicators are secondary.

Limitations

Missing external drivers: weather, planned work, service advisories, special events.

Monthly granularity → fewer observations per line; sensitivity to time window.

Hard-to-predict tail events (extreme delay months).

Next steps

Add external signals (weather + service advisory feeds).

Predict at weekly or daily granularity for more training signal and faster responsiveness.

Calibrate uncertainty / quantify tail-risk performance (e.g., separate metrics for high-APT months).

Reproducibility

CSV downloads cached (no repeated API calls).

Single consistent preprocessing pipeline across models for fair comparison.
