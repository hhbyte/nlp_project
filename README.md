# ICD-10 Chapter Codification

This repository contains the final submission for the Fundamentals of NLP
project on automatic ICD-10 chapter codification of short Spanish/Catalan
clinical literals.

The main result is a probability blend between a strong classical TF-IDF
ensemble and a fine-tuned Spanish biomedical-clinical transformer. The best
public Kaggle score obtained by the final system was **0.595**.

## Main Entry Points

Start here:

- `report/final_report/fnlp_project_report.pdf` - final written report.
- `best_submission/best_submission_pipeline.ipynb` - best reproducible pipeline.
- `best_submission/output/submissions/codi_a20.csv` - archived best submission.

The exploratory work is preserved in `archive/`, but the recommended code path
for teachers is `best_submission/`.

## Repository Structure

```text
.
├── README.md
├── requirements.txt
├── report/
│   ├── README.md
│   └── final_report/
│       ├── fnlp_project_report.pdf
│       ├── fnlp_project_report.tex
│       └── utils/
├── best_submission/
│   ├── README.md
│   ├── best_submission_pipeline.ipynb
│   └── output/
│       └── submissions/
│           └── codi_a20.csv
├── data/
│   ├── README.md
│   ├── raw/
│   ├── codiesp/
│   └── processed/
├── archive/
│   ├── README.md
│   ├── daniel/
│   ├── hermes/
│   └── shanthosh/
├── presentations/
└── information_files_from_campus_virtual/
```

## Best Pipeline Summary

The final pipeline:

1. Cleans and normalises the literals.
2. Trains a classical ensemble using character and word TF-IDF features.
3. Fine-tunes `PlanTL-GOB-ES/roberta-base-biomedical-clinical-es`.
4. Adds external CodiEsp ICD-coded examples for the transformer stage.
5. Blends classical and transformer probabilities.
6. Writes Kaggle-ready submissions to `best_submission/output/submissions/`.

The notebook reads shared data from the repository-level `data/` folder.

## How to Run

Install the Python dependencies:

```powershell
pip install -r requirements.txt
```

Open:

```text
best_submission/best_submission_pipeline.ipynb
```

Run it from inside the `best_submission/` folder. The notebook expects the
shared data at:

```text
../data/raw/
../data/codiesp/
```

The transformer stage is intended for a GPU runtime such as Google Colab or
Kaggle. The classical section can run locally.

## Report

The final report is self-contained in `report/final_report/`. The PDF is ready
to read, and the `.tex` file can be recompiled using the template utilities in
`report/final_report/utils/`.

## Archive

`archive/` contains the development notebooks, intermediate outputs, and
teammate experiment folders. It is included for transparency and provenance,
not as the main reproduction path.
