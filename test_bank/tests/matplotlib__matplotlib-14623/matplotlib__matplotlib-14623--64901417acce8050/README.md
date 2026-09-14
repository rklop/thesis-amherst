# test_nonsingular_reversed_nonpositive_bound

- **Instance:** `matplotlib__matplotlib-14623`
- **Test ID:** `matplotlib__matplotlib-14623--64901417acce8050`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 1

## What it checks

A reversed log range represents intentional inversion only when both endpoints are positive. If one endpoint is outside the log domain, reversing the endpoints must not change the sanitized range.

## Expected behavior

Both inputs produce (0.1, 10): the nonpositive endpoint is replaced by the smallest positive datum and the resulting range is increasing.

## Test command

`cd /testbed && python -m pytest -q lib/matplotlib/tests/test_ticker.py::TestLogLocator::test_nonsingular_reversed_nonpositive_bound`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
