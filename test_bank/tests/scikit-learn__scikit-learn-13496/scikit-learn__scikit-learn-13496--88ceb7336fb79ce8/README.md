# test_iforest_docstring_parameter_order

- **Instance:** `scikit-learn__scikit-learn-13496`
- **Test ID:** `scikit-learn__scikit-learn-13496--88ceb7336fb79ce8`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 2

## What it checks

Because IsolationForest accepts positional constructor arguments, its public Parameters documentation must list parameters in the same order as its callable signature. In particular, warm_start belongs between bootstrap and n_jobs, not after verbose.

## Expected behavior

The documented parameter sequence equals the constructor signature sequence, including the local order bootstrap, warm_start, n_jobs. candidate_b satisfies this; candidate_a documents warm_start after verbose and fails.

## Test command

`cd /testbed && python -m pytest -q sklearn/ensemble/tests/test_iforest.py::test_iforest_docstring_parameter_order`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
