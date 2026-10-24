# Holistic Data Preparer — Credit Risk Data Preprocessing

## Project Overview
This project demonstrates an end-to-end data-preprocessing and feature-engineering workflow for a credit-risk/customer dataset using Python, Pandas, NumPy, Matplotlib, Seaborn, and scikit-learn.

The notebook covers data ingestion from multiple file formats, data exploration, missing-value handling, outlier treatment, categorical encoding, numerical binning, feature scaling, transformations, and feature construction.

> **Status:** The notebook has been reviewed statically. The review identified several issues that should be fixed before describing the project as fully validated. The notebook's saved outputs do not contain recorded exceptions, but that does not prove it will run from a clean environment because the input files and execution environment were not provided with the notebook.

## Workflow Covered

1. **Data acquisition**
   - CSV: demographic and financial data
   - JSON: credit-behaviour data
   - SQLite database: customer/target data
   - JSON mock API response: customer-related external indicators

2. **Data understanding**
   - DataFrame shape, `info()`, and `describe()`
   - Missing-value counts
   - Dataset profiling with `ydata-profiling`

3. **Missing-value handling**
   - Mean imputation
   - Most-frequent imputation
   - Missingness indicators and random-sample imputation
   - K-nearest-neighbours (KNN) imputation
   - Iterative imputation (MICE-style)

4. **Outlier handling**
   - Box plots
   - Z-score filtering
   - IQR filtering
   - Percentile filtering
   - IQR-based clipping (winsorization-style treatment)

5. **Feature engineering and encoding**
   - Date-part extraction (year, month, day, weekday)
   - One-hot encoding
   - Ordinal encoding for education
   - Label encoding for gender
   - Income binning, threshold flags, quantile binning, and K-Means clustering

6. **Feature scaling and transformations**
   - StandardScaler
   - Normalizer
   - MinMaxScaler
   - MaxAbsScaler
   - RobustScaler
   - Log, reciprocal, and square-root transformations
   - Box-Cox and Yeo-Johnson transformations
   - ColumnTransformer pipeline

7. **Feature construction**
   - Debt-to-income ratio
   - Average monthly transactions
   - Spending-to-income-related feature

## Input Files Expected

The notebook currently uses Google Colab-style paths under `/content/`. Place the following files in that directory, or update the paths to match your environment:

| File | Expected use |
|---|---|
| `01_demographics_financial.csv` | Demographic and financial features |
| `02_credit_behavior.json` | Credit-behaviour features |
| `03_customer_target.sqlitedb` | SQLite database containing at least one table |
| `04_mock_api_response.json` | JSON response with a `records` field |

The notebook expects relevant columns such as `customer_id`, `age`, `annual_income`, `loan_amount`, `gender`, `employment_type`, `education_level`, `region`, `join_date`, `credit_score`, `repayment_history`, `transaction_count`, `loan_purpose`, and `spending_ratio`. Actual column names and formats must match the supplied data.

## Requirements

Suggested packages:

```bash
pip install pandas numpy scikit-learn matplotlib seaborn ydata-profiling openpyxl
```

## How to Run

1. Open the notebook in Google Colab or Jupyter.
2. Upload the four input files and update their paths if needed.
3. Install the listed dependencies.
4. Run the notebook from top to bottom in a fresh kernel.
5. Inspect the output of each step and confirm that joins, transformations, and imputations preserve the expected row counts and columns.
6. Save any final processed dataset explicitly after the pipeline is complete.

## Important Review Findings

The following items need attention before the project can be considered complete:

1. **Incorrect condition in the loan-purpose encoding cell:** the cell checks whether `region` exists in `df_cbd`, but then attempts to encode `loan_purpose`. The condition should check `loan_purpose` in `df_cbd.columns`. Also, an earlier cell one-hot-encodes all object/category columns in `df_cbd`, which may already encode `loan_purpose`; avoid encoding it twice.
2. **Missing indicators are created too late for some columns:** `age`, `gender`, and `employment_type` are imputed before the missing-indicator cell runs. Their original missingness is therefore no longer represented. Create indicators before imputing those columns if retaining missingness information is intended.
3. **The same output filename is used twice:** both the demographics and API KNN-imputation cells write `knn_imputed_dataset.csv`, so the second export overwrites the first. Use distinct filenames.
4. **The final Excel file is only read, not created:** the last cell tries to load `cleaned_transformed_datasets (1).xlsx`, but no earlier cell writes that file. This will fail unless the file already exists. Export the intended final dataset before reading it, or remove the read cell.
5. **The SQL step reads a SQLite database, not a `.sql` script:** the input must be a valid SQLite database file with at least one table. The current code selects the first table, which may not be the intended table if several tables exist.
6. **Potential row duplication during the date merge:** verify that `df_api['customer_id']` is unique before merging it into `df_z`; duplicate IDs can multiply rows.
7. **Transformation assumptions:** Box-Cox requires strictly positive values. Reciprocal transformation is problematic for zero values and can be unstable near zero. Validate values before applying these transformations.
8. **Binning assumptions:** `pd.qcut` can fail or produce fewer groups when many values are tied. The `duplicates='drop'` setting may result in fewer bins than the four labels requested, depending on the data.
9. **Preprocessing leakage risk:** scalers, imputers, binning, and transformations are fitted to the whole dataset. For a predictive machine-learning project, split training/test data first and fit preprocessing only on training data (preferably using a scikit-learn Pipeline).
10. **Target and final dataset are not integrated into one defined pipeline:** `df_ct` is loaded and explored, but the notebook does not clearly join it to the feature data or define a final modelling table. Confirm the intended target and join keys.
11. **Spending-to-income ratio is not actually calculated:** `df_api['spending_to_income_ratio']` is assigned directly from `spending_ratio`. If `spending_ratio` is already a ratio, rename/document it; otherwise calculate it using the appropriate spending and income columns.
12. **Final export is missing:** after the final feature-construction steps, explicitly save the desired output, for example with `df_cd_fd.to_csv('cleaned_transformed_datasets.csv', index=False)`. If data from multiple sources is required, merge them intentionally first.
13. **Label encoding for gender:** `LabelEncoder` is generally designed for target labels. For a binary input feature, mapping the two categories explicitly or using `OneHotEncoder` is often clearer. The notebook also keeps both `gender` and `gender_lebel`, which may duplicate information.
14. **Normalization is row-wise:** `Normalizer` scales each row vector, not each column independently. This may be appropriate for some models, but it is different from column-wise feature scaling.

## Recommended Next Steps

- Correct the encoding condition and avoid duplicate encoding.
- Create missing indicators before imputation.
- Validate required columns and unique customer IDs before transformations and merges.
- Decide which dataset is the modelling table and join the target table deliberately.
- Split train/test data before fitting preprocessing steps if the goal is machine learning.
- Add a final export cell and verify that it creates the output file.
- Restart the kernel and run every cell from top to bottom to test reproducibility.

**
AUTHOR 
PATHAN SAHIL**
