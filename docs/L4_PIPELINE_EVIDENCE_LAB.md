# L4: Build Features and Demand Data

- **Duration:** 35 to 45 minutes
- **Cloud required:** No
- **Outcome:** A verified feature file and pipeline summary with an evidence-based finding.

## Expected Result

Before running the pipeline, record what you expect it to do:

```text
The pipeline will reuse the configured January 2024 source files, clean the
trips, create hourly demand, generate demand and fare features, split both
tables by time, and write a summary.
```

## Task 1: Verify Inputs and Configuration

1. Confirm the repository root and project interpreter.
2. Confirm both January 2024 files exist under `data/raw`.
3. Inspect the configured period, directories, cleaning bounds, split fractions,
   lags, and rolling windows in `config/config.yaml`.

If raw inputs are missing on a fresh checkout, acquire and build them once with
the following command (requires internet), then use Task 2 for the cached rerun:

```powershell
.\.venv\Scripts\python.exe -m scripts.run_pipeline --year 2024 --month 1
```

Result:

- Repository root: `/Users/rimjhimsingh/Documents/Certificate-Program-AIML/taxi-forecasting-student`.
- Project interpreter: the repo uses `.venv/bin/python` in this environment.
- January 2024 raw files present in `data/raw`:
  - `yellow_tripdata_2024-01.parquet`
  - `taxi_zone_lookup.csv`
- Configured period: `tlc_year = 2024`, `tlc_month = 1`.
- Data directories: `data/raw`, `data/processed`, and `data/features`.
- Cleaning bounds: fare 0 to 500, trip distance 0 to 100, passenger count 1 to 8, trip duration 1 to 240 minutes.
- Split fractions: train 0.8, validation 0.1, test 0.1.
- Demand feature lags: `[1, 2, 3, 24, 168]`.
- Demand feature rolling windows: `[3, 24, 168]`.

## Task 2: Run the Pipeline

From the repository root:

```powershell
.\.venv\Scripts\python.exe -m scripts.run_pipeline --skip-download
```

Record whether the command completed and identify the last successful stage.

Result:

- The pipeline command completed successfully with exit code 0.
- The final successful stage was writing the generated parquet files and the summary artifact: `data/processed/pipeline_summary.json`.
- The logged pipeline sequence completed in order:
  1. ingest/load raw files
  2. clean trips and write `trips_clean.parquet`
  3. aggregate hourly demand and write `demand_hourly.parquet`
  4. build demand and fare features and write the feature parquet files
  5. create train/validation/test splits for both demand and fare
  6. write the final summary JSON

## Task 3: Inspect Persisted Evidence

Inspect:

- `data/processed/pipeline_summary.json`
- `data/processed/trips_clean.parquet`
- `data/processed/demand_hourly.parquet`
- `data/features/demand_features.parquet`
- `data/features/fare_features.parquet`
- The six demand and fare train, validation, and test files

Record the cleaning counts, demand dimensions, feature shapes, and chronological
date ranges shown by the files.

Result:

- Pipeline summary values:
  - input_rows = 2,964,624
  - output_rows = 2,727,349
  - total_dropped = 237,275
  - largest drop reason = `passenger_count_out_of_range` with 167,879 rows
- Clean trip rows: 2,727,349
- Demand hourly table: 188,232 rows, 253 unique zones, 744 unique hours, 118,510 zero-count rows
- Demand table date range: 2024-01-01 00:00:00 to 2024-01-31 23:00:00
- Demand feature table: 145,728 rows after warm-up removal
- Fare feature table: 2,727,349 rows
- Chronological split ranges:
  - demand_train: 116,582 rows, 2024-01-08 00:00:00 to 2024-01-27 04:00:00
  - demand_val: 14,573 rows, 2024-01-27 04:00:00 to 2024-01-29 14:00:00
  - demand_test: 14,573 rows, 2024-01-29 14:00:00 to 2024-01-31 23:00:00
  - fare_train: 2,181,879 rows, 2024-01-01 00:00:00 to 2024-01-26 06:29:51
  - fare_val: 272,735 rows, 2024-01-26 06:29:55 to 2024-01-28 22:20:01
  - fare_test: 272,735 rows, 2024-01-28 22:20:03 to 2024-01-31 23:59:55

## Task 4: Run Consistency Checks

Confirm that:

- Clean rows match the pipeline summary.
- Fare-feature rows match clean-trip rows.
- Summed hourly pickup counts match clean-trip rows.
- Demand zone-hour keys are unique.
- Demand and fare feature rows match the pipeline summary.

Result:

- `clean_rows = 2727349` and pipeline summary output rows = 2727349: pass.
- `fare_feats_rows = 2727349` and clean rows = 2727349: pass.
- `sum_pickup_count = 2727349` and clean rows = 2727349: pass.
- `unique_zone_hour_keys = 188232` and `demand_hourly_rows = 188232`: pass.
- `demand_feats_rows = 145728` and summary `demand_feature_rows = 145728`: pass.
- `fare_feats_rows = 2727349` and summary `fare_feature_rows = 2727349`: pass.
- All consistency checks passed.

## Task 5: Explain the Result

### Evidence or output paths

```text
data/processed/pipeline_summary.json
data/processed/trips_clean.parquet
data/processed/demand_hourly.parquet
data/features/demand_features.parquet
data/features/fare_features.parquet
data/features/demand_train.parquet
data/features/demand_val.parquet
data/features/demand_test.parquet
data/features/fare_train.parquet
data/features/fare_val.parquet
data/features/fare_test.parquet
```

### Main finding

```text
The January 2024 pipeline cleaned the raw trips successfully and preserved the expected downstream feature tables. After cleaning, the dataset retained 2,727,349 valid trips, while the hourly demand table had 188,232 zone-hour rows and unique zone-hour keys. The demand and fare feature tables matched the summary values, and the train/validation/test splits covered chronological, non-overlapping time ranges.
```

### Decision or next action

```text
Proceed to model development with the generated demand and fare feature tables. Use the persisted train/validation/test splits as the official evidence base for any downstream modeling or evaluation work.
```

### Limitation

```text
This evidence only establishes the behavior of the local January 2024 snapshot, not the full year or future data. The cleaning rules are project-defined and the geography is broad zone-level aggregation rather than trip-level identity or demographic context.
```

## Completion Check

- The expected result was recorded before execution.
- The pipeline command and final result are visible.
- At least one generated feature file and the pipeline summary were inspected.
- The consistency checks passed or any failure was explained.
- The finding cites observed evidence rather than inference alone.
- The next action follows from the evidence.
- The limitation stays within what a local January 2024 run can establish.