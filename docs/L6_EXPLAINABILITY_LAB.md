# L6: Explain Model Behavior

- **Duration:** 35 to 45 minutes
- **Cloud required:** No
- **Outcome:** A plain-language explanation grounded in SHAP and model-card evidence.

## Prerequisites

This preparation supports the recorded four-task walkthrough; it is not a new
training exercise inside L6.

Complete the feature pipeline and register a fare model. L5's `--no-register`
comparison creates tracked runs, but not the registered model that this analysis
loads. If no fare model is registered, run this bounded teaching configuration:

```powershell
.\.venv\Scripts\python.exe -m scripts.train_models --problem fare --n-trials 1
```

In the MLflow UI, confirm `taxi-fare-predictor` has a registered version and
accessible model artifacts. A one-trial model is sufficient for this exercise,
not production approval. If a suitable version already exists, reuse it and
record its identity instead of retraining. As in the finalized recording, record
the configured reference `models:/taxi-fare-predictor/latest` unless configuration
differs. The generated card currently shows registry version `n/a`; do not infer
a numeric version from `latest`. Add a resolved version only when separately
verified, and otherwise retain the identity gap.

Result:

- The `taxi-fare-predictor` model was not registered initially, so the bounded prerequisite command was run: `python -m scripts.train_models --problem fare --n-trials 1`.
- LightGBM was selected and registered as `taxi-fare-predictor` version `1` from run `e0f4777d18364bf4aa01f36f51a48eeb`.
- The generated card shows registry version `n/a`; the registry separately confirms version `1`. The analysis used the configured reference `models:/taxi-fare-predictor/latest`.

## Task 1: Generate Fare Analysis

From the repository root:

```powershell
.\.venv\Scripts\python.exe -m scripts.responsible.run_analysis --problem fare
```

Record the command result and the artifact paths it creates or refreshes.

Result:

- Command run successfully: `python -m scripts.responsible.run_analysis --problem fare`.
- The analysis loaded `models:/taxi-fare-predictor/latest`, evaluated the fare test split, and reported three flagged subgroup dimensions.
- Generated or refreshed artifacts:
	- `artifacts/responsible/fare_shap_summary.png`
	- `artifacts/responsible/fare_fairness.json`
	- `docs/model_cards/fare.md`
- To verify both registered problem models, also ran `python -m scripts.responsible.run_analysis --problem demand`. It loaded `models:/taxi-demand-forecaster/latest`, completed successfully, and reported three flagged subgroup dimensions.
- Demand outputs refreshed: `artifacts/responsible/demand_shap_summary.png`, `artifacts/responsible/demand_fairness.json`, and `docs/model_cards/demand.md`.

## Task 2: Inspect Explainability Evidence

Open the fare outputs under `artifacts/responsible/` and the fare model card
under `docs/model_cards/`. Identify the most influential features, the direction
or magnitude the evidence supports, and the model/version represented.

Follow the recorded global beeswarm reading order:

1. Vertical feature order summarizes mean absolute SHAP influence over the sample.
2. Horizontal position is signed contribution relative to the model baseline.
3. Color represents the feature's low-to-high input value, not good/bad outcomes.
4. Each dot is one sampled row for one feature; density and overlap show spread.
5. Read numeric location IDs as labels, not geographic distance or ordered value.

Result:

- The analyzed model is `LGBMRegressor`, target `fare_amount`, using the registered fare model reference `models:/taxi-fare-predictor/latest` (verified registry version `1`).
- Evidence is in `artifacts/responsible/fare_shap_summary.png`, `artifacts/responsible/fare_fairness.json`, and `docs/model_cards/fare.md`.
- The SHAP ranking is led by `trip_distance` (mean absolute SHAP `6.8493`) and `trip_duration_min` (`3.7284`); next are `PULocationID` (`0.6443`) and `DOLocationID` (`0.5209`). Higher distance and duration values generally appear with positive SHAP contributions in the beeswarm, while lower values tend toward negative contributions. These are shifts relative to the model baseline, not causal effects.
- Numeric location IDs are labels, not an ordered geographic scale. The card reports flagged RMSE disparity for borough (ratio `8.12`, worst `Unknown`), time of day (`1.84`, worst `overnight`), and airport status (`2.54`, worst `airport`). Day-type disparity (`1.32`) and the available holiday comparison (`1.00`) were not flagged at the `1.5` threshold.
- The plot summarizes a sample of up to 2,000 test rows; it does not explain a particular trip.

The global plot does not explain one named trip. Use a single-row explanation
for that question. Correlated inputs can share attribution, and neither the plot
nor its ranking establishes causality, fairness, or production readiness.

## Task 3: Write a Bounded Explanation

Use this structure:

```text
For this analyzed model and data sample, [feature] has [observed influence].
This suggests [plain-language interpretation]. The evidence does not prove
[causal or individual-level claim].
```

SHAP describes how model inputs contribute to predictions. It does not prove
that changing a feature causes the real-world outcome to change.

Result:

For the analyzed LightGBM fare model and its January 2024 test-data sample, `trip_distance` has the greatest global influence (mean absolute SHAP `6.8493`), and higher distance values generally have positive contributions to predicted fare. This suggests the model uses trip distance as a major input when estimating fares. The evidence does not prove that distance causes a real-world fare change or explain any individual trip.

## Task 4: Connect the Model Card

Record one intended use, one limitation, and one monitoring or review action
from the model card. Explain how each item changes the way the SHAP result
should be communicated.

Result:

- Intended use: operational forecasting to support fleet positioning and rider fare estimates. Do not present this SHAP result as a basis for individual pricing decisions or decisions with legal or financial consequences for individuals.
- Limitation: training data covers one month of NYC Yellow Taxi trips, so results may not generalize to other periods, sparsely represented boroughs, or other vehicle types. Describe the SHAP findings as specific to this model and sample.
- Monitoring/review action: monitor drift and subgroup performance, including the borough, overnight, and airport groups flagged in the fairness report. The SHAP ranking alone does not establish fairness, causality, or production readiness.
- Model-card test metrics: RMSE `3.4717`, MAE `0.8151`, MAPE `8.6912`, and R-squared `0.9550`.

## Completion Check

- The responsible-analysis command completed or its failure was explained.
- The explanation cites a generated artifact and the represented model.
- Feature influence is described without claiming causation.
- Intended use and limitations from the model card are included.
- The conclusion stays within the analyzed sample and model version.