# Four-round differentiating pipeline results

This repository contains the result evidence from the 100-instance run of the multi-round core differentiating pipeline. It is **not** the older one-shot comparison pipeline and does **not** include the separate holdout-test experiment.

## What the pipeline did

1. **Round 1 — seed evidence.** Imported initial candidate pairs and prior separating-test evidence, then admitted reusable positive and negative constraints.
2. **Round 2 — regenerate and differentiate.** Generated patches against the accumulated suite, independently validated them, searched both candidate directions for behavioral separating tests, and checked those tests against the actual gold patch.
3. **Round 3 — repeat with feedback.** Repeated generation and differentiation using the stronger suite produced by earlier accepted tests.
4. **Round 4 — final attempt.** Ran final candidate generation and bidirectional search without another feedback round.

All 100 instances reached a terminal category and all four rounds completed. The root run status is `infrastructure_failed` because final bank validation was not performed; it does **not** mean the four rounds failed to run. See [`run_summary.json`](run_summary.json) for the exact recorded status.

## Final terminal categories

| Category | Instances | Meaning |
|---|---:|---|
| `identical` | 43 | The retained candidates were effectively identical under the pipeline's identity rule. |
| `saturated` | 14 | Accepted differentiation evidence reached the stopping rule. |
| `round_limit` | 20 | Still active when Round 4 ended. |
| `zero_passing_patches` | 10 | No candidate passed the required current suite. |
| `other_pair_generation_exhausted` | 7 | The generator did not yield another usable distinct pair. |
| `gold_gate_execution_failure` | 4 | A gold-check execution failed at the infrastructure layer. |
| `candidate_generation_technical_failure` | 2 | Candidate generation ended in a technical failure. |

These labels describe pipeline stopping conditions; they are not a simple correct/incorrect score.

## How to browse

- [`COMBINED_RESULTS.json`](COMBINED_RESULTS.json) is the simplest single-file view: all 100 instances grouped by their final stopping condition, with links to every round in which each instance appears.
- [`rounds/round_1/`](rounds/round_1/) through [`rounds/round_4/`](rounds/round_4/) are the four actual pipeline rounds.
- Each round has a concise `README.md`, a `summary/` folder, and `instances/<instance_id>/` folders.
- Each instance folder combines that round's candidate generation, evaluator reports, separating-test attempts, and gate evidence where available.
- [`constraints/`](constraints/) contains the accumulated accepted test bank.
- [`provenance/completed.json`](provenance/completed.json) records the final per-instance terminal decision.
- [`PUBLICATION_MANIFEST.json`](PUBLICATION_MANIFEST.json) states exactly what was retained and excluded.

Disposable cloned repositories (`workspace/`, `testrepo/`, and nested `.git/`) were excluded because they are reproducibility scratch space, not experimental results. Patches, prompts, model trajectories, proposed tests, execution logs, evaluator reports, decisions, and summaries were retained.
