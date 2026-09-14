# test_ridge_classifier_cv_store_cv_values_positional

- **Instance:** `scikit-learn__scikit-learn-10297`
- **Test ID:** `scikit-learn__scikit-learn-10297--c4e7fb819a4950e3`
- **Status:** Rejected and not used in later generation
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 1

## What it checks

The new boolean flag must also work through its public positional constructor slot immediately after fit_intercept, not only when passed by keyword. The generated candidate instead interprets the third positional argument as normalize and silently leaves CV-value storage disabled.

## Expected behavior

After fitting, cv_values_ exists and has shape (4, 1, 2): four samples, one binary response, and two alpha values.

## Test command

`cd /testbed && python -m pytest -q sklearn/linear_model/tests/test_ridge.py::test_ridge_classifier_cv_store_cv_values_positional`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
