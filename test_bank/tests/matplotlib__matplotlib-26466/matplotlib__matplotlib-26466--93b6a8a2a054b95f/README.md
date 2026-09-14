# test_annotation_arraylike_xy_snapshot

- **Instance:** `matplotlib__matplotlib-26466`
- **Test ID:** `matplotlib__matplotlib-26466--93b6a8a2a054b95f`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 1

## What it checks

Annotation coordinates that are NumPy-coercible two-value objects must be normalized into renderable numeric coordinates while being snapshotted. Calling dispatchable `np.copy` can preserve a duck-array wrapper that is not itself iterable, even though it validly supplies coordinates through `__array__`.

## Expected behavior

The annotation renders without error and is pixel-identical to an annotation created from the immutable original pair `(0.25, 0.25)`; the arrow remains horizontal after the external mutation.

## Test command

`python -m pytest -q lib/matplotlib/tests/test_text.py::test_annotation_arraylike_xy_snapshot`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
