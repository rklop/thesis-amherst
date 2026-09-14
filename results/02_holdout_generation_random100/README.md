# Holdout-test generation: random 100

This experiment reran the strict differentiating-test pipeline on a random
100-instance cohort. Candidate A was the first officially passing generated
patch and candidate B was the passing SWE-bench Verified gold patch.

A test was accepted only when the generated candidate failed and gold passed.
These accepted tests are evaluation-only holdouts.

## Headline results

- 70 instances produced an accepted candidate-fail/gold-pass test.
- 30 instances produced no valid separator within the configured attempts.
- MiniMax rated the 70 accepted tests as 56 high-signal, 10 low-signal, and
  4 ambiguous.
- All 70 candidates failed when their tests were replayed independently.
- All 70 gold patches passed during generation.

## Files to open

- `test_quality_inventory.json`: best starting point; groups all 70 accepted
  tests by MiniMax rating and links to the test, proposal, verdict, and replay.
- `cohort.json`: all 100 selected instances and selection metadata.
- `candidate_first_passing_100.json`: candidate patches used as candidate A.
- `candidate_replay_results.json`: independent replay result for every accepted test.
- `candidate_replay/`: independent replay logs.
- `runs/`: complete per-instance four-stage evidence, including the 30 cases
  where no separator was accepted.
- `batch_state.json` and `batch_logs/`: execution bookkeeping and controller logs.

Paths in `test_quality_inventory.json` and `candidate_replay_results.json` are
relative to this directory, so collaborators can follow them after cloning.
