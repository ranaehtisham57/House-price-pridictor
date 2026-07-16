# Data Preprocessing Practice Project — Residential House Sales (Messy Dataset)

## 0. Concepts Covered (source: preprocessing video)

The video walks through a full preprocessing pipeline. The concepts it covers, and that this
project is built to exercise, are:

- Data loading & initial exploration (`.info()`, `.describe()`, `.shape`, `.head()`)
- Identifying and handling **missing values** (column-level and row-level strategies: drop, mean/median/mode impute, forward/backward fill, model-based impute)
- Detecting and removing **duplicate records** (exact and partial/fuzzy duplicates)
- Fixing **incorrect data types** (numbers stored as text, currency strings, mixed date formats)
- Standardizing **inconsistent categorical values** (casing, abbreviations, synonyms)
- **Outlier detection** (IQR, Z-score, visual methods) and treatment (cap, remove, transform)
- Handling **invalid/impossible entries** (negative ages, impossible coordinates, malformed emails)
- **String cleaning / formatting** (whitespace, unicode, tabs, capitalization)
- Reconciling **mixed units** within a single field (sqft vs sqm, km vs miles)
- Reducing **noise** in free text
- Handling **rare categories** (grouping into "Other", frequency thresholding)
- Identifying **highly correlated / redundant features**
- Removing **constant columns**
- Managing **high-cardinality identifier columns**
- Handling **imbalanced target variables** (resampling, class weights, stratified splitting)
- **Feature engineering** (deriving new columns from raw ones)
- **Categorical encoding** (one-hot, ordinal, target/frequency encoding)
- **Feature scaling** (standardization, normalization) and **skewness correction** (log/Box-Cox)
- **Train/test splitting**
- Saving the cleaned dataset and comparing before/after states

---

## 1. Dataset Description

**File:** `house_prices_dirty_dataset.csv`
**Domain:** Residential real-estate listings for a fictional multi-city brokerage (USA)
**Rows:** 9,730 (includes intentionally injected exact and partial duplicates)
**Columns:** 40
**Target-like columns:** `Sale_Price` (regression target), `Is_Foreclosure` (imbalanced binary classification target, ~5% positive)

The dataset simulates an export pulled directly from a real-estate brokerage's listing
database and CRM, merged with agent contact records and buyer feedback forms — exactly the
kind of multi-source, never-been-cleaned data a junior data scientist would receive on day one.
**Nothing in this file has been cleaned.** Every issue below is intentional and must be handled
by you.

---

## 2. Data Dictionary

| Column | Meaning | Intended Type |
|---|---|---|
| Property_ID | Unique property identifier (high cardinality) | string |
| MLS_Number | Multiple Listing Service number — near-duplicate of Property_ID | string |
| Listing_Date | Date the property was listed | date (mixed formats) |
| Sale_Date | Date the property sold (blank if unsold) | date (mixed formats) |
| Street_Address | Street address | string |
| City | City name | string (inconsistent) |
| State | US state | string (full name / abbreviation mixed) |
| Zip_Code | Postal code | string (some invalid) |
| Country | Country | string (inconsistent) |
| Latitude | Property latitude | float |
| Longitude | Property longitude | float |
| Property_Type | House / Condo / Townhouse / Apartment / Villa / Duplex | categorical (nominal, inconsistent) |
| Bedrooms | Number of bedrooms | int (some stored as text/words, some negative) |
| Bathrooms | Number of bathrooms | float |
| Lot_Size_SqFt | Lot size in square feet | float (right-skewed, outliers) |
| Living_Area_Reported | Living area, reported in sqft OR sqm (mixed units, text) | string |
| Year_Built | Year the property was built | int (some invalid/impossible) |
| Year_Renovated | Year of last renovation | float (mostly missing) |
| Garage_Spaces | Number of garage spaces | float |
| Has_Pool | Whether the property has a pool | boolean-like (inconsistent Yes/Y/1/True forms) |
| Has_Garden | Whether the property has a garden | boolean-like (inconsistent forms) |
| Heating_Type | Heating system type | categorical (includes rare categories) |
| Cooling_Type | Cooling system type | categorical (heavily missing, ~64%) |
| School_Rating | Nearby school rating, 1–10 | ordinal float (missing values) |
| Crime_Rate_Index | Local crime index (higher = worse) | float (skewed) |
| Neighborhood_Quality_Score | Composite quality score 1–100 | float (correlated with School_Rating & Crime_Rate_Index) |
| Distance_To_City_Center_Reported | Distance to city center, mixed miles/km (text) | string |
| HOA_Fee_Monthly | Monthly HOA fee | string (currency symbols, commas, "Unknown") |
| Property_Tax_Annual | Annual property tax | string (currency symbols, commas, "Unknown") |
| Days_On_Market | Days the listing was active | int (some negative/invalid) |
| List_Price | Original asking price | string (currency symbols, commas) |
| Sale_Price | Final sale price (**regression target**) | float (missing if unsold, contains outliers & a few negative data-entry errors) |
| Price_Per_SqFt | Sale_Price ÷ living area | float (**highly correlated / redundant with Sale_Price**) |
| Is_Foreclosure | Whether the sale was a foreclosure (**classification target**) | boolean-like, ~5% positive (imbalanced) |
| Currency | Listing currency | **constant column — always "USD"** |
| Agent_Name | Listing agent's name | string |
| Agent_Phone | Agent phone number | string (some contain letters/invalid formats) |
| Agent_Email | Agent email | string (some missing "@") |
| Listing_Description | Free-text marketing description | text (typos, noise) |
| Buyer_Feedback | Free-text post-sale feedback | text (many missing, some blank strings) |

