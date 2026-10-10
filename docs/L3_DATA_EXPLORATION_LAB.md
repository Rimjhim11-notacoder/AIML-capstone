# L3: Explore the Data and Trace Its Lineage

- **Cloud required:** No; initial source download requires internet.
- **Outcome:** Three findings about cleaning, demand, and feature/split behavior,
  each grounded in the executed notebook or persisted pipeline evidence.
- **Recording companion:** Follow the finalized L3 route below. L4 revisits the
  persistence and consistency checks; it does not replace this exploratory walkthrough.

## 1. Read the Complete Route

Open the [lineage diagram](data-lineage-route.png). Trace source, import,
cleaning, the demand/fare branch, chronological splits, and downstream consumers.
Distinguish transformation arrows from reference/evidence arrows.

Keep the three row meanings explicit:

- Raw trips: one published completed trip.
- Demand: one observed pickup zone during one hour, including inserted zero counts.
- Fare: one accepted completed trip after cleaning, targeting `fare_amount`.

Result:

- Raw trips are the source-level Yellow Taxi records before filtering.
- Demand rows are summarized by `PULocationID` and `pickup_hour`, with zero rows inserted for missing zone-hour combinations.
- Fare rows are the cleaned trip records used for `fare_amount` prediction features.

## 2. Verify the Official Source

Open the [NYC TLC Trip Record Data page](https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page).
Identify Yellow Taxi trip Parquet and the taxi-zone lookup CSV. The recording
uses January 2024; this is the configured teaching snapshot, not a claim that it
is the latest publication. Zones are broad labels, not identities or GPS traces.

Result:

- Official source: NYC TLC Yellow Taxi trip Parquet for January 2024.
- Lookup file: `taxi_zone_lookup.csv` from the TLC public dataset.
- Configured snapshot in this project: `tlc_year = 2024`, `tlc_month = 1`.
- Zone IDs are location labels used for aggregation, not identity or GPS trace data.

## 3. Connect Source to Code

Inspect [configuration](../config/config.yaml),
[ingestion](../src/data/ingestion.py), and
[the pipeline entry point](../scripts/run_pipeline.py), in that order. Record
the configured period, source URLs, filenames, and local destinations. Separate
download/load responsibility from validation and feature engineering.

Result:

- Configured period: year `2024`, month `1`.
- Base URL: `https://d37ci6vzurychx.cloudfront.net/trip-data`.
- Zone lookup URL: `https://d37ci6vzurychx.cloudfront.net/misc/taxi_zone_lookup.csv`.
- Trip filename: `yellow_tripdata_2024-01.parquet`.
- Zone filename: `taxi_zone_lookup.csv`.
- Local raw destinations: `data/raw/`.
- Processed artifacts: `data/processed/`.
- Feature outputs: `data/features/`.
- Download/load responsibility is handled in `src/data/ingestion.py`.
- Validation and feature engineering are handled separately in `src/data/validation.py`, `src/features/demand_features.py`, and `src/features/fare_features.py`.

## 4. Run and Open the Notebook

Complete [environment setup](SETUP.md) first. With raw inputs already present,
the recorded persistence command is:

```powershell
.\.venv\Scripts\python.exe -m scripts.run_pipeline --skip-download
```

If a raw file is missing, acquire it using the recorded fallback:

```powershell
.\.venv\Scripts\python.exe -m scripts.run_pipeline
```

Open [the exploration notebook](../notebooks/01_explore_dataset.ipynb) and select
the project `.venv` kernel. The recording preparation also rebuilds this notebook
with `python -m scripts.build_explore_notebook`. That command overwrites the
generated notebook, so use it only when intentionally regenerating it, not after
adding work you need to retain. No rebuild is required merely to open the notebook.

Result:

- The raw January 2024 files were already present in `data/raw/`, so the recorded `--skip-download` route was used successfully.
- Command run successfully: `./.venv/bin/python -m scripts.run_pipeline --skip-download`.
- The notebook was verified against the persisted pipeline output rather than assumed from memory.
- Rebuilds were not needed for this run because the generated notebook already existed and the data pipeline had already produced the artefacts.

## 5. Execute the Notebook in Order

Run one code cell at a time and interpret its output before continuing. Follow
the recorded sequence, using section names if cell positions have changed:

