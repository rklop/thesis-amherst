# test_Mul_reciprocal_of_positive_power

- **Instance:** `sympy__sympy-21612`
- **Test ID:** `sympy__sympy-21612--de1214068138b94a`
- **Status:** Rejected and not used in later generation
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 1

## What it checks

Parentheses are required when a nested reciprocal denominator renders as division and would otherwise be ambiguous, but not around every nested power. A positive-power denominator should retain its ordinary canonical form because exponentiation already binds more tightly than division.

## Expected behavior

Both constructions print exactly "x/y**2". Candidate_b instead prints the unevaluated construction as "x/(y**2)" because it parenthesizes every Pow denominator.

## Test command

`cd /testbed && python bin/test sympy/printing/tests/test_str.py -k test_Mul_reciprocal_of_positive_power --no-colors`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
