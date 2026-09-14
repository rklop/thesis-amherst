# test_Poly_mul_immutable_matrix

- **Instance:** `sympy__sympy-13757`
- **Test ID:** `sympy__sympy-13757--23fbcb666a13f219`
- **Status:** Rejected and not used in later generation
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 2

## What it checks

Left-hand multiplication must also dispatch to Poly when the expression is an ImmutableMatrix, an Expr-derived public type with its own arithmetic precedence.

## Expected behavior

The result is Poly(2*x, x, domain='ZZ'), rather than an ImmutableMatrix containing that polynomial. Candidate_b only ties ImmutableMatrix's precedence and therefore leaves the matrix as the outer result.

## Test command

`cd /testbed && python bin/test -C --no-colors -k test_Poly_mul_immutable_matrix sympy/polys/tests/test_polytools.py`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
