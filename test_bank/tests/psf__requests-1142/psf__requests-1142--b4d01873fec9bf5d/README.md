# test_get_without_body_has_no_none_special_case

- **Instance:** `psf__requests-1142`
- **Test ID:** `psf__requests-1142--b4d01873fec9bf5d`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 2

## What it checks

A bodyless GET must omit Content-Length; preparation should not retain a purported special case for checking seek capability on None, which has no such capability.

## Expected behavior

The prepared request has no Content-Length header, and the implementation contains no claim that seek capability is checked specially when the body is None.

## Test command

`cd /testbed && python test_content_length_regression.py`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
