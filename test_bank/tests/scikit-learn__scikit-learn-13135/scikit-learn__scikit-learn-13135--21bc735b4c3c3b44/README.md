# test_kmeans_sorts_learned_centers_in_place

- **Instance:** `scikit-learn__scikit-learn-13135`
- **Test ID:** `scikit-learn__scikit-learn-13135--21bc735b4c3c3b44`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 1

## What it checks

For strategy='kmeans', ordering must be normalized on the learned center sequence itself, not merely on a temporary copy used to calculate bin edges.

## Expected behavior

fit_transform completes with shape (6, 1), nondecreasing ordinal codes containing exactly [0, 1, 2, 3, 4], and the learned KMeans centers are strictly increasing.

## Test command

`cd /testbed && python -m pytest -q sklearn/preprocessing/tests/test_discretization.py::test_kmeans_sorts_learned_centers_in_place`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
