# test_fixture_dirs_with_normalized_duplicates

- **Instance:** `django__django-15987`
- **Test ID:** `django__django-15987--810112f6927c1ebb`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 1

## What it checks

FIXTURE_DIRS entries that normalize to the same pathname are duplicates even when represented differently, such as a pathlib.Path and a string containing a '.' segment.

## Expected behavior

loaddata raises ImproperlyConfigured with "settings.FIXTURE_DIRS contains duplicates." before loading fixtures.

## Test command

`PYTHONPATH=. python tests/runtests.py fixtures_regress.tests.TestFixtures.test_fixture_dirs_with_normalized_duplicates --verbosity 0`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
