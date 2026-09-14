# test_repeated_kfold_invalid_repeats_do_not_set_n_splits

- **Instance:** `scikit-learn__scikit-learn-14983`
- **Test ID:** `scikit-learn__scikit-learn-14983--bf19a3fe2a76abc5`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 1

## What it checks

RepeatedKFold must validate n_repeats before publishing constructor parameters. When initialization is rejected, n_splits should not remain as partially initialized public state.

## Expected behavior

Initialization raises ValueError, and the rejected instance has no n_splits attribute.

## Test command

`python -m pytest -q sklearn/model_selection/tests/test_split.py::test_repeated_kfold_invalid_repeats_do_not_set_n_splits`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
