# BigQuery ML (BQML)

**Impact:** High

## What This Covers

Creating, training, evaluating, and serving machine learning models directly inside BigQuery using SQL, including built-in model types and imported TensorFlow/ONNX models.

## Why It Matters

BQML eliminates the need to export data to external ML tools. Models train where the data lives, reducing data movement, simplifying pipelines, and enabling analysts without Python/ML expertise to build models.

## Example

```sql
-- Create a logistic regression model
CREATE OR REPLACE MODEL `project.dataset.churn_model`
OPTIONS(
  model_type = 'LOGISTIC_REG',
  input_label_cols = ['churned'],
  auto_class_weights = TRUE,
  max_iterations = 20
) AS
SELECT
  tenure_months,
  monthly_charges,
  contract_type,
  payment_method,
  churned
FROM `project.dataset.customers_train`;

-- Evaluate model performance
SELECT *
FROM ML.EVALUATE(MODEL `project.dataset.churn_model`,
  (SELECT * FROM `project.dataset.customers_test`));

-- Generate predictions
SELECT
  customer_id,
  predicted_churned,
  predicted_churned_probs
FROM ML.PREDICT(MODEL `project.dataset.churn_model`,
  (SELECT * FROM `project.dataset.customers_score`));

-- Time-series forecasting with ARIMA_PLUS
CREATE OR REPLACE MODEL `project.dataset.sales_forecast`
OPTIONS(model_type = 'ARIMA_PLUS', time_series_timestamp_col = 'sale_date',
  time_series_data_col = 'daily_revenue', auto_arima = TRUE, horizon = 30) AS
SELECT sale_date, SUM(revenue) AS daily_revenue
FROM `project.dataset.sales` GROUP BY sale_date;

SELECT * FROM ML.FORECAST(MODEL `project.dataset.sales_forecast`,
  STRUCT(30 AS horizon, 0.95 AS confidence_level));
```

## Edge Cases / Pitfalls

- **Cost:** Model training consumes slots (on-demand: billed per bytes processed; flat-rate: consumes reservation slots). Large training jobs can be expensive -- use `max_iterations` and `early_stop` to control cost.
- **Supported model types:** LINEAR_REG, LOGISTIC_REG, KMEANS, BOOSTED_TREE_CLASSIFIER/REGRESSOR, ARIMA_PLUS, DNN_CLASSIFIER/REGRESSOR, AUTOML_CLASSIFIER/REGRESSOR, imported TensorFlow and ONNX models.
- **Feature preprocessing:** BQML auto-encodes categoricals (one-hot) and standardizes numerics by default. Override with `TRANSFORM` clause for custom preprocessing.
- **Model size limits:** Imported TensorFlow and ONNX models can be up to 450 MB. Built-in model types (LINEAR_REG, KMEANS, etc.) do not have a documented user-visible size cap.
- **Prediction quotas:** `ML.PREDICT` is subject to standard query limits. For serving at scale, export the model to Vertex AI endpoints.
- **Data leakage:** BQML does not auto-split train/test. You must partition data yourself or use `DATA_SPLIT_METHOD`.
