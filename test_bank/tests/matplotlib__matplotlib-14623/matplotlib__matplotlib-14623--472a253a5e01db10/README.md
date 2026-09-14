# test_view_limits_reversed_with_nonpositive_endpoint

- **Instance:** `matplotlib__matplotlib-14623`
- **Test ID:** `matplotlib__matplotlib-14623--472a253a5e01db10`
- **Status:** Rejected and not used in later generation
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 2

## What it checks

Reversed log limits preserve inversion only when both endpoints are positive. If one endpoint is non-positive and must be replaced, repairing it must not accidentally create an inverted interval.

## Expected behavior

The invalid zero endpoint is replaced by the axis minimum-positive value, producing the ordered limits (1, 10).

## Test command

`cd /testbed && python -m pytest -q lib/matplotlib/tests/test_ticker.py::TestLogLocator::test_view_limits_reversed_with_nonpositive_endpoint`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
