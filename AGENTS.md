# Agent Instructions

## Project Overview

Kaggle competition project: **Irrigation Need Prediction** — multiclass classification (3 classes) of irrigation needs based on agricultural/environmental features.

## Project Structure

- `eda.ipynb` — Exploratory data analysis: distributions, correlations, outliers, class imbalance
- `preprocessing.ipynb` — Feature engineering, model training (LinearSVC, LogisticRegression, XGBoost), hyperparameter tuning, submission generation
- `formater.md` — Custom agent: Notebook Markdown Formatter
- `duckdb/` — DuckDB related files
- Data files (`*.csv`) are git-ignored

## Conventions

- **Language**: Notebooks use Spanish for markdown explanations
- **Data pipeline**: `sklearn.pipeline.Pipeline` with `ColumnTransformer` (StandardScaler for numeric, OneHotEncoder for categorical)
- **Evaluation metric**: `balanced_accuracy` for hyperparameter tuning; macro/weighted F1 and confusion matrices for reporting
- **Target column**: `Irrigation_Need`
- **Index column**: `id`
- **GPU**: XGBoost uses `device='cuda'`

## Key Libraries

pandas, scikit-learn, xgboost, seaborn, matplotlib, numpy
