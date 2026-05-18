# ICD-10 Automated Codification

This project explores automatic ICD-10 codification for short clinical
literals in Spanish and Catalan. Given a medical literal such as a diagnosis,
procedure, symptom, abbreviation, or noisy clinical phrase, the goal is to
predict the most appropriate ICD-10 / CIE-10-ES code or category for the ASHO
UAB NLP project and Kaggle-style leaderboard task.

The main project idea is to develop several complementary models in parallel
and then combine their outputs. Each model is expected to be good at a
different part of the problem: exact repeats, typo variants, rare codes,
common categories, semantic matches, or noisy multilingual text. The final
system should use the predictions and confidence scores from all models to
select the most reliable output.

## Project Motivation

ICD codes standardize clinical information so it can be used for reporting,
health management, payment/claims processing, and analysis of disease trends.
Manual clinical codification is time-consuming and error-prone, especially
when the input is short, noisy, ambiguous, or uses local abbreviations.

This project focuses on the harder short-literal setting: instead of coding a
full medical report, the model receives a brief literal and must infer the
correct ICD information.

## Task Summary

- Input: one short clinical literal from the leaderboard/test data.
- Training signal: literal-code pairs from `codification_data.csv`.
- Reference knowledge: official ICD code descriptions and diagnosis/procedure
  flags from `icd_d_p_pairs.csv`.
- Output: the course baseline defines `y_category` as the first character of
  the code; the broader pipeline keeps full-code candidates as well so the
  ensemble can be extended beyond category prediction.
- Evaluation: Kaggle leaderboard, with a public/private split in the hidden
  leaderboard data.

## Key Challenges

- Long-tail labels: the training data contains many ICD codes with very few
  examples.
- Noisy literals: literals include abbreviations, typos, mixed Spanish/Catalan
  forms, uppercase/lowercase variation, and incomplete phrases.
- Ambiguity: some literals can map to more than one clinically plausible code.
- Mixed structures: diagnosis and procedure codes have different formats and
  should not always be handled by the same assumptions.
- Rare and unseen codes: many leaderboard literals may not be exact repeats of
  the training set.

## Data

| File | Rows | Description |
| --- | ---: | --- |
| `data/raw/codification_data.csv` | 13,700 | Training literals with known `Code` labels. |
| `data/raw/leaderboard_data.csv` | 6,667 | Leaderboard/test literals with `id` and `Literal`. |
| `data/raw/icd_d_p_pairs.csv` | 179,742 | ICD reference table with `Code`, `D_P`, and `Description`. |
| `data/processed/processed_all_codes_icd10_and_flagged_icd9.csv` | 13,700 | Training data enriched with cleaned codes and ICD-reference flags. |
| `data/processed/processed_icd10_reference_valid_only.csv` | 9,943 | Subset matched to the ICD reference, enriched with descriptions and target helpers. |

The raw training set contains 4,059 unique codes. In the processed valid ICD-10
subset, 7,932 rows are diagnosis codes and 2,011 rows are procedure codes.

## Repository Structure

```text
data/
  raw/          Original competition and reference CSV files.
  processed/    Cleaned/enriched training data used for modelling.
info/           Local task slides, baseline notes, and ICD-coding background.
presentations/  Follow-up project presentations and pipeline summaries.
```

## Proposed Pipeline

The planned system is a multi-strategy cascade and ensemble.

1. Preprocessing

   Build shared cleaning utilities while allowing model-specific text views.
   For example, transformer models can keep accents and punctuation, while
   matching and retrieval models can use normalized text, accent folding,
   abbreviation expansion, typo handling, and whitespace cleanup.

2. Exact and Fuzzy Matching

   Use direct lookup for literals seen in the training data. For near-matches,
   use edit distance or token/character similarity to catch spelling variants,
   small typos, and formatting differences. This strategy should receive high
   confidence when the match is exact and unambiguous.

3. Semantic Retrieval

   Compare input literals against official ICD descriptions using retrieval
   methods such as character n-gram TF-IDF, BM25, or embeddings. This branch is
   useful for rare codes and literals that do not appear in training.

4. Traditional ML Classification

   Train classifiers such as TF-IDF plus logistic regression or SVM. These
   models are strong baselines for frequent labels/categories and can be
   trained quickly for many experiments.

5. Deep Learning Classification

   Fine-tune a Spanish biomedical transformer such as
   `PlanTL-GOB-ES/roberta-base-biomedical-clinical-es`. The provided baseline
   frames this as single-label multiclass classification over `y_category`,
   using the first character of each ICD code as the target.

6. Ensemble Decision Layer

   Combine all model outputs into a common candidate format and select the
   final answer with weighted voting, confidence calibration, and rule-based
   overrides. Exact unambiguous matches should usually dominate; ambiguous
   cases should be resolved using agreement between retrieval, classifier, and
   transformer branches.

## Model Output Contract

To make parallel model development easy, each model should export predictions
with a shared schema:

```text
id,Literal,model_name,predicted_code,predicted_category,confidence,top_k_codes
```

The ensemble can then aggregate by `id`, compare the predicted code/category
from each branch, and apply confidence-weighted voting or fallback rules.

## Baselines and Evaluation Ideas

Useful metrics to track during development:

- `y_category` accuracy for the current course baseline.
- Full-code accuracy when full-code predictions are available.
- Top-k accuracy for retrieval and candidate-generation models.
- Separate scores for exact-match literals, fuzzy matches, rare codes, and
  diagnosis vs. procedure codes.
- Leaderboard score on the Kaggle public/private split.

The deep learning baseline in `info/` reports validation accuracy around
0.56-0.57 for first-character category prediction using a biomedical Spanish
RoBERTa model with CLS or mean pooling.

## Reference Material

- `presentations/first_followup_presentation.pdf`: project challenge analysis
  and proposed multi-strategy cascade.
- `info/asho-uab-codification.pdf`: ASHO/UAB task context and competition
  framing.
- `info/NLP-I Task_ ICD-10 Codification DL Baselines.pdf`: deep learning
  baseline formulation and transformer setup.
- `info/survey_icd_coding.pdf`: background survey on automated ICD coding.

## Current Status

This repository currently contains the data, processed data, reference
documents, and project presentation material. The next implementation step is
to add model scripts/notebooks for the parallel branches and an ensemble module
that reads their shared prediction files.
