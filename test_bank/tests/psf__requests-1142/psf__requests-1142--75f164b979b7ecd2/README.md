# test_bodyless_get_does_not_probe_for_body_capabilities

- **Instance:** `psf__requests-1142`
- **Test ID:** `psf__requests-1142--75f164b979b7ecd2`
- **Status:** Rejected and not used in later generation
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 1

## What it checks

A missing request body is semantically distinct from a zero-length body. Preparing a bodyless GET should leave Content-Length absent and should not process the None sentinel as though it were a body object with capabilities.

## Expected behavior

Request preparation completes without error, prepared.body is None, and the prepared headers contain no Content-Length field.

## Test command

`python -m unittest test_requests.RequestsTestCase.test_bodyless_get_does_not_probe_for_body_capabilities`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
