# test_implemented_function_evalf_nested_argument_precision

- **Instance:** `sympy__sympy-12096`
- **Test ID:** `sympy__sympy-12096--5c2ce24d7e205951`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 3

## What it checks

Recursive evaluation of an implemented function's arguments must preserve enough working precision for the enclosing numerical implementation, not merely enough digits to display the final result. Otherwise a nested identity function can change a precision-sensitive computation.

## Expected behavior

The returned numeric value has absolute value below 1e-12. The tolerance allows harmless numerical residue while requiring the nested identity implementation to preserve pi accurately enough for the outer high-frequency sine calculation.

## Test command

`cd /testbed && bin/test sympy/core/tests/test_evalf.py -k test_implemented_function_evalf_nested_argument_precision`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
