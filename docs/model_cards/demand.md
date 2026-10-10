# Model Card — taxi-demand-forecaster

_Generated: 2026-10-04 13:22 UTC_

## Model details

- **Problem:** demand
- **Algorithm:** LGBMRegressor
- **Target:** `pickup_count`
- **Registry version:** n/a

## Intended use

Operational forecasting of pickup_count to support fleet positioning and rider fare estimates. Not intended for individual pricing decisions or any use with legal/financial consequences for individuals.

## Evaluation — overall (test split)

| Metric | Value |
|--------|-------|
| rmse | 8.6187 |
| mae | 2.5241 |
| mape | 40.2065 |
| r2 | 0.9713 |

## Evaluation — subgroup fairness

### By borough

Disparity ratio (worst/best RMSE): **211.13** (threshold 1.5) — ⚠️ **flagged**. Worst: `Manhattan`, best: `Staten Island`.

| Group | n | RMSE | MAE | MAPE |
|-------|---|------|-----|------|
| Manhattan | 3862 | 15.347 | 7.677 | 29.88 |
| Queens | 3971 | 6.528 | 1.184 | 52.60 |
| Unknown | 116 | 3.334 | 2.067 | 48.05 |
| Brooklyn | 3457 | 0.738 | 0.439 | 53.70 |
| Bronx | 2476 | 0.441 | 0.260 | 63.57 |
| EWR | 58 | 0.269 | 0.098 | 97.18 |
| Staten Island | 633 | 0.073 | 0.044 | 92.42 |

### By time_of_day

Disparity ratio (worst/best RMSE): **2.70** (threshold 1.5) — ⚠️ **flagged**. Worst: `evening`, best: `overnight`.

| Group | n | RMSE | MAE | MAPE |
|-------|---|------|-----|------|
| evening | 4554 | 11.661 | 3.327 | 37.67 |
| afternoon | 3947 | 8.581 | 2.966 | 35.87 |
| morning | 3036 | 6.184 | 2.363 | 38.98 |
| overnight | 3036 | 4.316 | 0.907 | 58.03 |

### By day_type

Disparity ratio (worst/best RMSE): **1.00** (threshold 1.5) — ok. Worst: `weekday`, best: `weekday`.

| Group | n | RMSE | MAE | MAPE |
|-------|---|------|-----|------|
| weekday | 14573 | 8.619 | 2.524 | 40.21 |

### By holiday

Disparity ratio (worst/best RMSE): **1.00** (threshold 1.5) — ok. Worst: `non-holiday`, best: `non-holiday`.

| Group | n | RMSE | MAE | MAPE |
|-------|---|------|-----|------|
| non-holiday | 14573 | 8.619 | 2.524 | 40.21 |

### By airport

Disparity ratio (worst/best RMSE): **4.74** (threshold 1.5) — ⚠️ **flagged**. Worst: `airport`, best: `non-airport`.

| Group | n | RMSE | MAE | MAPE |
|-------|---|------|-----|------|
| airport | 116 | 37.732 | 26.193 | 38.94 |
| non-airport | 14457 | 7.966 | 2.334 | 40.23 |

## Top features (SHAP)

| Feature | Mean |SHAP| |
|---------|-----------|
| lag_1 | 12.4169 |
| lag_168 | 7.6835 |
| lag_24 | 4.2317 |
| rolling_mean_3 | 0.9685 |
| hour_cos | 0.6063 |
| lag_3 | 0.4060 |
| hour | 0.3874 |
| rolling_std_24 | 0.3714 |
| rolling_std_3 | 0.3687 |
| rolling_mean_168 | 0.3428 |

## Limitations

- Trained on a single month of NYC Yellow Taxi data; may not generalise to other periods, boroughs served sparsely, or green/for-hire vehicles.
- Predictions degrade under distribution shift (see drift monitoring).

## Ethical considerations

- No demographic or protected attributes are used or available; fairness is assessed as geographic/temporal service equity, not demographic parity.
- Large borough-level disparities could translate into unequal service quality across neighbourhoods and should be monitored.
