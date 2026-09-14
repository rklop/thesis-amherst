# test_log_scale_inverted_limits_with_positive_domain_scalar

- **Instance:** `matplotlib__matplotlib-14623`
- **Test ID:** `matplotlib__matplotlib-14623--73ab5f54b8b41e3d`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 3

## What it checks

For two descending positive log limits, inversion follows from their ordering and the positivity of the smaller endpoint; preserving inversion must not require an additional greater-than-zero comparison on the larger scalar.

## Expected behavior

set_ylim completes without error, get_ylim returns (10.0, 1.0), and yaxis_inverted() is true.

## Test command

`python -m pytest -q lib/matplotlib/tests/test_log_inverted_limits.py`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
