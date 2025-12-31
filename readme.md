# Suicide Rate Prediction with XGBoost and J48 in KNIME
This project builds an end‑to‑end machine learning pipeline in KNIME to predict suicide rates (suicides per 100k population) using demographic and economic data.  
The workflow covers data cleaning, feature engineering, model training with **XGBoost** and **J48 (WEKA Decision Tree)**, evaluation, and visualisation.

This project implements an end-to-end machine learning workflow in **KNIME Analytics Platform** to analyse and model suicide rates using demographic and economic data.

Two complementary models are developed:
- **XGBoost Regression** to predict the exact suicide rate (suicides per 100k population)
- **J48 (WEKA Decision Tree) Classification** to classify suicide risk into **Low / Medium / High** categories

The workflow covers data cleaning, feature engineering, model training, evaluation, interpretability, and visualisation.

---

## Project Overview

The project uses the **“Suicide Rates Overview 1985–2016”** dataset to study suicide patterns across countries, age groups, and years.

The analysis answers two related research questions:

1. **Regression:**  
   Can we accurately predict the numeric suicide rate using demographic, economic, and historical features?

2. **Classification:**  
   Can we explain suicide risk by age and other factors using interpretable decision-tree rules?

This dual-model approach provides both **high predictive accuracy** and **clear interpretability**.

---

## Dataset & Targets

**Dataset:** Suicide Rates Overview 1985–2016

**Targets:**
- `suicide_rate` (numeric) → XGBoost regression
- `risk_class` (categorical: Low / Medium / High) → J48 classification


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
   - Creation of `risk_class` for classification:
     - Low: `suicide_rate < 5`
     - Medium: `5 ≤ suicide_rate < 15`
     - High: `suicide_rate ≥ 15`

3. **Pre‑model preparation**
   - Column filtering to select the final feature set:
     `suicide_rate`, `risk_class`, `year`, `log_population`, `log_gdp_per_capita`,
     `log_gdp_for_year`, `HDI for year`, `gdp_per_capita`, `lag1_rate`,
     `lag2_rate`, `country_mean_rate`, `sex`, `age`, `generation`.
   - Domain calculation and type casting for categorical and numeric columns.

4. **Train / test split**
   - Stratified 80/20 split by `country` to preserve country distribution.

5. **Model training**
   - **XGBoost Tree Ensemble Learner (regression)** with tuned hyperparameters:
     learning rate 0.05, max depth 6, 100 rounds, subsample 0.8, colsample_bytree 0.8.
     - Target column: `suicide_rate`.
   - **J48 (WEKA Decision Tree) Learner** for classification:
     - Target column: `risk_class`.
     - Predictors include age, sex, generation, country, year, lag features, economic indicators.
     - Default pruning used for interpretability.

6. **Prediction & evaluation**
   - XGBoost Predictor on both training and test partitions.
     - Numeric Scorer metrics:
       - Training: R² ≈ 1.0, RMSE ≈ 0.145, MAE ≈ 0.104.
       - Test:    R² ≈ 0.978, RMSE ≈ 2.722, MAE ≈ 1.104.
   - WEKA Predictor (J48) applied to the test set.
     - Classification Scorer metrics:
       - Training accuracy: 1.0
       - Test accuracy: 1.0
   - These results show strong generalisation and interpretable risk predictions.

7. **Feature importance & visualisation**
   - XGBoost feature importance (Gain) identifies key predictors:
     `country_mean_rate`, `HDI for year`, `lag1_rate`, `age`, `generation`.
   - Visualisations:
     - Predicted vs actual suicide_rate.
     - Residuals plot.
     - Train vs test metric bar chart (R², RMSE, MAE).
     - J48 decision tree rules for risk_class.

## Repository contents

- `Suicide Rate Prediction Complete workflow.knwf`  
  Just import the project and KNIME workflow implementing both regression and classification pipelines.

- `models/xgb_suicide_model.knime`  
  Trained XGBoost model.

- `models/j48_risk_model.weka`  
  Trained J48 model.

- `reports/`  
  Project report and slides.


## How to run

1. Install KNIME Analytics Platform (same major version used for this project).
2. Import the workflow from `workflow/Suicide_Rate_XGBoost_J48.knwf`.
3. Configure the CSV Reader path to point to your local copy of the dataset.
4. Execute the workflow from top to bottom.
5. Inspect the Numeric Scorer nodes for XGBoost and the Confusion Matrix node for J48.  
   Also explore visualisation nodes for interpretation.

## Collaboration

The main branch contains the stable version of the workflow.  
Contributors can open feature branches, edit the KNIME workflow or documentation, and create pull requests for review.
