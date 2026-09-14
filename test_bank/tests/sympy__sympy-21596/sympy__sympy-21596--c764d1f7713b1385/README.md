# test_imageset_real_intersection_excludes_singular_parameter

- **Instance:** `sympy__sympy-21596`
- **Test ID:** `sympy__sympy-21596--c764d1f7713b1385`
- **Status:** Rejected and not used in later generation
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 2

## What it checks

Intersecting a complex-valued integer ImageSet with Reals must both restrict parameters to zero-imaginary-part solutions and exclude solutions where the mapping is non-finite.

## Expected behavior

The intersection with Reals is exactly FiniteSet(-1/2), the image of the only nonsingular zero-imaginary parameter n = -1.

## Test command

`cd /testbed && bin/test sympy/sets/tests/test_imageset_real_singularity.py --no-colors`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
