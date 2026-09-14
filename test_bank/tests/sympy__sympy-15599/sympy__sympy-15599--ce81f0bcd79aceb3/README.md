# test_mod_with_unconstrained_factor

- **Instance:** `sympy__sympy-15599`
- **Test ID:** `sympy__sympy-15599--ce81f0bcd79aceb3`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 1

## What it checks

Reducing a multiplicative coefficient modulo the divisor is valid here only when the whole dividend is known to be integer. An unconstrained symbolic factor may later be noninteger.

## Expected behavior

The result after substitution must be 3/2, since (3*(1/2)) mod 2 is 3/2. It must not become 1/2 through an integer-only coefficient reduction.

## Test command

`python bin/test sympy/core/tests/test_mod_issue_15599.py --no-colors`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
