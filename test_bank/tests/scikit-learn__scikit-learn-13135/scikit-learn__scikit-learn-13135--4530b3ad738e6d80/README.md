# test_kmeans_strategy_preserves_kmeans_fitted_state

- **Instance:** `scikit-learn__scikit-learn-13135`
- **Test ID:** `scikit-learn__scikit-learn-13135--4530b3ad738e6d80`
- **Status:** Rejected and not used in later generation
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 1

## What it checks

Sorting the centers for bin-edge construction must not mutate the fitted KMeans model. Its stored training labels must remain consistent with predictions from its fitted centers.

## Expected behavior

After KBinsDiscretizer.fit returns, calling predict on the recorded KMeans instance with the training data produces exactly its stored labels_. candidate_b sorts a copy of the centers and preserves this invariant; candidate_a sorts the cluster_centers_ view in place and changes center-to-label ordering.

## Test command

`python -m pytest -q sklearn/preprocessing/tests/test_discretization.py::test_kmeans_strategy_preserves_kmeans_fitted_state`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
