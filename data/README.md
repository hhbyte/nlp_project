# Data

Shared data for the project. The best submission notebook reads from this folder rather than keeping its own private data copy.

```text
data/
  raw/
    codification_data.csv
    leaderboard_data.csv
    icd_d_p_pairs.csv
  codiesp/
    train_annotations_task*_processed.tsv
    dev_annotations_task*_processed.tsv
  processed/
    processed/intermediate CSVs from earlier experiments
```

`raw/` contains the competition input files and ICD reference table. `codiesp/` contains the external CodiEsp annotation files used by the transformer augmentation stage. `processed/` contains derived files produced during exploration and kept for reproducibility.
