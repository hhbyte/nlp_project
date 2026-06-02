# Notebook Summary

This project is split into six notebooks to separate data preparation, experiments, analysis, and final submission generation. Each notebook has a clear role in the workflow.

---

## 01_data_preparation.ipynb

### Purpose
This notebook prepares the raw datasets for all later modelling and experiments.

### What it does
- Loads the original codification dataset
- Loads the ICD reference dataset
- Cleans ICD codes
- Normalises medical literals
- Creates `Literal_match`
- Separates ICD-10 rows from flagged ICD-9 rows
- Creates broad category labels (`y_category`)

### Data Used
**Inputs**
```text
codification_data.csv
icd_d_p_pairs.csv
```

**Outputs**
```text
processed_all_codes_icd10_and_flagged_icd9.csv
processed_icd10_reference_valid_only.csv
```

### Why it matters
This notebook standardises all text and creates the cleaned datasets used throughout the project.

---

## 02_retrieval_experiments.ipynb

### Purpose
This notebook tests retrieval-based approaches to see whether similar medical literals can be used to predict categories without relying entirely on supervised learning.

### What it does
- Embedding retrieval
- Literal-to-literal retrieval
- Literal-to-description retrieval
- TF-IDF retrieval
- BM25 retrieval
- KNN retrieval
- Top-k voting
- Weighted top-k voting
- Category centroid retrieval
- Retrieval + SVC hybrid experiments

### Data Used
**Inputs**
```text
processed_icd10_reference_valid_only.csv
```

**Outputs**
```text
retrieval_experiment_summary.csv
retrieval_validation_predictions.csv
```

### Main finding
Retrieval methods worked reasonably for repeated literals but struggled with short abbreviations and unseen terms. The LinearSVC classifier consistently outperformed retrieval-only approaches.

---

## 03_model_experiments.ipynb

### Purpose
This notebook compares supervised machine learning approaches and identifies the strongest classifier.

### What it does
- Logistic Regression
- Complement Naive Bayes
- SGDClassifier
- LinearSVC full-code prediction
- LinearSVC category prediction
- Character TF-IDF SVC
- Character + word TF-IDF SVC
- Tuned LinearSVC hyperparameter search
- Class-weighted SVC
- SGD + SVC ensemble
- High-purity exact matching
- Abbreviation expansion
- Broad context flag model
- Selective context modelling

### Data Used
**Inputs**
```text
processed_icd10_reference_valid_only.csv
```

**Outputs**
```text
model_experiment_summary.csv
validation_predictions_main_models.csv
linear_svc_tuning_results.csv
```

### Main finding
The best core model was a **char + word TF-IDF LinearSVC**, which handled spelling variation, abbreviations, and noisy medical literals better than other models.

---

## 04_error_analysis.ipynb

### Purpose
This notebook investigates **why predictions fail** rather than simply measuring accuracy.

### What it does
- Confusion matrix analysis
- Root-cause error categorisation
- Unseen literal analysis
- Ambiguous literal analysis
- Short abbreviation analysis
- Code-like literal analysis
- O/Z overprediction analysis
- Literal purity analysis

### Data Used
**Inputs**
```text
processed_icd10_reference_valid_only.csv
validation_predictions_main_models.csv
```

**Outputs**
```text
error_cause_summary.csv
confusion_pairs.csv
error_analysis_full.csv
```

### Main finding
Many remaining errors were caused by:
- unseen literals
- ambiguous repeated literals
- missing medical context
- inconsistent labels in training data

rather than purely weak modelling.

---

## 05_icd9_gem_experiments.ipynb

### Purpose
This notebook investigates whether **flagged ICD-9 rows** can improve predictions for unseen literals.

### What it does
- Finds overlap between test literals and ICD-9 literals
- Broad ICD-9 range mapping
- Official GEM ICD-9 → ICD-10 mapping
- Exact GEM overrides
- Weak ICD-9 training
- Safe GEM literal mapping
- Submission disagreement analysis

### Data Used
**Inputs**
```text
processed_all_codes_icd10_and_flagged_icd9.csv
leaderboard_data.csv
2018_I9gem.txt
```

**Outputs**
```text
test_literals_found_only_in_flagged_icd9.csv
flagged_icd9_rows_matching_test.csv
safe_gem_literal_map.csv
submission_change_summary.csv
submission_disagreement_rows.csv
```

### Main finding
Broad ICD-9 usage introduced noise, but carefully constrained **safe GEM overrides** helped on a small number of unseen literals.

---

## 06_final_submission.ipynb

### Purpose
This notebook contains the **clean final Kaggle pipeline**.

### What it does
1. Loads processed ICD-10 data
2. Loads leaderboard/test data
3. Loads safe GEM literal map
4. Normalises literals
5. Trains final char + word TF-IDF LinearSVC
6. Predicts categories
7. Applies selective context override
8. Applies safe GEM override
9. Generates final submission

### Data Used
**Inputs**
```text
processed_icd10_reference_valid_only.csv
leaderboard_data.csv
safe_gem_literal_map.csv
```

**Outputs**
```text
final_submission.csv
```

### Main finding
The strongest final system was:

```text
char + word TF-IDF LinearSVC
+ selective context override
+ constrained safe GEM post-processing
```

This provided the best balance between generalisation and targeted correction.

---

# Recommended Notebook Order

For full reproduction:

```text
01_data_preparation.ipynb
02_retrieval_experiments.ipynb
03_model_experiments.ipynb
04_error_analysis.ipynb
05_icd9_gem_experiments.ipynb
06_final_submission.ipynb
```

For only generating the final Kaggle submission:

```text
01_data_preparation.ipynb
05_icd9_gem_experiments.ipynb
06_final_submission.ipynb
```
