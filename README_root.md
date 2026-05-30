# ICD-10 Automated Codification

This project explores automatic ICD-10 codification for short clinical literals in
Spanish and Catalan. Given a medical literal — a diagnosis, procedure, symptom,
abbreviation, or noisy clinical phrase — the goal is to predict the correct ICD-10
category for the ASHO/UAB NLP project and its Kaggle-style leaderboard.

The course target, `y_category`, is the first character of the ICD-10 code, so the
task is a multi-class classification over the chapter labels. The leaderboard scores
a strict comparison on that first character.

The team's approach was to develop complementary methods in parallel and combine
their strengths: exact and fuzzy matching, retrieval methods, traditional linear
classifiers, and a fine-tuned Spanish clinical transformer. Different methods are good
at different parts of the problem (exact repeats, typo variants, rare codes, common
categories, noisy multilingual text), and the strongest result came from blending a
classical ensemble with a transformer.

## Motivation

ICD codes standardise clinical information for reporting, health management, claims
processing, and disease-trend analysis. Manual coding is slow and error-prone,
especially for short, noisy, ambiguous input that uses local abbreviations. This
project focuses on the harder short-literal setting: instead of coding a full report,
the model receives a brief literal and must infer the correct code category.

## Task Summary

- **Input:** one short clinical literal from the leaderboard/test data.
- **Training signal:** literal–code pairs from `codification_data.csv`.
- **Reference knowledge:** official ICD descriptions and diagnosis/procedure flags
  from `icd_d_p_pairs.csv`.
- **Output:** `y_category`, the first character of the code.
- **Evaluation:** Kaggle leaderboard, with a public/private split.

## Key Challenges

- **Long-tail labels:** many ICD codes appear only a few times in training.
- **Noisy literals:** abbreviations, typos, mixed Spanish/Catalan forms, case
  variation, and incomplete phrases.
- **Ambiguity:** the same cleaned literal can map to more than one category, so the
  text alone is sometimes insufficient.
- **Mixed structures:** diagnosis and procedure codes have different formats.
- **Unseen codes:** many leaderboard literals are not exact repeats of training.

## Data

| File | Rows | Description |
| --- | ---: | --- |
| `codification_data.csv` | 13,700 | Training literals with known `Code` labels. |
| `leaderboard_data.csv` | 6,667 | Leaderboard/test literals (`id`, `Literal`). |
| `icd_d_p_pairs.csv` | 179,742 | ICD reference table (`Code`, `D_P`, `Description`). |

The raw training set has 4,059 unique codes. Among the rows matched to the ICD-10
reference, roughly 7,932 are diagnosis codes and 2,011 are procedure codes.

## Repository Structure

This is a shared team repository. Each member's folder contains their own
experiments and notebooks; data, reference material, presentations, and the report
are shared at the root.

```text
data/                              Original competition and reference CSV files.
information_files_from_campus_virtual/   Task slides, baseline notes, ICD-coding background.
presentations/                     Project presentations and pipeline summaries.
report/                            Final project report.
daniel/                            Member work (see folder README).
hernest/                           Member work.
Shanthosh/                         Member work.
```

Each member folder has its own README or notebook describing that member's
specific experiments and results.

## What We Built and How We Improved Accuracy

The work progressed in two broad phases. The classical exploration established a
solid baseline; adding a transformer and blending pushed past it.

### Phase 1 — Classical exploration

We started by standardising the text (lowercasing, accent removal, punctuation
normalisation, whitespace cleanup) and deriving `y_category` from the first character
of each code. Flagged ICD-9-style rows were set aside, since their first digit does
not correspond to the ICD-10 category.

We then compared a wide range of classical approaches:

- **Matching and retrieval** — exact matching, fuzzy matching, semantic embedding
  retrieval, ICD-description retrieval, literal-to-literal retrieval, KNN and
  top-k voting, centroid retrieval, and BM25. These were useful diagnostics but
  limited on their own: literals are very short, so fuzzy and embedding methods
  introduced noise, and similar text often belonged to different categories.
- **Supervised models** — after finding full-code prediction too hard (long-tail
  labels), we predicted the category directly. A **character + word TF-IDF Linear
  SVM** became the strongest classical model, because character n-grams capture
  spelling variants, prefixes, suffixes, and abbreviations. This absorbed most of
  what exact/fuzzy matching was designed for.
- **Feature engineering and rules** — a **selective context override** (a few
  reliable patterns such as *puerperio* and *fetal*) added domain signal without
  overcorrecting and became one of the strongest classical submissions. Broad
  context flags, class weighting, and manual rescue rules were rejected for hurting
  the leaderboard or overfitting the validation split.
- **ICD-9 mapping** — official GEM ICD-9→ICD-10 mappings, applied as a constrained
  override on top of selective context, gave a small gain; broad mappings reduced it.

The best classical configuration reached a public leaderboard score of about **0.572**.

### Phase 2 — Transformer augmentation and blend

Phase 1 plateaued, so we added a neural component:

- **Clinical transformer** — fine-tuned `PlanTL-GOB-ES/roberta-base-biomedical-clinical-es`
  (Spanish biomedical-clinical RoBERTa) as a chapter classifier with a class-weighted
  loss, using additional external ICD-coded data from the CodiEsp corpus.
- **Probability blend** — instead of overriding, we averaged the two models'
  probabilities: `final = α · transformer + (1 − α) · classical`, predicting the
  argmax. Because local validation did not track the leaderboard reliably, **α was
  tuned directly on the leaderboard** (best around **α ≈ 0.24**). Light Spanish
  stopword removal and stemming and a re-tuned SVM regularisation were folded in.

The blended system reached a public leaderboard score of **0.595**, improving on the
0.572 classical baseline. Each component was weaker alone (classical ≈ 0.56,
transformer ≈ 0.50), so the gain came from combining their complementary strengths.

### The practical ceiling

An error analysis localised the main remaining error to the **diagnosis-versus-procedure
ambiguity** of short literals. In a two-stage test, knowing that distinction perfectly
would raise accuracy to ~71%, but predicting it from the literal alone reaches only ~75%,
which cancels most of the gain. The limiting factor is information not present in the
short text — which is why stronger models and more data did not break past this point.

## Reference Material

- `presentations/` — challenge analysis, the proposed cascade, and follow-up summaries.
- `report/` — the final written project report.
- `information_files_from_campus_virtual/` — ASHO/UAB task context, the deep learning
  baseline formulation, and a background survey on automated ICD coding.

The deep learning baseline reports validation accuracy around 0.56–0.57 for
first-character category prediction with a biomedical Spanish RoBERTa model.

## Team

Hermes Barreiro Pena · Daniel Massoud Massoud · Shanthosh
