# test_powered_creation_operator_latex_is_braced

- **Instance:** `sympy__sympy-21930`
- **Test ID:** `sympy__sympy-21930--42d19b2df3ff9574`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 1

## What it checks

Bracing must apply to both bosonic and fermionic creation operators, and their public import must not trigger Python invalid-escape diagnostics for the LaTeX \dagger command.

## Expected behavior

The subprocess imports successfully and prints exactly two lines: {b^\dagger_{p}}^{2} and {a^\dagger_{p}}^{2}.

## Test command

`python bin/test sympy/physics/tests/test_secondquant_latex_regression.py --no-subprocess`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
