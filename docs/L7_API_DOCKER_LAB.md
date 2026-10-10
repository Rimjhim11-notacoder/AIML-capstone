# L7: Call and Package the Prediction API

- **Duration:** 45 to 60 minutes
- **Cloud required:** No
- **Docker required:** Optional; a local Python fallback is provided
- **Outcome:** Valid and invalid API evidence plus a verified container or local runtime.

## Prerequisites

From the repository root, confirm that the project environment and serving
artifacts exist:

```powershell
.\.venv\Scripts\python.exe --version
Test-Path mlflow.db
Test-Path mlruns
Test-Path data\processed\demand_hourly.parquet
```

Path existence alone does not prove that models are registered. In the MLflow
UI, confirm both `taxi-demand-forecaster` and `taxi-fare-predictor` have registered
versions with accessible artifacts. L5's `--no-register` run does not satisfy
this requirement, and L6 may have registered only fare.

If inputs or registered models are missing, complete the pipeline and
model-training steps in [SETUP.md](SETUP.md). For a bounded teaching run that
registers both models, use:

```powershell
.\.venv\Scripts\python.exe -m scripts.train_models --n-trials 1
```

If you plan to use Route A, create and verify the portable registry required by
the Docker image:

```powershell
.\.venv\Scripts\python.exe scripts\make_mlflow_portable.py `
	--src mlflow.db `
	--dest mlflow.container.db `
	--new-base "file:///app/mlruns"
Test-Path mlflow.container.db   # should return True
```

Repeat this command after retraining so the container uses the latest registry.
Route B does not require `mlflow.container.db`.

### Prerequisite Check Results

Commands run from the repository root in the macOS shell:

```bash
source .venv/bin/activate && python --version && for path in mlflow.db mlruns data/processed/demand_hourly.parquet; do if test -e "$path"; then printf 'PASS %s\n' "$path"; else printf 'FAIL %s\n' "$path"; fi; done
```

Exact output:

```text
Python 3.11.7
PASS mlflow.db
PASS mlruns
PASS data/processed/demand_hourly.parquet
```

Command used to verify both registry artifacts through the API serving loader:

```bash
source .venv/bin/activate && python -c 'from src.serving.registry import load_model; print("demand:", type(load_model("demand")).__name__); print("fare:", type(load_model("fare")).__name__)'
```

Observed result:

```text
demand: LGBMRegressor
fare: LGBMRegressor
```

Command used to check the optional Docker route:

```bash
docker version
```

 result: `bash: docker: command not found` (exit code `127`). Docker Route A
is unavailable; use the local Python Route B. The required local prerequisites
and both registered model loads passed.

## Task 1: Predict the Runtime Result

Record the expected health status, one valid prediction result, and the status
code expected from an invalid request before starting the service.

Result (predicted before starting the API):
- Port check performed with `lsof -nP -iTCP:8000 -sTCP:LISTEN`; it returned no listener, so the API is not running yet.
- Task 1 is a prediction step, so no health or prediction HTTP request has been executed. The commands below are for verifying these predictions after Task 2 starts the API.

1. Health check

Command:

```bash
curl -i http://127.0.0.1:8000/health
```

Expected result: HTTP `200`; JSON should contain `"status":"ok"`, `"demand_model_loaded":true`, `"fare_model_loaded":true`, plus positive `history_hours` and `zones` values.

Actual Result: curl: (7) Failed to connect to 127.0.0.1 port 8000 after 0 ms: Couldn't connectto server

Explanation: Both registered model artifacts loaded during prerequisite checks, and the configured demand-history Parquet file exists. `/health` reports readiness only after the serving service and its history are loaded.

2. Valid fare prediction

Command:

```bash
curl -i -X POST http://127.0.0.1:8000/predict/fare \
	-H 'Content-Type: application/json' \
	-d '{"trip_distance":3.4,"pu_location_id":132,"do_location_id":230,"passenger_count":1,"pickup_datetime":"2024-01-15T18:30:00"}'
```

Expected result: HTTP `200`; JSON fields should include a numeric `predicted_fare`, `"duration_estimated":true`, and a generated `request_id`. The exact fare value depends on the registered model.

Explanation: The payload satisfies the fare schema. It omits optional `trip_duration_min`, so the API estimates duration from distance and marks it as estimated.

3. Invalid fare prediction

Command (same request with required `trip_distance` omitted):

```bash
curl -i -X POST http://127.0.0.1:8000/predict/fare \
	-H 'Content-Type: application/json' \
	-d '{"pu_location_id":132,"do_location_id":230,"passenger_count":1,"pickup_datetime":"2024-01-15T18:30:00"}'
