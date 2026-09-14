# test_nthroot_mod_zero_composite_multiple

- **Instance:** `sympy__sympy-18199`
- **Test ID:** `sympy__sympy-18199--65a819d853ef4583`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 1

## What it checks

The zero-residue case also applies to composite moduli, where several residue classes can be roots. The supplied reference returns the complete sorted root list for this composite case.

## Expected behavior

[0, 2, 4, 6], exactly the even residue classes modulo 8. Each cube is divisible by 8.

## Test command

`python bin/test sympy/ntheory/tests/test_residue.py -k test_nthroot_mod_zero_composite_multiple --no-colors`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
