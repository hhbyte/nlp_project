# ICD-10 Chapter Codification 

My approach to the UAB-ASHO AI Codification task: assigning an ICD-10 chapter (the first character of the code) to short Spanish/Catalan medical literals.

The final model is a blend of a classical TF-IDF ensemble and a fine-tuned Spanish clinical transformer, with the blend weight tuned directly on the leaderboard. It scored **0.595** on the public leaderboard, up from the **0.572** baseline we started with.

## What's in this folder

| File | What it is |
|------|------------|
| `npl_latest_pipeline.ipynb` | The full pipeline — classical model, transformer fine-tuning, and the blend. This is the notebook to run. |
| `codification_data.csv` | Training data (literal → code), provided by the competition. |
| `leaderboard_data.csv` | The literals to predict and submit. |
| `icd_d_p_pairs.csv` | Official ICD code + description pairs (used by the transformer data step). |
| `train_annotations_task{1,2,3}_processed.tsv` | CodiEsp external data (train split) — extra Spanish clinical literal→code pairs. |
| `dev_annotations_task{1,2,3}_processed.tsv` | CodiEsp external data (dev split). |

The CodiEsp `.tsv` files come from the CLEF 2020 CodiEsp shared task (BSC), via Zenodo. They're included here so the notebook runs without a separate download.

## How to run it

The transformer step needs a GPU, so this is built for **Google Colab** (or Kaggle). On the free Colab T4 a full run is roughly 10–15 minutes.

1. Open `npl_latest_pipeline.ipynb` in Colab and set the runtime to GPU
   (`Runtime → Change runtime type → T4 GPU`).
2. Upload the data files so they sit next to the notebook. The notebook expects them under `data/raw/`, so either upload them into that path or run the small setup cell at the top that copies them there. The files you need:
   - `codification_data.csv`, `leaderboard_data.csv`, `icd_d_p_pairs.csv`
   - the six `*_annotations_task*_processed.tsv` files (only needed for the CodiEsp step)
3. `Runtime → Run all`.

The notebook writes its submission CSVs to `output/submissions/`. Download the best one and upload it to Kaggle.

### google colab

The first cell installs/upgrades `transformers`, `datasets`, and `pyarrow`. After it runs, **restart the session once** (`Runtime → Restart session`) and then run from the top — otherwise Colab keeps the old `pyarrow` loaded and you get a `pyarrow.lib.IpcReadOptions size changed` error on `from datasets import Dataset`. Restarting after the install fixes it.

## How it works

**Target.** The leaderboard is scored on a strict match of the *first character* of the code, so the whole problem is reduced to predicting that one chapter character per literal.

**Preprocessing.** Lowercase, strip accents, drop a small Spanish stopword list, and apply light suffix stemming. Stemming + stopword removal was worth about +0.4 on local validation.

**Classical ensemble.** TF-IDF on character n-grams (2–5) and word n-grams (1–2), fed into three models — LinearSVC, ComplementNB, and Logistic Regression — whose probabilities are averaged. Character n-grams matter here because the literals are full of abbreviations (`VHC`, `TCE`, `HTA`) and typos.

**Transformer.** `PlanTL-GOB-ES/roberta-base-biomedical-clinical-es` (BSC's Spanish biomedical-clinical RoBERTa) fine-tuned as a chapter classifier, class-weighted to handle the chapter imbalance, and trained on our literals plus the CodiEsp pairs.

**Blend.** `final = α · transformer + (1 − α) · classical`. We didn't trust local validation to pick α (it kept disagreeing with the leaderboard), so we tuned α by submitting a few values. The best was around **α = 0.24**.

## What didn't work

A fair amount, and it's worth recording:

- **Exact / fuzzy match override.** The obvious idea, if a leaderboard literal exactly matches a training literal, just reuse its code — actually *lowered* the score. Around 57% of repeated literals map to more than one chapter, because the same short phrase gets coded differently depending on report context. So we let the classifier decide instead of overriding it.
- **Bigger / multilingual models.** `xlm-roberta-base` did worse than the Spanish biomedical model (0.355 vs 0.502 standalone). Domain knowledge mattered more than multilingual coverage.
- **More external data.** Adding the full 179K reference descriptions to training, and adding CodiEsp, barely moved the blended score (about +0.00). Both slightly *lowered* the transformer's standalone accuracy, the outside text reads differently from the competition's literals.
- **Seed-averaging the transformer.** Averaging three runs tied the single run; the runs were making the same predictions, so there was nothing to average out.
- **Class-weighting toward procedures.** Boosting the numeric (procedure) chapters raised procedure accuracy and dropped diagnosis accuracy by the same amount..

## On the ceiling

The score stalls around 0.595 for a concrete reason. The biggest single source of error is telling **diagnoses from procedures**: in a two-stage test, if the diagnosis/procedure split were known perfectly, accuracy would jump... but predicting it from the short literal text alone only reaches ~75%, and that error rate cancels the gain. The literals are short, noisy, bilingual, and often genuinely ambiguous between a condition and the procedure done about it. That ambiguity isn't recoverable from the text, which is why stronger models and more data don't push past this point.

## Team group 4

Daniel Massoud Massoud · Shanthosh · Hermes Barreiro Pena  
