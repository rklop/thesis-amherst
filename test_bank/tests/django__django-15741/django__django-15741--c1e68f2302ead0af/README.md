# test_get_format_bytes

- **Instance:** `django__django-15741`
- **Test ID:** `django__django-15741--c1e68f2302ead0af`
- **Status:** Rejected and not used in later generation
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 1

## What it checks

get_format() should decode a bytes-valued textual format parameter rather than treating Python's bytes representation as the format. This is a narrow type variant of the issue's requirement that string-like parameters be accepted.

## Expected behavior

The function returns the text "Y-m-d". candidate_a instead returns "b'Y-m-d'" because str() preserves the bytes representation, while candidate_b's force_str() decodes the bytes.

## Test command

`python tests/runtests.py i18n.tests.FormattingTests.test_get_format_bytes`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
