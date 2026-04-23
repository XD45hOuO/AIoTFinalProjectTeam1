# Design: Smart Meter Anomaly Detection System

## Context
Greenfield Python project. No existing codebase. Target environment is a local machine running Python 3.10+. The Streamlit dashboard runs on `localhost:8501`. The UCI dataset is a single CSV file (~60MB) loaded once at startup.

## Goals / Non-Goals
**Goals:**
- Reproducible, single-command pipeline from raw CSV to Streamlit dashboard
- Model persisted to disk so dashboard loads instantly without retraining
- Validation script with F1 comparison against Z-Score and 3σ baselines

**Non-Goals:**
- Real-time streaming ingestion (batch only for this project)
- Cloud deployment or edge device testing (future work)
- Multi-user authentication on the dashboard

## Decisions

### Project structure
```
src/
  data_pipeline.py      # load, clean, normalize
  feature_engineering.py
  model.py              # train, predict, persist
  validation.py         # synthetic injection + metrics
app.py                  # Streamlit dashboard entrypoint
data/                   # raw UCI CSV
models/                 # saved model.pkl
requirements.txt
```
**Why:** Flat `src/` keeps imports simple for a small academic project.

### Model persistence with joblib
Save/load the trained model with `joblib.dump` / `joblib.load` into `models/model.pkl`.
**Why:** joblib is already a scikit-learn transitive dependency and handles NumPy arrays efficiently.

### Single consumer at a time in the dashboard
The dashboard scores and displays one consumer per session selection rather than all 370 at once.
**Why:** Scoring all 370 consumers on every page load would be slow and unnecessary for an audit use case.

## Risks / Trade-offs
- **Risk:** MinMaxScaler fitted on training data; if applied to a consumer with all-zero consumption, produces NaN. → **Mitigation:** Add a guard that skips columns with zero variance.
- **Risk:** Lag/rolling features introduce significant NaN rows at the start of each series. → **Mitigation:** Drop NaN rows after feature creation, document the effective date range loss.

## Migration Plan
No migration needed — this is a new project. Setup:
1. `pip install -r requirements.txt`
2. Place UCI CSV in `data/`
3. `python src/model.py` (trains and saves model)
4. `streamlit run app.py`

## Open Questions
- Which single consumer should be the default selection in the dashboard? (Suggest MT_001)
- Should validation compare against all 370 consumers or a fixed sample? (Suggest 1 representative consumer for the report)
