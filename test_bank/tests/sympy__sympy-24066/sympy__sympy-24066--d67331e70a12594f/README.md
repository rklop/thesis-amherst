# test_issue_24066_multi_argument_dimensionless_function

- **Instance:** `sympy__sympy-24066`
- **Test ID:** `sympy__sympy-24066--d67331e70a12594f`
- **Status:** Rejected and not used in later generation
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 1

## What it checks

Functions whose arguments are all dimensionless must have one aggregate dimension, Dimension(1), regardless of arity. The collector must continue returning its documented two-item (factor, dimension) tuple rather than one dimension entry per argument.

## Expected behavior

Collection completes without an unpacking error, and the returned aggregate dimension is Dimension(1).

## Test command

`cd /testbed && python bin/test sympy/physics/units/tests/test_quantities.py -k test_issue_24066_multi_argument_dimensionless_function --no-colors`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
