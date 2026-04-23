# Proposal: Smart Meter Anomaly Detection System

## Why
Electricity theft accounts for over 90% of non-technical losses (NTL), costing ~$101.2B globally per year. Taiwan has 4.23M smart meters deployed but no automated anomaly analysis — manual meter reading cycles (monthly) mean theft goes undetected for weeks. We need an automated, unsupervised detection pipeline that works without labeled data and is deployable on edge hardware.

## What Changes
- **NEW** End-to-end Python data pipeline ingesting UCI ElectricityLoad Dataset (370 consumers, 15-min intervals)
- **NEW** Feature engineering module: lag features, rolling statistics, and cyclical time features
- **NEW** Isolation Forest model training (contamination=0.05, random_state=42) with anomaly scoring
- **NEW** Synthetic anomaly injection (Spike ×5, Drop ×0.1, n=10, seed=42) for validation and F1 evaluation
- **NEW** Streamlit dashboard with time-series visualization, red anomaly markers, anomaly score display, and CSV export

## Capabilities
- **New Capabilities:**
  - `data-pipeline` — data loading, preprocessing, normalization
  - `feature-engineering` — lag, rolling stats, cyclical time features
  - `anomaly-detection-model` — Isolation Forest training and scoring
  - `model-validation` — synthetic anomaly injection, Precision/Recall/F1 evaluation
  - `streamlit-dashboard` — visualization, anomaly log export

- **Modified Capabilities:** *(none — this is a greenfield project)*

## Impact
- New Python project structure with `requirements.txt`
- Dependencies: `pandas`, `numpy`, `scikit-learn`, `streamlit`, `matplotlib`
- Runs locally at `localhost:8501`
- No external API dependencies; dataset sourced from UCI ML Repository
