# Differentiating-test pipeline vs. manual review

## Result

The pipeline exactly matched the manual three-bucket review on **64/100 instances (64.0%)**. It was within one adjacent bucket on **82/100 (82.0%)**. Cohen's kappa was **0.429**; quadratic-weighted kappa was **0.519**.

This is agreement with a manual reference, not objective ground-truth accuracy.

## Confusion matrix

Rows are the manual reference. Columns are the pipeline-derived result.

| Manual \ Pipeline | Very different | Somewhat different | Not different |
|---|---:|---:|---:|
| Very different | 24 | 0 | 1 |
| Somewhat different | 14 | 3 | 0 |
| Not different | 17 | 4 | 37 |

## Pipeline outcomes

- Completed separating tests: **62**
- No valid separation found: **38**
- MiniMax high-signal: **55**
- MiniMax low-signal: **6**
- MiniMax ambiguous: **1**
- Severe two-bucket disagreements: **18**
- Severe cases: `astropy__astropy-13033`, `astropy__astropy-13579`, `astropy__astropy-14369`, `astropy__astropy-14995`, `django__django-11095`, `django__django-11099`, `django__django-11206`, `django__django-11276`, `django__django-11400`, `django__django-11477`, `django__django-11532`, `django__django-11790`, `django__django-12039`, `django__django-12125`, `django__django-12143`, `django__django-12406`, `django__django-13112`, `django__django-13343`

## How results were mapped

- A completed separation rated `high_signal` maps to **Very different**.
- A completed separation rated `low_signal` or `ambiguous` maps to **Somewhat different**.
- `no_valid_separation` provisionally maps to **Not different**.

The final mapping is asymmetric: finding a valid pass/fail test is concrete behavioral evidence, but failing to find one after three attempts does not prove that the patches are equivalent.

## What the largest disagreements mean

The raw 64% agreement score does not make the manual review ground truth. Reviewing all **18** two-bucket disagreements found:

- **10** strong new behavioral distinctions that the prior manual review likely understated.
- **4** real or plausible differences whose relevance depends on task scope.
- **3** differentiators using questionable or out-of-domain inputs.
- **1** likely pipeline search miss.

This suggests the pipeline is sensitive to behavioral differences—recovering **24/25** manual “Very different” cases—but it tends to promote subtle or novel edge cases into “Very different.” See each severe case in combined_results.json and its matching note in instances/.

## Files

- `combined_results.json`: all metrics and all 100 joined instance records.
- `combined_results_swebench_style.json`: flat counts, grouped ID lists, and a compact record for every instance.
- `instances/`: one concise evidence note per instance.
- `validation.json`: integrity checks and source hashes.
- `runs/`: exactly one retained raw pipeline run per instance.
- `batch_state.json`: authoritative batch state and run-directory mapping.

## Sampling limitation

This is not a representative random sample: it contains 17 Astropy and 83 Django instances and no other repositories.
