# Spec: model-validation

## ADDED Requirements

### Requirement: Inject synthetic anomalies
The system SHALL inject exactly 10 synthetic anomalies into a consumer's time series using `random_state=42`: 5 Spike anomalies (multiply value by 5×) and 5 Drop anomalies (multiply value by 0.1×), following the Jokar et al. (2016) method.

#### Scenario: Spikes injected
- **WHEN** synthetic injection runs with seed=42
- **THEN** 5 time steps have their consumption multiplied by 5.0

#### Scenario: Drops injected
- **WHEN** synthetic injection runs with seed=42
- **THEN** 5 time steps have their consumption multiplied by 0.1

### Requirement: Compute Precision, Recall, and F1-Score
The system SHALL compare model-predicted anomaly labels against the known injected anomaly positions to compute Precision, Recall, and F1-Score.

#### Scenario: Metrics computed correctly
- **WHEN** predicted labels and ground-truth injection labels are compared
- **THEN** Precision, Recall, and F1-Score are returned as floats between 0.0 and 1.0

### Requirement: Isolation Forest outperforms baseline methods
The system's F1-Score SHALL exceed that of both a Z-Score threshold baseline and a 3σ moving-average rule on the same injected dataset.

#### Scenario: F1 comparison
- **WHEN** all three methods (Isolation Forest, Z-Score, 3σ MA) are evaluated on the same synthetic dataset
- **THEN** Isolation Forest achieves a higher F1-Score than both baselines