```

Expected result: HTTP `422`; the validation response should identify `body.trip_distance` as missing (`Field required`).

Explanation: `trip_distance` is required by `FareRequest`; FastAPI rejects the request during schema validation before calling the model.


## Task 2: Start the API

Choose one route.

### Route A: Docker Compose

Confirm Docker is available, then build and start the API and MLflow UI:

```powershell
docker version
docker compose -f docker/docker-compose.yml up --build
```

If `docker version` reports that the command is unavailable, cannot connect to
the Docker daemon, or does not include Compose, skip the second command and use
Route B. Docker is optional for L7; record that you used the local fallback.

Keep this terminal open. The API is available at `http://localhost:8000` and
the MLflow UI at `http://localhost:5000`.

### Route B: Local Python

Use this route when Docker is unavailable:

```powershell
.\.venv\Scripts\python.exe -m uvicorn src.serving.api:app --host 127.0.0.1 --port 8000
```

Keep this terminal open. This route starts the API at `http://localhost:8000`;
it does not start the MLflow UI, which is not required to complete L7.

Result:

- Route used: B, local Python. Docker was unavailable (`docker: command not found`, exit code `127`).
- Command run: `source .venv/bin/activate && python -m uvicorn src.serving.api:app --host 127.0.0.1 --port 8000`.
- Observed startup output:

```text
INFO:     Started server process [61240]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
INFO:     Uvicorn running on http://127.0.0.1:8000 (Press CTRL+C to quit)
```

- Explanation: Uvicorn completed startup and is listening on the local API address. This confirms the process started; health and prediction behavior are checked in Tasks 3-4.

## Task 3: Verify Health and Documentation

In another PowerShell terminal:

```powershell
Invoke-RestMethod http://localhost:8000/health
Start-Process http://localhost:8000/docs
```

Record the health response and identify the fare and demand prediction
endpoints in Swagger UI.

Result:

- Health-check command run: `curl -sS -i http://127.0.0.1:8000/health`.
- Exact response body: `{"status":"ok","demand_model_loaded":true,"fare_model_loaded":true,"history_hours":744,"zones":253}`.
- HTTP status: `200 OK`; request ID: `cad94ce2-e3a8-4012-83a1-ead25e9c1da7`.
- Opened Swagger UI at `http://127.0.0.1:8000/docs`. Confirmed `POST /predict/fare` and `POST /predict/demand` are documented. The UI also lists `GET /health`, `POST /forecast/demand`, `POST /explain/fare`, and `POST /explain/demand`.
- Explanation: the health response confirms both registered models are loaded and demand history is available for 744 hours across 253 zones. Swagger exposes the fare and demand prediction operations required by the lab.

## Task 4: Exercise Valid and Invalid Requests

Use the request examples in `docs/API_EXAMPLES.md` to send:

1. One valid fare request.
2. One valid demand request.
3. One invalid request with a missing or out-of-range field.

Record the prediction fields from the valid responses. For the invalid request,
record the HTTP status and the validation message without changing server code.

## Task 5: Inspect the Packaging Boundary

Open `docker/Dockerfile` and identify:

- The builder and runtime stages.
- The copied source, model registry, model artifacts, and demand history.
- Port `8000`.
- The Uvicorn startup command.

Then open `docker/docker-compose.yml` and identify:

- The `api` and `mlflow-ui` services and their port mappings.
- The development `mlflow.db`, model-artifact, and data mounts that override
	the corresponding files baked into the image.
- The API `/health` container check.

If you used Route B, inspect these files as packaging evidence but state that
you did not execute the image build, Compose wiring, or container health check.

Explain why Docker packages the API and its dependencies but does not train the
models or prove that the service is deployed to AWS.

## Task 6: Stop the Runtime

For Docker Compose, press `Ctrl+C`, then run:

```powershell
docker compose -f docker/docker-compose.yml down
docker compose -f docker/docker-compose.yml ps   # should show no running services
```

For local Python, press `Ctrl+C` in the Uvicorn terminal.

## Result Record

### Runtime route and evidence

