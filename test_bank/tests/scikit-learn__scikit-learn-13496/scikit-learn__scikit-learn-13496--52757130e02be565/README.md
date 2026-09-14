# test_iforest_warm_start_cooperative_forwarding

- **Instance:** `scikit-learn__scikit-learn-13496`
- **Test ID:** `scikit-learn__scikit-learn-13496--52757130e02be565`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 1

## What it checks

IsolationForest should forward the newly exposed warm_start argument in the corresponding inherited BaseBagging constructor position, immediately before n_jobs.

## Expected behavior

The recorded keywords place warm_start immediately before n_jobs, and the resulting estimator exposes warm_start as True.

## Test command

`cd /testbed && python -m pytest -q sklearn/ensemble/tests/test_iforest.py::test_iforest_warm_start_cooperative_forwarding`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
