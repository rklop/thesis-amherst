# test_Mod_even_dividend_quotient

- **Instance:** `sympy__sympy-15599`
- **Test ID:** `sympy__sympy-15599--7391bff7037c2fa1`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 1

## What it checks

An even symbolic dividend does not imply that dividing it by 2 produces an even value; Mod must not discard this uncertainty before substitution.

## Expected behavior

The substituted result equals Mod(3, 2), which evaluates to 1. It must not be prematurely simplified to 0.

## Test command

`python bin/test sympy/core/tests/test_arit.py -k test_Mod_even_dividend_quotient --no-colors`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
