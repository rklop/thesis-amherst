# test_repeated_split_repr_preserves_forwarded_cvargs

- **Instance:** `scikit-learn__scikit-learn-14983`
- **Test ID:** `scikit-learn__scikit-learn-14983--15378e821d88a5ab`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 2

## What it checks

A repeated splitter's repr must preserve every explicit constructor parameter forwarded to its wrapped cross-validator, not only `n_splits`.

## Expected behavior

`repr(splitter)` returns exactly `RepeatedShuffleSplit(n_repeats=3, random_state=7, test_size=0.25)`.

## Test command

`cd /testbed && python -m pytest -q sklearn/model_selection/tests/test_repeated_split_repr.py`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
