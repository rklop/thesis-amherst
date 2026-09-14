# test_ridge_classifier_cv_preserves_positional_parameter_order

- **Instance:** `scikit-learn__scikit-learn-10297`
- **Test ID:** `scikit-learn__scikit-learn-10297--bed413bb17f50197`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 1

## What it checks

Adding the optional store_cv_values parameter must preserve RidgeClassifierCV's existing positional constructor order. In particular, the third through sixth positional arguments must remain normalize, scoring, cv, and class_weight; store_cv_values is appended with a default of False.

## Expected behavior

get_params(deep=False) reports the six supplied values under their original parameter names and reports store_cv_values=False.

## Test command

`cd /testbed && python -m pytest -q sklearn/linear_model/tests/test_ridge.py::test_ridge_classifier_cv_preserves_positional_parameter_order`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
