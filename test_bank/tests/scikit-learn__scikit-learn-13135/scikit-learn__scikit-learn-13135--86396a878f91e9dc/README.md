# test_kmeans_bin_edges_monotonicity_is_explicit

- **Instance:** `scikit-learn__scikit-learn-13135`
- **Test ID:** `scikit-learn__scikit-learn-13135--86396a878f91e9dc`
- **Status:** Rejected and not used in later generation
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 2

## What it checks

For the problematic high-bin-count case, k-means bin edges must be strictly increasing, and the sorting step must explicitly preserve that invariant. The patches are executable-equivalent; their only differentiator is that candidate_b records this invariant alongside the fix.

## Expected behavior

No exception; fitted bin edges are strictly increasing and transformed ordinal values are nondecreasing. The implementation source explicitly identifies that centers are sorted to ensure monotonically increasing bin edges.

## Test command

`python -m pytest -q sklearn/preprocessing/tests/test_discretization.py::test_kmeans_bin_edges_monotonicity_is_explicit`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
