# test_implemented_function_sympy_result_precision

- **Instance:** `sympy__sympy-12096`
- **Test ID:** `sympy__sympy-12096--71d8fb015d9d72eb`
- **Status:** Rejected and not used in later generation
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 2

## What it checks

Recursive evalf applies not only to an implemented function's arguments, but also to a SymPy expression returned by its implementation. That returned expression must be evaluated using the requested precision before conversion to Float.

## Expected behavior

The result numerically approximates 3.1415926535897932384626433832795028841971693993751, with absolute error below 1e-49 compared with pi.evalf(50).

## Test command

`python bin/test sympy/core/tests/test_evalf.py -k test_implemented_function_sympy_result_precision --no-colors`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
