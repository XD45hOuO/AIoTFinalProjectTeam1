# Spec: streamlit-dashboard

## ADDED Requirements

### Requirement: Display time-series chart with anomaly markers
The dashboard SHALL render an interactive time-series line chart for the selected consumer's power usage and overlay red markers at all time steps predicted as anomalous.

#### Scenario: Normal points displayed
- **WHEN** the dashboard loads predictions for a consumer
- **THEN** all normal data points are shown as a continuous line

#### Scenario: Anomaly points highlighted
- **WHEN** the model predicts anomalies for a consumer
- **THEN** those time steps are marked with red dots on the chart

### Requirement: Display anomaly score plot
The dashboard SHALL render a second chart showing the raw anomaly score (from `decision_function()`) for every time step, allowing users to assess score magnitude.

#### Scenario: Score chart rendered
- **WHEN** the consumer selection changes
- **THEN** the anomaly score chart updates to reflect the new consumer's scores

### Requirement: Consumer selector widget
The dashboard SHALL provide a dropdown or selectbox widget letting the user choose any of the 370 consumers (MT_001–MT_370).

#### Scenario: Consumer switch
- **WHEN** the user selects a different consumer from the dropdown
- **THEN** both charts and the anomaly table refresh to show that consumer's data

### Requirement: Export anomaly event log as CSV
The dashboard SHALL provide a download button that exports all detected anomaly events (timestamp, consumer ID, anomaly score) as a CSV file for use by auditors.

#### Scenario: CSV download
- **WHEN** the user clicks the export button
- **THEN** a CSV file is downloaded containing only anomalous rows with columns: timestamp, consumer_id, anomaly_score