| Notebook segment | Evidence to inspect |
| --- | --- |
| Setup and source inventory | Project root, interpreter, provider, files, and source period |
| Raw inputs and schema | Trip preview, required fields, types, and zone lookup |
| Cleaning | Unsuitable values, ordered rules, retained/removed counts and reasons |
| Demand aggregation | Unique zone-hour keys, dense dimensions, zero rows, hourly profile, busiest zones |
| Demand features | Calendar inputs, past-only lags/rolling statistics, and history warm-up |
| Fare features | One clean trip per row, distance/duration/passengers/zones/calendar, and fare-distance plot |
| Chronological splits | All six train/validation/test outputs and their time boundaries |
| Artifacts and lineage checks | Saved paths, row-count relationships, unique keys, and assertions |

The notebook calculates stages in memory to explain them. The pipeline writes
the official persisted outputs. Do not claim a saved file was refreshed merely
because its in-memory equivalent was recalculated.

Result:

- Cleaning evidence: 2,964,624 raw rows became 2,727,349 clean rows; 237,275 rows were dropped.
- Demand evidence: 188,232 zone-hour rows, 253 zones, 744 hours, and 118,510 zero rows.
- Demand feature evidence: 145,728 rows after warm-up removal, with past-only lag and rolling features.
- Fare feature evidence: 2,727,349 rows preserved from the cleaned trips table.
- Split evidence: six outputs were created for demand and fare, with forward-only chronological boundaries.
- The notebook output and pipeline output were reconciled; the persisted files remain the source of truth.

## 6. Reconcile Persisted Evidence

Check the two raw inputs, clean trips, hourly demand, pipeline summary, both
complete feature tables, and all six split files: thirteen expected artifacts
in the recorded route. File existence proves persistence, not correctness.

Run the notebook's final assertions. Confirm cleaning did not add trips, fare
rows match clean rows, demand zone-hour keys are unique, and split times move
forward. Inspect the lag/rolling code separately to establish past-only inputs.

Result:

- The thirteen expected artifacts are present in the project data folders.
- `data/processed/trips_clean.parquet` contains 2,727,349 rows, matching the cleaned output reported by the pipeline summary.
- `data/processed/demand_hourly.parquet` contains 188,232 rows, with unique `PULocationID` + `pickup_hour` zone-hour keys.
- `data/features/fare_features.parquet` has the same row count as the clean trips table, confirming fare rows are preserved.
- Split files are ordered in time: demand train -> validation -> test, and fare train -> validation -> test.
- Past-only lags and rolling windows are implemented in the demand feature code; warm-up rows are dropped before model training.

## 7. Present Three Findings

Use current outputs, not memorized recording values:

1. **Cleaning:** raw, retained, and removed row counts; largest removal reason.
2. **Demand:** zones, hours, zone-hour rows, zero-count rows, and busiest zone.
3. **Features and splits:** fare row preservation, demand history warm-up, and
   chronological split boundaries.

For each finding record the notebook output or artifact, its configured period,
what it supports, and one limit. Counts can differ when input data or configuration
changes; preserve and investigate the difference instead of forcing a match.

Result:

1. Cleaning finding: January 2024 Yellow Taxi data had 2,964,624 rows, and the cleaned output retained 2,727,349 rows. The largest drop reason was `passenger_count_out_of_range` (167,879 rows). This supports that the cleaning rules were applied as expected, but the result is limited to the January 2024 snapshot and project assumptions.
2. Demand finding: the hourly demand table covered 253 pickup zones across 744 hours, with 188,232 zone-hour rows and 118,510 zero rows. The busiest demand zone was `PULocationID` 132. This supports the spatial and temporal density pattern, but it is limited to one month and broad zone geography rather than individual trip-level behavior.
3. Features and splits finding: the fare feature table preserved 2,727,349 rows, while demand features had 145,728 rows after warm-up removal. The chronological split boundaries were valid and time-ordered. This supports the past-only feature design and proper train/validation/test sequencing, but it is limited by one-month scope and no direct demographic or currentness guarantees.

## Completion Check

- Source and configuration are identified before interpreting results.
- All notebook sections were executed or a specific failed/skipped step is recorded.
- The three row meanings and the two feature branches remain distinct.
- Pipeline outputs and notebook calculations were reconciled.
- Three findings cite observed evidence.
- Limitations cover one-month Yellow Taxi scope, broad geography, no direct
  demographic attributes, project cleaning assumptions, and no currentness guarantee.