# test_nthroot_mod_zero_residue_respects_all_roots

- **Instance:** `sympy__sympy-18199`
- **Test ID:** `sympy__sympy-18199--075dccf91977edd4`
- **Status:** Rejected and not used in later generation
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 1

## What it checks

When a is divisible by a composite modulus and the zero-residue equation has multiple roots, all_roots=False should still return only the smallest representative, while all_roots=True returns the complete root set.

## Expected behavior

nthroot_mod(8, 3, 8, all_roots=True) returns [0, 2, 4, 6], while the default single-root mode returns [0] on the composite-modulus path.

## Test command

`cd /testbed && python bin/test sympy/ntheory/tests/test_nthroot_mod_zero.py`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
