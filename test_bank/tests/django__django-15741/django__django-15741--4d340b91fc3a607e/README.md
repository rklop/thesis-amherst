# test_get_format_keeps_docstring

- **Instance:** `django__django-15741`
- **Test ID:** `django__django-15741--4d340b91fc3a607e`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 2

## What it checks

Accepting a lazy format_type must not remove get_format()'s existing public documentation metadata. candidate_b places executable code before the function docstring, causing get_format.__doc__ to become None.

## Expected behavior

get_format.__doc__ contains "format_type is the name of the format". candidate_a preserves this output; candidate_b exposes None instead.

## Test command

`cd /testbed && python tests/runtests.py i18n.tests.FormattingTests.test_get_format_keeps_docstring`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
