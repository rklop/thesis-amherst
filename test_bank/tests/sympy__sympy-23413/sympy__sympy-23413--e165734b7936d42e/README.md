# test_hermite_normal_tall_single_column

- **Instance:** `sympy__sympy-23413`
- **Test ID:** `sympy__sympy-23413--e165734b7936d42e`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 2

## What it checks

For a tall one-column matrix, the lowest nonzero entry is the sole pivot and must be positive. Rows above it are not additional pivot rows and must not undo that sign normalization.

## Expected behavior

Matrix([[-2], [1]]). The only unimodular transformation available for one column is a sign change, making the bottom pivot positive. candidate_a reprocesses the upper row and flips the column back, leaving a negative pivot.

## Test command

`python bin/test sympy/matrices/tests/test_normalforms.py --no-colors -k test_hermite_normal_tall_single_column`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
