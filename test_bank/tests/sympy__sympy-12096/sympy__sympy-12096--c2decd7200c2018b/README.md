# test_implemented_function_evalf_symbolic_result

- **Instance:** `sympy__sympy-12096`
- **Test ID:** `sympy__sympy-12096--c2decd7200c2018b`
- **Status:** Rejected and not used in later generation
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 1

## What it checks

Recursive evalf must apply not only to arguments passed into `_imp_`, but also to a symbolic numeric expression returned by `_imp_`. This includes complex-valued SymPy expressions that cannot be directly converted to `Float`.

## Expected behavior

The call returns the same numeric complex value as `(2*I).evalf()` (displayed as approximately `2.00000000000000*I`), rather than remaining as the unevaluated expression `f(2)`.

## Test command

`cd /testbed && bin/test sympy/core/tests/test_evalf.py -k test_implemented_function_evalf_symbolic_result`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
