# Employee Attrition Prediction & Risk Dashboard

Predicting which employees are likely to leave, using the IBM HR Analytics dataset, and presenting the results in a 3-page Power BI dashboard that HR can act on.

## Overview

- **Goal:** identify employees at high risk of attrition and the factors driving it.
- **Data:** IBM HR Analytics Employee Attrition dataset (1,470 employees, 35 columns).
- **Challenge:** only ~16% of employees leave, so plain accuracy is misleading (always predicting "stay" already scores ~84%). Models were judged on ROC-AUC, PR-AUC and recall instead.
- **Output:** a risk score for every employee (out-of-fold, so nobody is scored by a model that trained on them), grouped into Low / Medium / High risk bands.

## Approach

1. **Cleaning:** dropped constant columns (EmployeeCount, Over18, StandardHours); kept EmployeeNumber as an ID.
2. **EDA:** attrition rate by category, numeric distributions by attrition, correlation analysis.
3. **Feature engineering:** `IncomeToLevel`, `PromoRatio`, `TenureRatio`, `CompaniesPerYear`, `OT_LowSatisfaction`.
4. **Encoding:** one-hot encoding for nominal columns only; ordinal ratings (e.g. JobSatisfaction) kept numeric.
5. **Imbalance handling:** class weights in every model.
6. **Modeling:** 7 models in scikit-learn pipelines: Logistic Regression, Random Forest, SVM, XGBoost, LightGBM, CatBoost, AdaBoost.
7. **Tuning:** Logistic Regression (GridSearchCV) and Random Forest (RandomizedSearchCV), on the training set only.
8. **Evaluation:** 5-fold stratified CV on train, then a held-out 30% test set. All curves and AUCs use predicted probabilities.
9. **Threshold tuning:** chosen on out-of-fold training predictions using F2 (recall-weighted), then applied to the test set.
10. **Interpretability:** Logistic Regression odds ratios, Random Forest importances, SHAP.

## Results (held-out test set)

| Model | ROC-AUC | PR-AUC | Accuracy | Precision | Recall | F1 |
|---|---|---|---|---|---|---|
| **Logistic Regression** | **0.803** | **0.576** | 0.778 | 0.390 | **0.676** | **0.495** |
| SVM | 0.792 | 0.515 | 0.862 | 0.679 | 0.268 | 0.384 |
| AdaBoost | 0.783 | 0.459 | 0.844 | 0.531 | 0.239 | 0.330 |
| CatBoost | 0.770 | 0.515 | 0.825 | 0.450 | 0.380 | 0.412 |
| XGBoost | 0.761 | 0.479 | 0.810 | 0.408 | 0.408 | 0.408 |
| LightGBM | 0.749 | 0.450 | 0.837 | 0.489 | 0.324 | 0.390 |
| Random Forest | 0.738 | 0.375 | 0.832 | 0.459 | 0.239 | 0.315 |

**Logistic Regression performed best** (ROC-AUC 0.80), catching about two-thirds of leavers on unseen data, and it is also the easiest to explain.

SVM has the highest accuracy (86.2%) but recall of only 26.8%: it mostly predicts "stay", which is why accuracy alone is misleading here.

Note: the test set is small (~440 rows, ~70 leavers), so scores can shift a few points with a different split.

## Key Findings

- **Risk bands separate well:** actual attrition is ~5% in the Low band, ~10% in Medium and ~35-40% in High, against a 16.1% company average (566 / 443 / 461 employees).
- **Highest-risk profile:** employees aged 18-25, earning under 3K/month, in their first 2 years, and working overtime.
- **Overtime:** ~30% attrition for employees working overtime vs ~10% for those who don't.
- **Drivers that raise attrition odds:** OverTime, frequent business travel, years since last promotion, single marital status, and certain roles (e.g. Laboratory Technician, Sales Representative).
- **Drivers that lower attrition odds:** higher environment, relationship and job satisfaction, age, and stock option level.

## Dashboard (Power BI)

**1. Executive Overview**: KPI cards (employees, attrition rate, high-risk count), attrition by risk band, by job role, and risk mix by department.


<img width="1920" height="1080" alt="empag1" src="https://github.com/user-attachments/assets/6f5a4111-a25e-4e3a-b56f-7890cd036427" />



**2. Employee Deep Dive**: a ranked list of high-risk employees plus attrition by age group, income band, tenure band and overtime.



<img width="1920" height="1080" alt="em pg2 wo high" src="https://github.com/user-attachments/assets/a9df2046-19d7-4e9c-b933-cbe795da52f3" />



<img width="1920" height="1080" alt="em pg3 w high" src="https://github.com/user-attachments/assets/4a4c90a7-f3ab-4dd1-9e79-bc4e140be974" />



**3. Model Insights**: model comparison (ROC-AUC, recall, etc.) and the top drivers that raise and lower attrition odds.


<img width="1920" height="1080" alt="em pag3" src="https://github.com/user-attachments/assets/f9c8db98-f2b8-4f2b-9bc9-86bddcec5e6b" />



## Notes on the Risk Score

Models were trained with class weighting, which pushes scores upward (average score is ~37% vs a true attrition rate of ~16%). The score is therefore best used to **rank** employees and form risk bands, not as a literal probability of leaving. The band-level attrition rates above are real observed rates.



## Tech Stack

Python, pandas, scikit-learn, XGBoost, LightGBM, CatBoost, SHAP, matplotlib, seaborn, Power BI (DAX)

## Dataset

IBM HR Analytics Employee Attrition & Performance dataset (publicly available on Kaggle). It is a fictional dataset created by IBM data scientists.