```text
Route B (local Python) was used because `docker version` returned `bash: docker: command not found` (exit code 127).

Task 2 command:
source .venv/bin/activate && python -m uvicorn src.serving.api:app --host 127.0.0.1 --port 8000

Observed startup:
INFO: Started server process [61240]
INFO: Waiting for application startup.
INFO: Application startup complete.
INFO: Uvicorn running on http://127.0.0.1:8000

Task 3 health command:
curl -sS -i http://127.0.0.1:8000/health

Observed result: HTTP 200 OK; response body:
{"status":"ok","demand_model_loaded":true,"fare_model_loaded":true,"history_hours":744,"zones":253}
Request ID: cad94ce2-e3a8-4012-83a1-ead25e9c1da7.

Swagger was opened at http://127.0.0.1:8000/docs. It documented POST /predict/fare and POST /predict/demand, along with GET /health, POST /forecast/demand, POST /explain/fare, and POST /explain/demand.

Task 6: Ctrl+C was sent to the Uvicorn terminal. It logged shutdown, application shutdown complete, and finished server process [61240]. The terminal reported exit code 1 because the process was interrupted. Follow-up check `if lsof -nP -iTCP:8000 -sTCP:LISTEN; then echo 'API still listening'; else echo 'No API listener on port 8000'; fi` returned `No API listener on port 8000`.
```

### Valid response finding

```text
Fare command:
curl -sS -i -X POST http://127.0.0.1:8000/predict/fare -H 'Content-Type: application/json' -d '{"trip_distance":3.4,"pu_location_id":132,"do_location_id":230,"passenger_count":1,"pickup_datetime":"2024-01-15T18:30:00"}'

Exact result: HTTP 200 OK; {"predicted_fare":20.16,"duration_estimated":true,"request_id":"fe7bffe5-f234-437c-b134-a3ea871a593a"}.
Explanation: The request was valid. Since trip_duration_min was omitted, the API estimated duration and marked duration_estimated true. Fare is the registered model's prediction for this input, not a fixed expected value.

Demand command:
curl -sS -i -X POST http://127.0.0.1:8000/predict/demand -H 'Content-Type: application/json' -d '{"zone":132,"target_hour":"2024-02-01T00:00:00"}'

Exact result: HTTP 200 OK; {"zone":132,"target_hour":"2024-02-01T00:00:00","predicted_pickups":142.76,"request_id":"46c8548f-aad1-4c3b-9b14-a7f4888792de"}.
Explanation: The request used known zone 132 and a parseable target hour; the API returned a numeric pickup forecast.
```

### Invalid response finding

```text
Command (trip_distance omitted):
curl -sS -i -X POST http://127.0.0.1:8000/predict/fare -H 'Content-Type: application/json' -d '{"pu_location_id":132,"do_location_id":230,"passenger_count":1,"pickup_datetime":"2024-01-15T18:30:00"}'

Exact result: HTTP 422 Unprocessable Entity; {"detail":[{"type":"missing","loc":["body","trip_distance"],"msg":"Field required","input":{"pu_location_id":132,"do_location_id":230,"passenger_count":1,"pickup_datetime":"2024-01-15T18:30:00"}}]}.
Explanation: FastAPI rejected the payload during request-schema validation because FareRequest requires trip_distance; the model was not called.
```

### Packaging explanation and limitation

```text
The inspected Dockerfile uses a builder stage (Python 3.11 slim, build tools, libgomp, and requirements installed into a virtual environment) and a runtime stage (Python 3.11 slim with libgomp and the copied environment). The runtime copies src, config, scripts, mlruns, mlflow.container.db as mlflow.db, and data/processed/demand_hourly.parquet; it exposes port 8000 and starts Uvicorn with `uvicorn src.serving.api:app --host 0.0.0.0 --port 8000`.

docker-compose.yml defines `api` on host/container port 8000 and `mlflow-ui` on 5000. Development mounts provide mlflow.container.db at /app/mlflow.db and mlruns at /app/mlruns; the API also mounts ../data at /app/data. The API healthcheck requests http://localhost:8000/health.

Route B was used, so the image build, Compose wiring, and container healthcheck were not executed. Docker packages the already-trained API runtime and artifacts; it does not train models. Building or running a local image does not prove deployment to AWS, which requires separate infrastructure and deployment steps.
```

## Completion Check

- The expected result was recorded before starting the service.
- Health and Swagger documentation were inspected.
- Valid fare and demand requests returned interpretable responses.
- An invalid request returned an explained validation error.
- Docker was exercised, or its absence and the local fallback were recorded.
- The result distinguishes local packaging from AWS deployment.