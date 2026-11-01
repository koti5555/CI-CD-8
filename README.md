# MLOps Practical 8 — Publish Student Result Prediction to GHCR

This repository implements Practical 8 using the Student Result Prediction Flask application from Practical 7.

## Aim

Validate the Student Result Prediction model and API, publish its Docker image to GitHub Container Registry (GHCR), pull the published image, and test the running service.

## Repository files

- `.github/workflows/ghcr-publish.yml` — test, build, publish, pull, and verify pipeline.
- `app.py` — Flask `/health` and `/predict` endpoints.
- `train_model.py` — creates the sample dataset and trained model artifact.
- `quality_gate.py` — requires model accuracy of at least 0.85.
- `test_ml_pipeline.py` — seven ML pipeline tests.
- `test_app.py` — four Flask API tests.
- `Dockerfile`, `.dockerignore`, and `requirements.txt` — container configuration.

## GHCR image names

- Version: `ghcr.io/koti5555/student-result-prediction:v1`
- Latest: `ghcr.io/koti5555/student-result-prediction:latest`
- Immutable commit tag: `ghcr.io/koti5555/student-result-prediction:<commit-sha>`

## Automated workflow

On every push to `main`, or when manually started from Actions, GitHub-hosted Ubuntu:

1. Installs Python dependencies.
2. Trains the model and checks the 0.85 quality gate.
3. Runs the seven ML tests and four API tests.
4. Logs in to GHCR using the automatically provided `GITHUB_TOKEN` (no personal access token is stored).
5. Builds and publishes `v1`, `latest`, and the commit-SHA image tags.
6. Pulls the exact commit-tagged image from GHCR.
7. Starts a container from that pulled image.
8. Checks `GET /health`, verifies PASS and FAIL predictions, prints logs, and removes the container.

The workflow grants only `contents: read` and `packages: write` permissions. The package is initially private by default; adjust package visibility in GitHub Packages settings only if public access is intended.

## API validation data

PASS request:

```json
{"attendance":90,"internal_marks":85,"assignment_marks":88,"previous_score":80}
```

Expected: `PASS`.

FAIL request:

```json
{"attendance":55,"internal_marks":30,"assignment_marks":40,"previous_score":35}
```

Expected: `FAIL`.

Health endpoint: `GET /health`.

## Run without Docker Desktop on your computer

No local Docker installation or personal access token is needed for this automated workflow. GitHub-hosted runners perform the Docker build, registry login, push, pull, and container/API validation.

## Continuity note

The source application, Dockerfile, model features, prediction request format, and API behavior are carried forward from Practical 7. Because GitHub Actions runners are temporary and the separate Practical 7 runner's local Docker image is not shared with this repository, this repository regenerates the model artifact and builds the corresponding image in its own run before publishing it. It does not claim to reuse the previous runner's local image.

## Result

A successful workflow run confirms that the tested Student Result Prediction image was published to GHCR, pulled back by its commit tag, and passed health plus PASS/FAIL API checks.
