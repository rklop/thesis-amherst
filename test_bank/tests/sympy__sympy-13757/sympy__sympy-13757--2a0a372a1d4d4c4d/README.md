# test_poly_vector_scalar_multiplication_commutes

- **Instance:** `sympy__sympy-13757`
- **Test ID:** `sympy__sympy-13757--2a0a372a1d4d4c4d`
- **Status:** Rejected and not used in later generation
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 1

## What it checks

Poly multiplication must respect SymPy's arithmetic priority protocol when interacting with higher-priority public expression types, preserving scalar multiplication semantics on either operand side.

## Expected behavior

Both operand orders produce equal vectors: Poly(x) * C.i == C.i * Poly(x).

## Test command

`python bin/test sympy/polys/tests/test_poly_vector_dispatch.py --no-colors`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
