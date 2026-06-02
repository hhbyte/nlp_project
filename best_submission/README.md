# Best Submission Pipeline

This folder contains the best-performing submission pipeline from Daniel's
workspace, cleaned up as the main reproducible code path for the project. It is
organised so the final approach can be understood from start to finish without
digging through the trial notebooks.

The archived best public Kaggle score was **0.595**. The best saved submission is:

```text
output/submissions/codi_a20.csv
```

## Folder Contents

```text
best_submission/
  best_submission_pipeline.ipynb
  output/
    submissions/
      codi_a20.csv
```

The notebook reads shared data from the repository-level `data/` folder:

```text
data/
  raw/
    codification_data.csv
    leaderboard_data.csv
    icd_d_p_pairs.csv
  codiesp/
    train_annotations_task*_processed.tsv
    dev_annotations_task*_processed.tsv
```

## What the Notebook Does

The notebook has two main stages.

First, it builds a strong classical baseline:

- reads the competition training and leaderboard files;
- normalises literals with lowercasing, accent removal, light stopword removal,
  and simple Spanish suffix stemming;
- creates the target as the first character of each ICD-10 code;
- trains a TF-IDF ensemble using character and word n-grams;
- compares the improved ensemble against the earlier single-SVC baseline;
- saves the classical model's probability distribution for the leaderboard rows.

Second, it adds the neural component:

- fine-tunes `PlanTL-GOB-ES/roberta-base-biomedical-clinical-es`, a Spanish
  biomedical-clinical RoBERTa model;
- adds external CodiEsp ICD-coded examples as extra training pairs;
- uses class weighting to reduce the effect of chapter imbalance;
- predicts transformer probabilities for the leaderboard rows;
- blends classical and transformer probabilities:

```text
final = alpha * transformer + (1 - alpha) * classical
```

The submitted file archived here is `codi_a20.csv`, corresponding to
`alpha = 0.20`. Other nearby blend weights were also tested during development.

## How to Run

Open `best_submission_pipeline.ipynb` from inside this folder. For a local run,
start Jupyter with the working directory set to:

```text
best_submission/
```

The notebook expects:

```text
../data/raw/codification_data.csv
../data/raw/leaderboard_data.csv
../data/raw/icd_d_p_pairs.csv
../data/codiesp/*annotations_task*_processed.tsv
```

For Colab, upload the folder or upload the files into the same layout. The
transformer stage should be run with a GPU runtime. If Colab gives a `pyarrow`
binary mismatch after the install cell, restart the runtime once and run from
the top again.

## Why This Was the Best Pipeline

The classical TF-IDF ensemble was strong because the literals are very short and
often contain abbreviations, partial words, spelling variations, and code-like
fragments. Character n-grams capture many of these surface patterns better than
semantic retrieval or fuzzy matching alone.

The transformer was weaker as a standalone model, but it added complementary
signal. Blending at the probability level worked better than hard overrides
because the classical model remained the anchor while the transformer could
shift uncertain cases where it had useful clinical evidence.

The final score still plateaued because many literals are ambiguous without
additional clinical context, especially diagnosis-versus-procedure cases. The
pipeline is therefore best understood as the strongest text-only blend found in
the experiments, not as a complete solution to the missing-context problem.

## Main Output

The archived best submission is:

```text
output/submissions/codi_a20.csv
```

It has the expected Kaggle columns:

```text
id,Literal,y_category
```
