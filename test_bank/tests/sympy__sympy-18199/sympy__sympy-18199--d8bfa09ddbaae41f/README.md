# test_nthroot_mod_composite_default_root

- **Instance:** `sympy__sympy-18199`
- **Test ID:** `sympy__sympy-18199--d8bfa09ddbaae41f`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 2

## What it checks

The documented all_roots=False default should return the smallest root as a scalar even when the modulus is composite; the generated candidate instead forces all_roots=True for every composite modulus.

## Expected behavior

The function returns the scalar integer 1, rather than the one-element list [1].

## Test command

`python -c 'from sympy.ntheory.tests.test_residue import test_nthroot_mod_composite_default_root; test_nthroot_mod_composite_default_root()'`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
