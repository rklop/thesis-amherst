# test_hermite_normal_exhausted_columns

- **Instance:** `sympy__sympy-23413`
- **Test ID:** `sympy__sympy-23413--8dd1fbd6bb353be3`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 3

## What it checks

For a tall, full-column-rank integer matrix, HNF must preserve every input row and stop pivot traversal once every available column has a pivot.

## Expected behavior

The call returns Matrix([[12, 0], [8, 0], [0, 1]]) without the pivot-column cursor becoming negative.

## Test command

`cd /testbed && python -c "from sympy.matrices.tests.test_normalforms import test_hermite_normal_exhausted_columns as t; t()"`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
