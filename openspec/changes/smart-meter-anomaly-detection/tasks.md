# Tasks: Smart Meter Anomaly Detection System

## 1. Project Setup

- [ ] 1.1 Create project directory structure: `src/`, `data/`, `models/`
- [ ] 1.2 Create `requirements.txt` with `pandas`, `numpy`, `scikit-learn`, `streamlit`, `matplotlib`, `joblib`
- [ ] 1.3 Download UCI ElectricityLoad Dataset CSV and place in `data/`

## 2. Data Pipeline (`src/data_pipeline.py`)

- [ ] 2.1 Implement `load_dataset(path)` — read CSV, parse timestamp column as DatetimeIndex
- [ ] 2.2 Implement `handle_missing(df)` — apply forward-fill (ffill) to all columns
- [ ] 2.3 Implement `normalize(df)` — apply `MinMaxScaler` per-column, skip zero-variance columns
- [ ] 2.4 Implement `load_and_preprocess(path)` — orchestrates steps 2.1→2.2→2.3, returns cleaned normalized DataFrame
- [ ] 2.5 Verify missing value rate < 0.3% before and after fill (print a report)

## 3. Feature Engineering (`src/feature_engineering.py`)

- [ ] 3.1 Implement `add_lag_features(df, n)` — add `lag_1` … `lag_n` columns using `df.shift()`
- [ ] 3.2 Implement `add_rolling_features(df, k)` — add `rolling_mean_k` and `rolling_std_k` columns
- [ ] 3.3 Implement `add_time_features(df)` — extract `hour` and `weekday` from DatetimeIndex
- [ ] 3.4 Implement `build_feature_matrix(df, n=2, k=4)` — calls 3.1→3.2→3.3, drops NaN rows, returns feature matrix

## 4. Isolation Forest Model (`src/model.py`)

- [ ] 4.1 Implement `train_model(X)` — fit `IsolationForest(contamination=0.05, random_state=42)` and return fitted model
- [ ] 4.2 Implement `get_anomaly_scores(model, X)` — return `model.decision_function(X)` array
- [ ] 4.3 Implement `get_anomaly_labels(model, X)` — return `model.predict(X)` array (`-1` anomaly, `1` normal)
- [ ] 4.4 Implement `save_model(model, path)` and `load_model(path)` using `joblib`
- [ ] 4.5 Add a `__main__` block that runs the full training pipeline and saves `models/model.pkl`

## 5. Model Validation (`src/validation.py`)

- [ ] 5.1 Implement `inject_anomalies(series, n=10, random_state=42)` — inject 5 spike (×5) and 5 drop (×0.1) anomalies using Jokar et al. method; return modified series and ground-truth index list
- [ ] 5.2 Implement `compute_metrics(y_true, y_pred)` — return Precision, Recall, F1-Score using `sklearn.metrics`
- [ ] 5.3 Implement Z-Score baseline detector (`zscore_baseline(series, threshold=3.0)`)
- [ ] 5.4 Implement 3σ moving-average baseline detector (`moving_avg_baseline(series, window=4, sigma=3.0)`)
- [ ] 5.5 Implement `compare_methods(series)` — runs all three methods on the same injected data, prints a comparison table of Precision / Recall / F1
- [ ] 5.6 Verify Isolation Forest F1 > both baselines and print result

## 6. Streamlit Dashboard (`app.py`)

- [ ] 6.1 Add consumer selectbox widget (MT_001–MT_370) in the sidebar
- [ ] 6.2 Load preprocessed data and trained model on app startup (cache with `@st.cache_resource`)
- [ ] 6.3 Build feature matrix for selected consumer and run prediction
- [ ] 6.4 Render time-series line chart with normal points in blue and anomaly points as red markers
- [ ] 6.5 Render anomaly score chart below the time-series chart
- [ ] 6.6 Show a summary table of detected anomaly events (timestamp + anomaly score)
- [ ] 6.7 Add a CSV download button that exports the anomaly event table

## 7. Integration & Verification

- [ ] 7.1 Run `python src/model.py` end-to-end — confirm `models/model.pkl` is created without errors
- [ ] 7.2 Run `python src/validation.py` — confirm Isolation Forest F1 > Z-Score and 3σ baselines
- [ ] 7.3 Run `streamlit run app.py` — confirm dashboard loads at `localhost:8501` with no errors
- [ ] 7.4 Test consumer switch in dropdown — confirm charts and table refresh correctly
- [ ] 7.5 Test CSV export — confirm downloaded file contains only anomaly rows with correct columns
