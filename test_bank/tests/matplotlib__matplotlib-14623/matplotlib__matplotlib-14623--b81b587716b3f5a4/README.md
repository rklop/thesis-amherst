# test_nonsingular_invalid_inverted_limits

- **Instance:** `matplotlib__matplotlib-14623`
- **Test ID:** `matplotlib__matplotlib-14623--b81b587716b3f5a4`
- **Status:** Rejected and not used in later generation
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 1

## What it checks

Preserving reversed logarithmic limits must not reinterpret a nonpositive second endpoint as a lower bound. For `(10, 0)`, zero remains an invalid upper endpoint and triggers the standard fallback range.

## Expected behavior

A `UserWarning` containing “Data has no positive values” is emitted, and the returned limits are `(1, 10)`.

## Test command

`cd /testbed && python -m pytest -q lib/matplotlib/tests/test_ticker.py::TestLogLocator::test_nonsingular_invalid_inverted_limits`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
