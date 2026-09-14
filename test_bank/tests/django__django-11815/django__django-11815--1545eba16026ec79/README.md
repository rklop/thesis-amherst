# test_named_integer_regex_flag

- **Instance:** `django__django-11815`
- **Test ID:** `django__django-11815--1545eba16026ec79`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 2

## What it checks

Compiled-regex serialization should preserve the symbolic name of an int-compatible regex flag when that name remains available after flag normalization, extending the issue's name-over-value rule to this nested enum entry point.

## Expected behavior

MigrationWriter.serialize() returns "re.compile('^foo$', re.RegexFlag['IGNORECASE'])" with the import set {'import re'}, and evaluating that expression produces an equivalent regex object. candidate_a instead emits the numeric flag value 2.

## Test command

`python tests/runtests.py migrations.test_regex_flag_serialization.RegexFlagSerializationTests.test_named_integer_regex_flag`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
