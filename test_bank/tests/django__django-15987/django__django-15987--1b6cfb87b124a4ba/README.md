# test_fixture_dirs_with_pathlike_duplicates

- **Instance:** `django__django-15987`
- **Test ID:** `django__django-15987--1b6cfb87b124a4ba`
- **Status:** Rejected and not used in later generation
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 1

## What it checks

FIXTURE_DIRS duplicate detection must compare os.PathLike entries using their filesystem representation (the os.fspath() protocol), not their unrelated str() representation.

## Expected behavior

Calling loaddata raises ImproperlyConfigured with "settings.FIXTURE_DIRS contains duplicates." Candidate A converts both entries with os.fspath() and detects the duplicate; candidate B converts the object with str(), so it misses the duplicate.

## Test command

`cd /testbed && python tests/runtests.py fixtures_regress.tests.TestFixtures.test_fixture_dirs_with_pathlike_duplicates`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
