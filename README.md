# SWE-bench Differentiating-Test Results

This repository is a data-only release of two experiments comparing verified
MiniMax patches with SWE-bench Verified gold patches. It contains aggregate
JSON results, readable summaries, generated tests, model outputs, execution
logs, and MiniMax judgments. Pipeline source code and cloned benchmark
repositories are intentionally excluded.

## Start here

| Experiment | Scope | Best first file |
| --- | --- | --- |
| [Main four-stage pipeline](results/01_main_pipeline_first100/) | 100 patch pairs; compares pipeline classifications with an earlier manual review | [Report](results/01_main_pipeline_first100/REPORT.md) or [complete JSON](results/01_main_pipeline_first100/combined_results.json) |
| [Holdout-test generation](results/02_holdout_generation_random100/) | 100 patch pairs; searches specifically for tests that fail on the candidate and pass on gold | [Holdout test inventory](results/02_holdout_generation_random100/test_quality_inventory.json) |

## How each run is organized

Every directory under an experiment's `runs/` folder represents one SWE-bench
instance. Its four numbered stages are:

1. `00_inputs/`: the candidate patch, gold patch, and SWE-bench instance.
2. `01_codex/`: test-generation prompts, responses, proposals, and test patches.
3. `02_execution/`: candidate and gold execution logs plus machine-readable results.
4. `03_minimax/`: MiniMax's quality-review prompt, response, and final verdict.

Open `SUMMARY.md` or `summary.json` inside an individual run before reading the
stage details. When a separating test was accepted, `selected_test.patch` and
`selected_proposal.json` identify the chosen test.

## Important interpretation

A test is mechanically separating when one patch fails and the other passes.
MiniMax's `high_signal`, `ambiguous`, and `low_signal` labels judge whether that
difference is meaningful for the issue. A high-signal label is model judgment,
not independently established ground truth.

Raw records retain some original `/home/rocky/...` strings as provenance. The
collaborator-facing aggregate indexes use paths relative to their experiment
directory.

## Publication scope

The release includes all result evidence but omits full cloned workspaces,
nested Git repositories, Python caches, controller PID files, and pipeline
implementation scripts. Patch and test files are retained because they are
experiment outputs rather than copies of source repositories.
