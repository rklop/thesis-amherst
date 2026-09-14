# test_repeated_kfold_repr_with_parameter_aware_subclass

- **Instance:** `scikit-learn__scikit-learn-14983`
- **Test ID:** `scikit-learn__scikit-learn-14983--8fbb063b6c192e4b`
- **Status:** Rejected and not used in later generation
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 1

## What it checks

The public `n_splits` constructor parameter should be available when inherited initialization assigns other parameters. This permits a `RepeatedKFold` subclass's `n_repeats` property to validate against `n_splits`, while preserving an accurate repr.

## Expected behavior

Construction succeeds and `repr` returns `LimitedRepeatedKFold(n_repeats=2, n_splits=3, random_state=None)`.

## Test command

`cd /testbed && python -m pytest -q sklearn/model_selection/tests/test_split.py::test_repeated_kfold_repr_with_parameter_aware_subclass`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
