# Suicide Rate Prediction with XGBoost in KNIME

This project builds an end‑to‑end machine learning pipeline in KNIME to predict suicide rates (suicides per 100k population) using demographic and economic data.  
The workflow covers data cleaning, feature engineering, model training with XGBoost, evaluation, and visualisation.

## Project overview

We use the “Suicide Rates Overview 1985–2016” dataset to model suicide_rate as a regression target.  
The main features include country, year, sex, age group, generation, population, GDP indicators, HDI, and several engineered features.

The KNIME workflow does the following:

1. **Data import & cleaning**
   - Read the CSV and fix data types.
   - Clean `gdp_for_year ($)` from string with `$` and commas to a numeric `gdp_for_year`.
   - Rename `suicides/100k pop` → `suicide_rate` and `gdp_per_capita ($)` → `gdp_per_capita`.
   - Handle missing values (median imputation for HDI and GDP, drop rows with missing target).

2. **Feature engineering**
   - Log transforms: `log_population`, `log_gdp_per_capita`, `log_gdp_for_year`.
   - Country‑wise lag features: `lag1_rate`, `lag2_rate` computed per country.
   - Target encoding: `country_mean_rate` (average suicide_rate per country).

3. **Pre‑model preparation**
   - Column filtering to select the final feature set:
     `suicide_rate`, `year`, `log_population`, `log_gdp_per_capita`,
     `log_gdp_for_year`, `HDI for year`, `gdp_per_capita`, `lag1_rate`,
     `lag2_rate`, `country_mean_rate`, `sex`, `age`, `generation`.
   - Domain calculation and type casting for categorical and numeric columns.

4. **Train / test split**
   - Stratified 80/20 split by `country` to preserve country distribution.

5. **Model training**
   - XGBoost Tree Ensemble Learner (regression) with tuned hyperparameters:
     learning rate 0.05, max depth 6, 100 rounds, subsample 0.8, colsample_bytree 0.8.
   - Target column: `suicide_rate`.

6. **Prediction & evaluation**
   - XGBoost Predictor on both training and test partitions.
   - Numeric Scorer for metrics:
     - Training: R² ≈ 1.0, RMSE ≈ 0.145, MAE ≈ 0.104.
     - Test:    R² ≈ 0.978, RMSE ≈ 2.722, MAE ≈ 1.104.
   - These results show strong generalisation with low error in suicides per 100k.

7. **Feature importance & visualisation**
   - XGBoost feature importance (Gain) identifies key predictors:
     `country_mean_rate`, `HDI for year`, `lag1_rate`, `age`, `generation`.
   - Visualisations:
     - Predicted vs actual suicide_rate.
     - Residuals plot.
     - Train vs test metric bar chart (R², RMSE, MAE).

## Repository contents

- `workflow/Suicide_Rate_XGBoost_v2.knwf`  
  KNIME workflow implementing the entire pipeline.

- `model/xgb_suicide_model.knime`  
  Trained XGBoost model saved from the learner.

- `reports/`  
  Project report and slides.

- `data/`  
  (Optional) Original dataset or instructions on where to download it.

## How to run

1. Install KNIME Analytics Platform (same major version used for this project).
2. Import the workflow from `workflow/Suicide_Rate_XGBoost_v2.knwf`.
3. Configure the CSV Reader path to point to your local copy of the dataset.
4. Execute the workflow from top to bottom.
5. Inspect the Numeric Scorer nodes and visualisation nodes for model performance.

## Collaboration

The main branch contains the stable version of the workflow.  
Contributors can open feature branches, edit the KNIME workflow or documentation, and create pull requests for review.
