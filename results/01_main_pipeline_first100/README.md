# Main four-stage pipeline: first 100 patch pairs

This experiment ran the differentiating-test pipeline on 100 MiniMax/gold patch
pairs and compared its three-bucket classifications with an earlier manual
review.

## Headline results

- 62 instances produced a separating test; 38 produced no valid separation.
- MiniMax rated 55 tests high-signal, 6 low-signal, and 1 ambiguous.
- Pipeline and manual labels matched exactly on 64/100 instances and were
  within one adjacent bucket on 82/100.
- This is agreement with a manual reference, not objective accuracy.
- The cohort is not representative: it contains 17 Astropy and 83 Django cases.

## Files to open

- `REPORT.md`: readable experiment summary and interpretation.
- `combined_results.json`: full metrics and all 100 joined instance records.
- `combined_results_swebench_style.json`: compact counts and grouped instance IDs.
- `instances/`: one readable evidence note for every instance.
- `validation.json`: integrity checks and source hashes.
- `runs/`: complete per-instance four-stage evidence.
- `batch_state.json` and `batch_logs/`: execution bookkeeping and controller logs.

Within a run, start with `SUMMARY.md`. A completed run also has
`selected_test.patch`, `selected_proposal.json`, selected execution logs, and a
MiniMax verdict. A `no_valid_separation` run retains all attempted tests and
failure information.