---

## 3. Intentional Data Quality Issues (full list)

- **Missing values:** random scattered NaNs across several numeric/categorical columns; `Cooling_Type` and `Year_Renovated` are heavily missing (whole-column-level problem, 64%/78%); `Sale_Date`/`Sale_Price`/`Price_Per_SqFt` missing together for unsold properties (missing-not-at-random pattern).
- **Duplicates:** 140 exact duplicate rows; 90 partial duplicates (same property re-listed with a modified `Property_ID` suffix and slightly different `Days_On_Market`).
- **Incorrect data types:** `List_Price`, `Property_Tax_Annual`, `HOA_Fee_Monthly` stored as text with `$`, commas, or "Unknown"/"N/A"/"TBD"; `Bedrooms` sometimes stored as words ("three") or with a "beds" suffix; `Year_Built` sometimes stored as text.
- **Inconsistent categorical values:** `Property_Type`, `Has_Pool`, `Has_Garden`, `Is_Foreclosure`, `Country`, `City`, `State` all have multiple casing/abbreviation/synonym variants (e.g., Yes/YES/Y/1/True/true).
- **Outliers:** extreme `Sale_Price` values (mansions/data-entry errors), extreme `Lot_Size_SqFt` values, a few negative `Sale_Price` entries.
- **Invalid entries:** negative `Bedrooms` and `Days_On_Market`; impossible `Latitude`/`Longitude` (999 / -999); impossible `Year_Built` (2099, 1750, 20); malformed `Zip_Code`; phone numbers containing letters; emails missing "@".
- **Formatting problems:** leading/trailing spaces, tabs, mixed capitalization, and accented/unicode characters injected into address, city, state, and name fields.
- **Mixed units:** `Living_Area_Reported` mixes sqft and sqm in the same column; `Distance_To_City_Center_Reported` mixes km and miles in the same column.
- **Noise:** typos (duplicated characters), leetspeak substitutions ("e"→"3"), and inconsistent free text in `Listing_Description` and `Buyer_Feedback`.
- **Rare categories:** `Heating_Type` includes low-frequency categories ("Wood", "Coal").
- **Highly correlated features:** `Price_Per_SqFt` is mathematically derived from `Sale_Price`; `Neighborhood_Quality_Score` is derived from `School_Rating` and `Crime_Rate_Index`.
- **Redundant columns:** `MLS_Number` is a near 1:1 mirror of `Property_ID`.
- **Constant column:** `Currency` is always "USD".
- **High-cardinality columns:** `Property_ID`, `MLS_Number`, `Street_Address`, `Agent_Email`.
- **Imbalanced target:** `Is_Foreclosure` is ~5% positive / ~95% negative (across all its inconsistent spellings).
- **Date inconsistency:** `Listing_Date` and `Sale_Date` appear in at least six different formats, with a handful of impossible dates ("32/14/2025", "Feb 30 2023").

---

## 4. Feature Engineering Opportunities

- `Property_Age` = current year − `Year_Built`
- `Was_Renovated` = binary flag from `Year_Renovated` presence
- `Years_Since_Renovation`
- `Price_Per_SqFt` recomputed cleanly (after fixing units/currency) and compared to the original dirty column
- `Total_Rooms` = `Bedrooms` + `Bathrooms`
- `Listing_To_Sale_Days` recomputed from cleaned `Listing_Date`/`Sale_Date` and compared against `Days_On_Market`
- `Price_Gap` = `List_Price` − `Sale_Price`
- `Price_Gap_Pct` = `Price_Gap` / `List_Price`
- `Is_Luxury` = flag if `Sale_Price` above a chosen percentile threshold
- `Has_HOA` = binary flag from `HOA_Fee_Monthly`
- `Neighborhood_Score_Bucket` = binned version of `Neighborhood_Quality_Score` (ordinal encoding practice)

---

## 5. Categorical Feature Types Present

- **Binary:** `Has_Pool`, `Has_Garden`, `Is_Foreclosure`
- **Nominal:** `Property_Type`, `Heating_Type`, `Cooling_Type`, `City`, `State`, `Country`
- **Ordinal:** `School_Rating` (once cleaned/binned), `Neighborhood_Score_Bucket` (engineered)
- **High-cardinality:** `Property_ID`, `MLS_Number`, `Street_Address`, `Agent_Name`, `Agent_Email`

## 6. Numerical Feature Types Present

