# test_none_body_capability_probe

- **Instance:** `psf__requests-1142`
- **Test ID:** `psf__requests-1142--cac440b6530bb269`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 1

## What it checks

Removing the default zero length must not otherwise change the established body-capability detection path used while preparing a bodyless request.

## Expected behavior

The prepared request has no Content-Length header, while the existing seek-capability check for the absent body remains observable. candidate_b fails because its new outer None guard skips that check.

## Test command

`cd /testbed && pytest -q test_content_length_regression.py`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
