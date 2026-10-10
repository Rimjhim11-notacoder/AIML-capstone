# L2: Guess the Fare and Feel the Metrics

- **Duration:** 45 to 60 minutes
- **Cloud required:** No
- **Outcome:** A plain-language comparison of fare errors using MAE and RMSE.

## Learning Objectives

1. Distinguish an actual fare from a predicted fare.
2. Calculate signed, absolute, and squared errors.
3. Explain why positive and negative errors can cancel.
4. Interpret MAE and RMSE in dollars.
5. Show why RMSE reacts more strongly to one large miss.

## Part A: Commit Fare Guesses

Complete `Your guess` and `Reason` before viewing Part B.

| Trip | Description | Your guess | Reason |
| --- | --- | ---: | --- |
| A | 1.2 miles, weekday morning, Midtown | $12 | Morning Midtown short trip, usually a modest fare with a small base charge. |
| B | 2.5 miles, weekday afternoon, Manhattan | $15 | Midday Manhattan trip of a few miles, likely a typical city fare. |
| C | 4 miles, evening, Manhattan to Brooklyn | $22 | Longer evening cross-borough trip, so fare should be higher than a simple Manhattan ride. |
| D | 8 miles, evening airport route | $40 | Airport route is longer and usually more expensive, especially at night. |
| E | 5.5 miles, late evening, Queens | $23 | Late evening and a longer distance would push the fare above a short trip but not as high as an airport ride. |

## Part B: Calculate the Shared Errors

Use the convention:

```text
Error = prediction - actual
```

| Trip | Actual fare | Shared prediction | Error | Absolute error | Squared error |
| --- | ---: | ---: | ---: | ---: | ---: |
| A | $10 | $12 | +2 | 2 | 4 |
| B | $15 | $13 | -2 | 2 | 4 |
| C | $20 | $22 | +2 | 2 | 4 |
| D | $30 | $40 | +10 | 10 | 100 |
| E | $25 | $23 | -2 | 2 | 4 |

Complete these calculations:

```text
Sum of signed errors = 10
Mean signed error = 2.00

MAE = sum of absolute errors / 5 = (2 + 2 + 2 + 10 + 2) / 5 = 18 / 5 = 3.60

Mean squared error = sum of squared errors / 5 = (4 + 4 + 4 + 100 + 4) / 5 = 116 / 5 = 23.20
RMSE = square root of mean squared error = sqrt(23.20) ≈ 4.82
```

## Part C: Interpret the Baseline

1. The mean signed error can hide information because positive and negative errors cancel out, so a model can look unbiased even when some predictions are far off.
2. MAE means that the average absolute prediction error across these five trips is $3.60.
3. RMSE is larger than MAE because it squares each error before averaging, which gives extra weight to large mistakes.
4. Trip D contributes most to RMSE because its error is $10, and its squared error is 100, which is much larger than the other trips.
5. We should check separate operational groups such as airport trips, time of day, and borough or route type before drawing a broad conclusion.

### Verify After Calculating

The recording verifies the hand calculation with the project's metric helpers.
Run this from the repository root after completing the baseline arithmetic:

```powershell
@'
import numpy as np
from src.models.common import mae, rmse

actual = np.array([10, 15, 20, 30, 25], dtype=float)
predicted = np.array([12, 13, 22, 40, 23], dtype=float)
print("MAE:", mae(actual, predicted))
print("RMSE:", rmse(actual, predicted))
'@ | .\.venv\Scripts\python.exe -
```

Compare full precision before rounding to cents. Inspect
[the metric helpers](../src/models/common.py) and
[their tests](../tests/unit/test_models.py). Reversing subtraction changes the
signed error, but not MAE or RMSE after absolute values or squaring.

## Part D: Extreme-Miss Comparison

Change only trip D's prediction from `$40` to `$60`.

| Metric | Baseline | Extreme case | Increase |
| --- | ---: | ---: | ---: |
| MAE | 3.60 | 7.60 | 4.00 |
| RMSE | 4.82 | 13.53 | 8.71 |

Record the changed trip-D values:

```text
Signed error = 30
Absolute error = 30
Squared error = 900
```

After calculating the extreme case, rerun the verification block with only the
fourth prediction changed from `40` to `60`, as in the recording. The actuals
and the other four predictions must stay unchanged.

## Exit Ticket

Complete each statement:

1. Mean signed error can mislead because positive and negative errors cancel each other out.
2. MAE means the average absolute difference between predicted and actual fares in dollars.
3. RMSE is larger in the baseline because it gives greater weight to larger errors by squaring them.
4. MAE and RMSE should be reported with dollar units and enough context, including sample size and relevant subgroup breakdowns.

## Completion Check

- All five personal fare guesses are recorded before the shared values appear.
- The shared error table contains signed, absolute, and squared errors.
- MAE and RMSE calculations show the working and retain dollar units.
- Repository code confirms the hand-calculated values.
- The extreme-miss comparison changes only trip D.
- The interpretations state sample size, context, and relevant subgroup checks.