# test_hermite_normal_zero_columns

- **Instance:** `sympy__sympy-23413`
- **Test ID:** `sympy__sympy-23413--e89d8891a4186627`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 1

## What it checks

Hermite normal form must handle a ZZ DomainMatrix whose column budget is exhausted before row processing begins. It should not attempt to access a nonexistent column.

## Expected behavior

Return the same 3-by-0 DomainMatrix without raising an exception.

## Test command

`cd /testbed && /opt/miniconda3/envs/testbed/bin/python -c "from sympy.polys.matrices.tests.test_normalforms import test_hermite_normal_zero_columns; test_hermite_normal_zero_columns()"`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
