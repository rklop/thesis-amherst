# test_repeated_kfold_repr_with_write_once_parameter

- **Instance:** `scikit-learn__scikit-learn-14983`
- **Test ID:** `scikit-learn__scikit-learn-14983--de0e9e26350e49f5`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 3

## What it checks

RepeatedKFold must expose forwarded constructor parameters for repr without assigning the same parameter again after base initialization. This preserves support for subclasses that manage a public constructor parameter with a write-once descriptor.

## Expected behavior

Construction succeeds and repr returns "WriteOnceRepeatedKFold(n_repeats=2, n_splits=3, random_state=0)".

## Test command

`cd /testbed && python -m pytest -q sklearn/model_selection/tests/test_split.py::test_repeated_kfold_repr_with_write_once_parameter`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
