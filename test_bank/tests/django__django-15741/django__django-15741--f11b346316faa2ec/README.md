# test_get_format_bytes_uses_str_coercion

- **Instance:** `django__django-15741`
- **Test ID:** `django__django-15741--f11b346316faa2ec`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 1

## What it checks

Non-string format_type values are normalized using Python's ordinary str() semantics. In particular, bytes are represented rather than UTF-8 decoded into a setting name.

## Expected behavior

Return str(b"DEBUG"), which is "b'DEBUG'", rather than decoding the bytes and returning "DEBUG".

## Test command

`cd /testbed && ./tests/runtests.py i18n.tests.FormattingTests.test_get_format_bytes_uses_str_coercion`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
