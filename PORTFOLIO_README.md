# NYC Short-Term Rental Price ML Pipeline

## Overview

This project builds a reproducible machine learning pipeline for predicting short-term rental prices in New York City. The goal was not only to train a model, but to create a retrainable workflow that can ingest new data, validate data quality, train a model, track experiments, select a production-ready model, and release pipeline versions.

The project uses a realistic MLOps workflow with MLflow, Hydra, Weights & Biases, pytest, scikit-learn, and GitHub releases.

## Business Problem

A property management company receives new short-term rental listing data on a recurring basis. The company needs a reliable way to estimate typical rental prices from listing attributes and retrain the model as new data arrives.

To support this use case, the pipeline needs to:

- Ingest raw data samples.
- Clean and validate the data before training.
- Split the data into training, validation, and test sets.
- Train and compare models.
- Track artifacts and metrics across runs.
- Promote the best model to production.
- Re-run the pipeline on new incoming data.
- Detect data quality issues before they affect model training.

## What I Built

I implemented an end-to-end machine learning pipeline with the following stages:

1. Data ingestion
2. Exploratory data analysis
3. Data cleaning
4. Data validation tests
5. Train/validation/test split
6. Random Forest model training
7. Hyperparameter search with Hydra
8. Model selection using validation MAE
9. Production model tagging in W&B
10. Test-set verification
11. Pipeline visualization through W&B artifacts
12. GitHub release workflow
13. New-data retraining and pipeline update

## Tools and Technologies

- Python
- pandas
- scikit-learn
- pytest
- MLflow Projects
- Hydra
- Weights & Biases
- Git and GitHub
- GitHub Releases
- WSL Ubuntu

## Pipeline Flow

The pipeline follows this flow:

```text
download
  -> raw_data artifact
  -> basic_cleaning
  -> clean_sample artifact
  -> data_check
  -> train_val_test_split
  -> trainval_data artifact
  -> test_data artifact
  -> train_random_forest
  -> random_forest_export model artifact
  -> promote best model to prod
  -> test_regression_model
```

The final pipeline was also tested against a new data sample, `sample2.csv`, to verify that the released workflow could detect and recover from a real data quality issue.

## Data Cleaning

The cleaning step prepares the raw listing data for validation and model training. It performs three key operations:

### Price Outlier Filtering

Rows with prices outside the configured range are removed:

```text
minimum price: 10
maximum price: 350
```

This prevents extreme or invalid prices from distorting the model.

### Date Conversion

The `last_review` column is converted to a datetime type so the pipeline can handle the field consistently.

### NYC Geolocation Filtering

After testing the released pipeline on `sample2.csv`, one listing was found outside the expected New York City latitude and longitude boundaries. The cleaning step was updated to remove listings outside this range:

```text
longitude: -74.25 to -73.50
latitude: 40.5 to 41.2
```

This update allowed the pipeline to pass data validation on the new sample.

## Data Validation

The project includes automated data tests to protect the pipeline from bad or unexpected input data.

The tests check:

- Expected column names
- Valid neighborhood names
- Proper latitude and longitude boundaries
- Distribution similarity against a reference dataset
- Reasonable row count
- Valid price range

These tests helped catch the `sample2.csv` issue before model training continued.

## Modeling

The model is a Random Forest regressor trained with a scikit-learn pipeline. The pipeline includes preprocessing for text, categorical, ordinal, and numeric features.

The model training step logs:

- R-squared
- Mean Absolute Error
- Feature importance visualization
- Serialized MLflow model artifact

## Hyperparameter Search

Hydra was used to run a small hyperparameter sweep over:

```text
max_depth: 10, 50
n_estimators: 100, 200
```

The best sweep candidate was:

```text
max_depth: 50
n_estimators: 200
validation MAE: 34.1843
```

However, the default model configuration performed slightly better:

```text
validation MAE: 34.1276
validation R2: 0.5519
```

That default model was selected as the production model and tagged as `prod` in W&B.

## Key Results

Model selected for production:

```text
random_forest_export:v0
```

Validation performance:

```text
MAE: 34.1276
