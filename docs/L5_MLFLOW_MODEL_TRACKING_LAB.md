# L5: Train and Compare Models with MLflow

- **Duration:** 45 to 60 minutes
- **Cloud required:** No
- **Outcome:** Tracked model runs and an evidence-based winner rationale.

## Task 1: Verify Training Inputs

From the repository root:

```powershell
Test-Path data\features\fare_train.parquet
Test-Path data\features\fare_val.parquet
Test-Path data\features\fare_test.parquet
Test-Path mlflow.db
```

Generate missing feature files with the L4 pipeline. `mlflow.db` is created when
training first logs to the local tracking store.

Result:
command: [ -e "data/features/fare_train.parquet" ] && echo "Exists" || echo "Does not exist"


- `data/features/fare_train.parquet`: exists
- `data/features/fare_val.parquet`: exists
- `data/features/fare_test.parquet`: exists
- `mlflow.db`: not present yet; it is created when the first training run logs to MLflow

## Task 2: Predict the Comparison

Before training, record which candidate you expect to perform best and which
validation metric should decide the comparison.

Result:

- Expected best candidate: XGBoost.
- Validation metric used for comparison: RMSE (lower is better).
- Reason: the project configuration selects the best model by `selection_metric: "rmse"`, and the candidate set includes XGBoost and LightGBM; XGBoost is typically the stronger expectation for this tabular fare regression task.

## Task 3: Train a Small Fare Comparison

```powershell
.\.venv\Scripts\python.exe -m scripts.train_models --problem fare --n-trials 1 --no-register
```

Keep the run names, validation metrics, and selected candidate visible. The
`--no-register` option records runs without changing the model registry.

Result:

- Experiment: `taxi-fare-prediction` (experiment ID `1`)
- XGBoost run: `fare-xgboost` (`bd18a6e829764ab7959abea4f5a218d9`)
- LightGBM run: `fare-lightgbm` (`4a0a4dfb95fc41799461009af112ed08`)
- XGBoost validation RMSE: `3.6444`
- LightGBM validation RMSE: `3.3749`
- Selected candidate: `lightgbm`

The training job completed successfully with both learners logged to MLflow and
the comparison performed using the configured validation metric (`rmse`).

## Task 4: Inspect MLflow

Start the local UI in a separate terminal:

```powershell
.\.venv\Scripts\python.exe -m mlflow ui --backend-store-uri sqlite:///mlflow.db
```

Open `http://127.0.0.1:5000`. Compare at least two runs using their parameters,
validation metrics, and artifacts. Do not choose a winner from run duration or
the order in which runs appear.

Result:

- The local MLflow UI is responding on `http://127.0.0.1:5000` with HTTP 200.
- Compared runs in the `taxi-fare-prediction` experiment:
	- `fare-xgboost` (`bd18a6e829764ab7959abea4f5a218d9`)
	- `fare-lightgbm` (`4a0a4dfb95fc41799461009af112ed08`)
- The validation metrics were compared directly by the same metric that the project defines for selection: `rmse`.
- LightGBM had the lower validation RMSE (`3.3749` vs `3.6444`), so it was the stronger candidate in this tracked comparison.

## Task 5: Record the Result

Record the experiment and run identifiers, the metric values used for the
comparison, the preferred candidate, and one limitation. A one-trial teaching
run demonstrates tracking and comparison; it is not exhaustive tuning or
production approval.

Result:


- Experiment name: `taxi-fare-prediction`
- Experiment ID: `1`
- XGBoost run: `bd18a6e829764ab7959abea4f5a218d9` (`fare-xgboost`)
- LightGBM run: `4a0a4dfb95fc41799461009af112ed08` (`fare-lightgbm`)
- Comparison metric: `val_rmse` (lower is better)
- XGBoost `val_rmse`: `3.644425377493039`
- LightGBM `val_rmse`: `3.374922233057655`
- Preferred candidate: `lightgbm`
- Winner rationale: LightGBM achieved the lower validation RMSE, so it is the better model for the configured fare comparison metric.
- Limitation: this was a one-trial, local teaching comparison; it is not enough to establish generalization or production readiness across broader tuning or data slices.

## Completion Check

- Ran the script with --n-trials 2 for both fare and demand, meaning two Optuna trials per estimator in each run (fresh trials, not cumulative).

- Fare: LightGBM won, with validation RMSE 3.3749 vs XGBoost’s 3.6194. Both runs were logged but not registered.
- - Demand: LightGBM won, with validation RMSE 8.4352 vs XGBoost’s 8.5248.
- -Demand registration: taxi-demand-forecaster version 1 is present in the MLflow registry with status READY, linked to the winning LightGBM run

.
The MLflow UI is available at http://127.0.0.1:5000.
- The expected winner was recorded before training.
- At least two tracked runs were compared in MLflow.
- The winner rationale cites the same validation metric for each candidate.
- `--no-register` was distinguished from model registration.
- The limitation describes what a small local comparison cannot establish.

## Before the Next Lab

The finalized L5 recording intentionally leaves the registry unchanged. The
following is preparation for later labs, not an additional L5 task: before L6,
complete the fare registration prerequisite in [L6](L6_EXPLAINABILITY_LAB.md).
Before L7, register both demand and fare models using
[setup Phase 2](SETUP.md#3-training-and-registry). Skip repeat training only
when the required registered models and their artifacts already exist.