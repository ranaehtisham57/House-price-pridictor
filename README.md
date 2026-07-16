# 🏡 Real Estate Price Prediction using Machine Learning

## Overview

This project presents a complete end-to-end machine learning workflow for predicting real estate sale prices. The primary objective was not only to build a predictive model but also to demonstrate the complete data preprocessing pipeline required for real-world datasets.

The dataset contained numerous data quality issues, including missing values, inconsistent categorical values, invalid geographic coordinates, outliers, and highly skewed numerical features. A comprehensive preprocessing pipeline was developed to clean and prepare the data before training regression models.

---

## Project Objectives

* Clean and preprocess a real-world style real estate dataset.
* Handle missing and inconsistent data.
* Perform feature engineering and encoding.
* Detect and handle outliers.
* Reduce feature skewness using appropriate transformations.
* Scale numerical features.
* Train and evaluate multiple regression models.
* Compare model performance using the R² metric.

---

## Dataset Preprocessing

### Data Cleaning

The following preprocessing steps were performed:

* Removed duplicate records.
* Removed unnecessary identifier columns.
* Corrected inconsistent city and state names.
* Fixed invalid latitude and longitude values.
* Converted date columns into proper datetime format.
* Reconstructed date values using separate year, month, and day columns where required.

---

### Missing Value Handling

Different techniques were used depending on the nature of each feature.

* Mean imputation for appropriate numerical columns.
* Mode imputation for categorical columns.
* Dropped columns containing excessive missing values when they provided little predictive value.

---

### Feature Engineering

The following feature engineering techniques were applied:

* Extracted Year, Month, and Day from date columns.
* One-Hot Encoding for categorical features including:

  * City
  * State
  * Property Type
  * Heating Type
  * Cooling Type
* Removed textual columns that were not useful for prediction.

---

### Exploratory Data Analysis

Exploratory analysis included:

* Correlation analysis
* Distribution analysis
* Skewness analysis
* Histogram visualization
* Boxplot visualization
* Outlier detection using the IQR method

---

### Outlier Handling

Instead of deleting observations, outliers were handled using **IQR-based Winsorization (clipping)**.

Values below the lower bound were replaced with the lower bound, while values above the upper bound were replaced with the upper bound. This approach preserved all observations while reducing the influence of extreme values on the regression model.

---

### Feature Transformation

Several numerical features exhibited high positive skewness.

To improve their distributions, the **QuantileTransformer** was applied to selected features, transforming them into approximately normal distributions while reducing the influence of extreme values.

---

### Feature Scaling

Remaining numerical features were scaled using **RobustScaler**, which is less sensitive to outliers than standard normalization techniques.

---

## Machine Learning Models

The following regression algorithms were implemented and evaluated:

* Linear Regression
* Ridge Regression
* Lasso Regression
* Polynomial Regression

---

## Model Evaluation

The models were evaluated using the **Coefficient of Determination (R² Score)**.

Initial model performance was poor because the dataset contained numerous influential outliers. After applying IQR-based winsorization and feature transformation, the Linear Regression model showed a substantial improvement in predictive performance.

---

## Technologies Used

* Python
* NumPy
* Pandas
* Matplotlib
* Seaborn
* Scikit-learn

---

## Skills Demonstrated

* Data Cleaning
* Exploratory Data Analysis (EDA)
* Feature Engineering
* Missing Value Imputation
* Outlier Handling
* Data Transformation
* Feature Scaling
* Machine Learning
* Regression Analysis
* Model Evaluation
* End-to-End Data Science Workflow

---

## Project Structure

```text
├── Real_Estate_Price_Prediction.ipynb
├── dataset.csv
├── README.md
└── requirements.txt
```

---

## Future Improvements

* Hyperparameter tuning using GridSearchCV.
* Cross-validation for more robust evaluation.
* Feature selection techniques.
* Tree-based ensemble models such as Random Forest and Gradient Boosting.
* XGBoost and LightGBM for improved predictive performance.

---

## Conclusion

This project demonstrates the complete workflow of a supervised machine learning regression problem, starting from raw, messy data and progressing through preprocessing, feature engineering, transformation, scaling, model training, and evaluation. It highlights the importance of data preparation in building effective machine learning models and provides practical experience with real-world data preprocessing techniques.
