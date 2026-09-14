# test_imageset_intersect_real_maps_nonidentity_real_part

- **Instance:** `sympy__sympy-21596`
- **Test ID:** `sympy__sympy-21596--a09b7b0073ccf750`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 1

## What it checks

When intersecting an integer ImageSet with the reals, the valid integer indices must be mapped through the expression's real component; that component need not be the identity n.

## Expected behavior

FiniteSet(-2, 2), because mapping the two admissible indices through the real component 2*n produces -2 and 2.

## Test command

`python bin/test sympy/sets/tests/test_fancysets.py -k test_imageset_intersect_real_maps_nonidentity_real_part --no-colors`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
