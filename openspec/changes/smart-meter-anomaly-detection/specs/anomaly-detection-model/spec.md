# Spec: anomaly-detection-model

## ADDED Requirements

### Requirement: Train Isolation Forest model
The system SHALL train a scikit-learn `IsolationForest` with `contamination=0.05` and `random_state=42` on the feature matrix produced by the feature engineering step.

#### Scenario: Model trains successfully
- **WHEN** a valid feature matrix (no NaN) is passed to the trainer
- **THEN** a fitted IsolationForest model is returned without errors

### Requirement: Produce anomaly scores
The system SHALL compute an anomaly score for every sample using `decision_function()`, where lower (more negative) values indicate higher anomaly likelihood.

#### Scenario: Scores returned for all samples
- **WHEN** the fitted model scores a feature matrix of N rows
- **THEN** a NumPy array of length N is returned containing one score per sample

### Requirement: Produce binary anomaly labels
The system SHALL produce binary labels using `predict()`: `-1` for anomaly, `1` for normal.

#### Scenario: Labels match score direction
- **WHEN** a sample has an anomaly score below the contamination threshold
- **THEN** its predicted label is `-1`

### Requirement: Persist trained model
The system SHALL save the trained model to disk (e.g., `model.pkl`) so it can be loaded by the dashboard without retraining.

#### Scenario: Model saved and reloaded
- **WHEN** the training script completes
- **THEN** a `model.pkl` file exists and can be loaded with `joblib.load()` to reproduce identical predictions
