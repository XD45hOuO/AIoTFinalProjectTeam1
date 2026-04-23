# Spec: data-pipeline

## ADDED Requirements

### Requirement: Load UCI ElectricityLoad Dataset
The system SHALL load the UCI ElectricityLoad CSV dataset (370 consumers, 15-min intervals, 2011–2014) and parse the timestamp column into a proper datetime index.

#### Scenario: Successful load
- **WHEN** the dataset CSV is present at the configured path
- **THEN** the system returns a DataFrame with a DatetimeIndex and 370 consumer columns (MT_001–MT_370)

#### Scenario: Missing file
- **WHEN** the dataset file is not found
- **THEN** the system raises a clear FileNotFoundError with the expected path

### Requirement: Handle missing values
The system SHALL fill missing values using forward-fill (ffill) interpolation. The missing rate SHALL be less than 0.3% of total records.

#### Scenario: Forward fill applied
- **WHEN** the loaded DataFrame contains NaN values
- **THEN** all NaN values are filled via forward-fill before returning the cleaned DataFrame

### Requirement: Normalize with MinMaxScaler
The system SHALL normalize each consumer's time series to [0, 1] using MinMaxScaler, applied per-column.

#### Scenario: Normalization applied
- **WHEN** the cleaned DataFrame is passed to the normalizer
- **THEN** all values in the returned DataFrame are within [0.0, 1.0]

### Requirement: Fix random seed
The system SHALL fix `random_state=42` for all random operations to guarantee reproducibility of results across runs.

#### Scenario: Reproducible output
- **WHEN** the pipeline is run twice with the same input data
- **THEN** the output DataFrames and model results are identical