- Integers: `Bedrooms`, `Days_On_Market`, `Garage_Spaces`
- Floats: `Bathrooms`, `Lot_Size_SqFt`, `Sale_Price`, `Crime_Rate_Index`
- Right-skewed distributions: `Lot_Size_SqFt`, `Crime_Rate_Index`, `Days_On_Market`
- Roughly normal distributions: `School_Rating`, `Year_Built`
- Values with legitimate negatives after cleaning: `Price_Gap` (engineered)

---

## 7. Your Assignment

Complete the following tasks **in order**, in a Jupyter notebook or script. Do **not** skip
steps even if a column looks fine at first glance — inspect everything.

✔ Load the dataset and record its raw shape, dtypes, and memory footprint
✔ Explore the dataset (`.info()`, `.describe(include='all')`, unique value counts per column)
✔ Identify and document every column's *actual* vs *intended* data type
✔ Quantify missing values per column (count + percentage) and decide a justified strategy per column (drop / impute / flag)
✔ Detect and remove exact duplicate rows
✔ Detect and resolve partial/fuzzy duplicate rows (same property listed twice)
✔ Standardize all inconsistent categorical values (`Property_Type`, `Has_Pool`, `Has_Garden`, `Is_Foreclosure`, `Country`, `City`, `State`) into single canonical categories
✔ Clean and convert all currency-formatted columns (`List_Price`, `Property_Tax_Annual`, `HOA_Fee_Monthly`) into proper numeric types, handling "Unknown"/"N/A"/"TBD" explicitly
✔ Parse and standardize all date columns into a single consistent date type, handling impossible dates
✔ Split and standardize the mixed-unit columns (`Living_Area_Reported`, `Distance_To_City_Center_Reported`) into single-unit numeric columns
✔ Clean whitespace, tabs, unicode characters, and casing issues across all text/categorical fields
✔ Detect outliers (choose and justify a method: IQR, Z-score, or visual) in `Sale_Price`, `Lot_Size_SqFt`, and at least two other numeric columns
✔ Decide and apply an outlier treatment strategy (cap, remove, transform, or flag) — justify your choice per column
✔ Identify and resolve invalid/impossible entries (negative bedrooms, negative days on market, impossible coordinates, impossible years, malformed emails/phones/zip codes)
✔ Identify rare categories in `Heating_Type` and decide how to group/handle them
✔ Identify and remove/flag the constant column
✔ Identify and handle the redundant/near-duplicate identifier column
✔ Identify highly correlated numeric features (correlation matrix / heatmap) and decide what to drop or keep
✔ Engineer at least 6 new features from the list in Section 4 (or your own)
✔ Detect skewness in numeric columns and apply an appropriate transformation where needed
✔ Encode categorical variables appropriately (choose encoding method per variable type and justify)
✔ Scale numerical features appropriately for downstream modeling
✔ Analyze and address the class imbalance in `Is_Foreclosure` (document the imbalance ratio, then apply a strategy: resampling, class weighting, or stratified evaluation — justify your choice)
✔ Split the cleaned dataset into train/test sets (stratified where appropriate)
✔ Save the fully cleaned dataset to a new CSV file
✔ Produce a short before/after comparison report: shape, dtypes, missing-value totals, duplicate counts, and at least 3 summary statistics, comparing raw vs cleaned data

---

## 8. Evaluation Checklist

Use this to self-verify your work once finished.

**Data Integrity**
- [ ] No duplicate rows remain (exact or partial)
- [ ] Every column has a single, correct data type
- [ ] All date columns are proper datetime objects with one consistent format
- [ ] No impossible values remain (negative ages/bedrooms, coordinates out of range, invalid years)

**Missing Data**
- [ ] Every column's missing-value strategy is documented and justified
- [ ] No unintended NaNs remain after imputation (unless intentionally flagged)

**Categorical Cleaning**
- [ ] Every categorical column has been reduced to canonical categories (no more "Yes"/"YES"/"Y"/"1")
- [ ] Rare categories have been explicitly handled, not silently ignored

**Numeric Cleaning**
- [ ] Currency/formatted numeric columns are true numeric dtypes
- [ ] Mixed-unit columns are split into a single consistent unit
- [ ] Outliers were detected with a stated method and treated with a stated, justified strategy
- [ ] Skewed features were identified and transformed appropriately

**Structure**
- [ ] Constant column identified and handled
- [ ] Redundant/near-duplicate ID column identified and handled
- [ ] High-cardinality columns were addressed appropriately (not blindly one-hot encoded)
- [ ] Correlation between features was checked and addressed

**Feature Engineering & Modeling Prep**
- [ ] At least 6 new engineered features created and justified
- [ ] Categorical encoding method chosen and justified per variable type
- [ ] Feature scaling applied and justified
- [ ] Class imbalance in `Is_Foreclosure` analyzed and addressed with a stated method
- [ ] Train/test split performed correctly (no leakage, stratified where needed)

**Deliverables**
- [ ] Cleaned dataset saved as a new CSV
- [ ] Before/after comparison report produced
- [ ] All decisions are documented with brief justifications, not just code
