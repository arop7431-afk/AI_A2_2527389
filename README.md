# Musanze Harvest Intelligence Service

| Field | Value |
|---|---|
| Full name | [[AROP MALUAL DUWAR KUEK]] |
| Registration number | 25/27389 |
| Institutional username | ug2527389@ines.ac.rw |
| Verification code | AI-A2-7389 |
| Repository URL | [[https://github.com/arop7431-afk/AI_A2_2527389.git]] |
| Final commit hash | [[fc078c690e7d03828cec404977209fc7b6b9f386]] |
| Python version tested | [[ 3.13.14]] |
| Operating system tested | [[ Windows 11 ]] |

> **The dataset is synthetic.** It is generated from a formula and a personal seed.
> Nothing here describes real farms, and the system must not be used for real decisions.

## Problem
A Musanze potato cooperative needs a local, offline-capable tool that (1) estimates
harvest quantity (regression), (2) flags farms at high production risk (classification:
yield per hectare below the median) and (3) groups farms with similar conditions
(clustering). A FastAPI service exposes the models for a future mobile app.

## Setup and run
```bash
python -m venv .venv
# Windows:  .venv\Scripts\activate        
pip install -r requirements.txt

python run_all.py --student-id ug2527389@ines.ac.rw --seed 7389
python predict.py --json samples/valid.json
python predict.py --json samples/invalid.json     # deliberately invalid: exits with code 2
pytest -q
uvicorn app:app --host 127.0.0.1 --port 8000      # docs at http://127.0.0.1:8000/docs


## Project layout
```
run_all.py            one-command entry point
predict.py            CLI prediction (one JSON record)
app.py                FastAPI service (/health, /model-info, /predict)
data/generate_data.py individual synthetic data generator
src/                  config, identity, validation, preprocessing, modeling,
                      clustering, evaluation, schemas, inference, pipeline
tests/test_system.py  pytest suite (success and failure paths)
samples/              valid.json, invalid.json
models/               saved pipelines (generated)
artifacts/            metrics, predictions, plots, fingerprint (generated)
```

## Generated artifacts
`artifacts/regression_metrics.json`, `classification_metrics.json`, `clustering_report.json`,
`test_predictions.csv`, `data_fingerprint.txt`, `data_quality_report.json`, `run_metadata.json`,
`plots/residuals.png`, `plots/confusion_matrix.png`, `plots/clusters.png`;
`models/regression_model.joblib`, `classification_model.joblib`, `cluster_model.joblib`.

## API examples
```bash
curl http://127.0.0.1:8000/health
curl http://127.0.0.1:8000/model-info
curl -X POST http://127.0.0.1:8000/predict -H "Content-Type: application/json" -d @samples/valid.json
curl -X POST http://127.0.0.1:8000/predict -H "Content-Type: application/json" -d @samples/invalid.json   # HTTP 422
```

## Design decisions
* **Seed rule:** `--seed` must equal the last four digits of the registration number found in `--student-id`; anything else is rejected.
* **No leakage:** inputs are the 7 raw features (+3 features derived from the same row). `yield_tons`, yield per hectare, `high_risk` and `farm_id` are never inputs.
* **Train-only fitting:** imputers, scaler and encoder sit inside sklearn Pipelines and are fitted on training rows only. Duplicates are removed *before* the split.
* **Model choice:** by 5-fold cross-validation on the training split; the test set is used only for the final report.
* **Extras:** engineered features (`temp_gap_c`, `fertilizer_per_ha`, `water_index`), risk bands, `human_review_required` flag. No extra packages beyond the required stack (httpx is needed by FastAPI's test client).

## Known limitations
[[### Known Limitations
- Predictions have not been validated using real farming data.
- Model accuracy on real farms is unknown.
- Results may vary across crops, seasons, and locations.
- Every prediction requires human review by a qualified person.
- The system must not be used as the sole basis for financial, insurance, or public-service decisions.

### Synthetic Dataset Statement
**This project uses entirely synthetic data generated for educational and testing purposes.** The dataset does not represent real farms or farmers. All predictions and performance metrics are experimental and are not evidence of real-world agricultural accuracy.]]

## Responsible AI
* **Intended use:** classroom decision-support prototype for a field officer.
* **Limitations:** trained on synthetic data from a known formula; real farms differ; ranges are limited to the generator's ranges (inputs outside are rejected).
* **Risks:** over-trusting a number; unfair treatment of farmers if misused.
* **Prohibited use:** deciding access to finance, insurance or public services; replacing an agronomist.
* **Human review:** every output carries `human_review_required: true`.
