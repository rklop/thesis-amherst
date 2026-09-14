# test_datetime_cast_date_without_connection_timezone

- **Instance:** `django__django-11138`
- **Test ID:** `django__django-11138--370bfc009a0314ed`
- **Status:** Rejected and not used in later generation
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 2

## What it checks

A missing database timezone is a sentinel, not a timezone named "None". Date-cast SQL must not attempt timezone conversion unless the connection supplies a concrete source timezone.

## Expected behavior

datetime_cast_date_sql() returns "DATE(created_at)". candidate_b skips conversion, while candidate_a produces "DATE(CONVERT_TZ(created_at, 'None', 'Europe/Paris'))".

## Test command

`python tests/runtests.py backends.mysql.test_operations.DatabaseOperationsTests.test_datetime_cast_date_without_connection_timezone`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
