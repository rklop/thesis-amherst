# test_imageset_intersect_real_with_mixed_factors

- **Instance:** `sympy__sympy-21596`
- **Test ID:** `sympy__sympy-21596--5233d1f155baa12b`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 2

## What it checks

When a factored imaginary component has solvable linear factors plus a nonlinear factor that is strictly positive on integers, the linear-factor zeros should still determine the real-valued image points.

## Expected behavior

The intersection evaluates to FiniteSet(-1, 1), rather than remaining an unevaluated conditional set.

## Test command

`/opt/miniconda3/envs/testbed/bin/python bin/test sympy/sets/tests/test_fancysets.py -k test_imageset_intersect_real_with_mixed_factors --no-colors`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
