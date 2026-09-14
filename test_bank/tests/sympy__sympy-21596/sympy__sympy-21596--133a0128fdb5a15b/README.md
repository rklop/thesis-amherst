# test_imageset_reals_nonlinear_imaginary_part

- **Instance:** `sympy__sympy-21596`
- **Test ID:** `sympy__sympy-21596--133a0128fdb5a15b`
- **Status:** Rejected and not used in later generation
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 1

## What it checks

Real intersection must solve the complete imaginary-part equation over the ImageSet's integer domain, including nonlinear expressions that are not already products of linear factors.

## Expected behavior

The intersection is exactly FiniteSet(-2, 2), since the only integer parameters making the imaginary part zero are -2 and 2, and the real component equals n.

## Test command

`/opt/miniconda3/envs/testbed/bin/python bin/test sympy/sets/tests/test_fancysets.py -k test_imageset_reals_nonlinear_imaginary_part`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
