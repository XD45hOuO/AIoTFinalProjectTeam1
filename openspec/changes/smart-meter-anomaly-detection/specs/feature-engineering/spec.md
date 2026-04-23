# Spec: feature-engineering

## ADDED Requirements

### Requirement: Generate lag features
The system SHALL generate lag features `lag_1` through `lag_n` (configurable n) for each consumer's time series using a shift operation.

#### Scenario: Lag features created
- **WHEN** feature engineering is run with n=2
- **THEN** the output DataFrame contains columns `lag_1` and `lag_2` with NaN rows from shift dropped

### Requirement: Generate rolling statistics
The system SHALL compute `rolling_mean_k` and `rolling_std_k` over a configurable window size k for each consumer.

#### Scenario: Rolling mean computed
- **WHEN** feature engineering is run with k=4
- **THEN** the output DataFrame contains `rolling_mean_4` computed over a 4-step window

#### Scenario: Rolling std computed
- **WHEN** feature engineering is run with k=4
- **THEN** the output DataFrame contains `rolling_std_4` computed over a 4-step window

### Requirement: Extract cyclical time features
The system SHALL extract `hour` (0–23) and `weekday` (0=Monday, 6=Sunday) from the DatetimeIndex as integer columns.

#### Scenario: Hour and weekday extracted
- **WHEN** feature engineering is applied to a DataFrame with a DatetimeIndex
- **THEN** the output contains integer columns `hour` and `weekday` matching the index timestamps

### Requirement: Drop NaN rows after feature creation
The system SHALL drop all rows containing NaN values introduced by lag/rolling operations before returning the feature matrix.

#### Scenario: NaN rows removed
- **WHEN** lag and rolling features are computed
- **THEN** the returned DataFrame contains no NaN values
