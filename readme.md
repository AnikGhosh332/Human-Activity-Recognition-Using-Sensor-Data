# Zero-Shot HAR on PAMAP2 — V5

Diagnostic study of text-grounded zero-shot activity recognition from IMU data.
A dilated TCN encoder is trained against Sentence-BERT class prototypes and
evaluated on held-out activity classes under a protocol in which no
hyperparameter is selected on test labels.

**Main finding.** The representation is not the bottleneck. A cross-validated
linear probe on the held-out embeddings reaches a mean macro F1 of 0.95, while
the best zero-shot result on the same embeddings reaches 0.43 — a shortfall of
0.52 (95% CI [0.41, 0.63], p < 0.001) across nine unseen-class splits. The
held-out classes are separable; assigning the right name to each cluster is what
fails.

## Contents

| File | Purpose |
|---|---|
| `zsl_research_combined_V5_IN_USE.ipynb` | Full pipeline: preprocessing, prototype construction, training, three inference approaches, per-run diagnostics and figures |
| `visualisations.ipynb` | Cross-split aggregation; produces the paper tables and the two aggregate figures |
| `experiments_v5/results_store.json` | One record per run, keyed on `(prototype_set, split, seed, protocol_hash)` |
| `experiments_v5/results_*.log` | Human-readable log per run |

## Protocol

Nine subject-disjoint unseen-class splits, seed 42. Each split partitions the 18
PAMAP2 activities into four **unseen** classes (test only), three
**proxy-unseen** classes (excluded from training, used to select inference
hyperparameters), and eleven **seen** classes. Disjointness of the unseen and
proxy-unseen sets is asserted in code.

- Subjects 108 and 109 held out entirely; all splits are class- and subject-disjoint.
- 31 channels, 1000-frame windows (10 s at 100 Hz) for train, validation and test.
  Stride 250 / 500. Purity threshold 0.85. No test-set augmentation.
- Encoder: 3 dilated residual TCN blocks (64/96/128, dilations 1/2/4, kernel 5),
  global average pooling, 128→512→d head, L2-normalised. 0.73 M parameters at d=768.
- Objective: softmax cross-entropy over cosine similarities, τ = 0.07, denominator
  restricted to seen classes.
- AdamW, lr 1e-3, weight decay 1e-4, cosine annealing over 40 epochs, patience 15,
  gradient clip 1.0. Checkpoint on seen-class validation accuracy.

Every setting above is recorded in each result record under `protocol` and
hashed, so runs from different protocols cannot be compared by accident.

## Inference approaches

| | Method | Mode | Macro F1 |
|---|---|---|---|
| A1 | Cosine nearest-prototype | inductive | 0.27 ± 0.17 |
| A2 | Cross-modal centering + Inverted Softmax | transductive | 0.38 ± 0.15 |
| A3 | Ridge semantic-to-sensor map + Inverted Softmax | transductive | 0.39 ± 0.12 |

Paired across splits: A3 − A1 = +0.12 (95% CI [+0.02, +0.22], p = 0.029);
A2 − A1 = +0.11 (p = 0.056); A3 − A2 = +0.01 (p = 0.796). A2 and A3 use the
unlabelled test set collectively and are reported as transductive throughout.

The Inverted Softmax temperature is selected on proxy-unseen classes over a grid
derived from the standard deviation of the relevant similarity matrix. It lands
on a grid endpoint in 7 of 18 selections.

## Prototype spaces

Set `PROTOTYPE_SET` in cell 2 to one of `labels`, `hand`, `llm`, `attributes`.
Separability (mean off-diagonal cosine, lower is better): hand-written 0.45,
label names 0.31, LLM-selected 0.20. Only `llm` has been run end to end.

## Reproducing

1. Set `UNSEEN_IDS` and `VAL_UNSEEN_IDS` in cell 2, keeping them disjoint.
2. Run all. Results are appended to `experiments_v5/results_store.json`.
3. Repeat per split, then run `visualisations.ipynb` for the aggregate tables and
   figures.

## Known issues

- **Single seed.** All runs use seed 42, so reported intervals are across splits
  and quantify generalisation to a different choice of held-out activities, not
  optimisation variance.
- **One excluded split.** `1_4_11_19` is in the store but dropped from all
  analysis: car driving has zero test windows in the held-out subjects, which
  caps macro F1 below 1.0 and leaves the linear probe undefined. The aggregation
  cell drops it automatically and reports why.
- **Non-standard proxy-unseen sets.** Splits `2_12_17_20` and `4_6_7_24` use
  `[7,19,24]` and `[1,5,19]` respectively, forced because the default `[7,17,24]`
  overlaps their held-out classes.
- **A3 hyperparameters.** `K = 9` and `λ = 0.001` are selected on every split.
  With ten alignment pairs a 9-component PCA interpolates exactly, so the
  leave-one-out criterion cannot discriminate. Cap `K` below `n_pairs / 2` before
  drawing conclusions from this choice.
- **No external baseline.** UniMTS export exists in the notebook but has not been
  run.