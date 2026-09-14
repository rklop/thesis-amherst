# test_imported_level_tags_track_override

- **Instance:** `django__django-15127`
- **Test ID:** `django__django-15127--4a53dfcc5cb38078`
- **Status:** Rejected and not used in later generation
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 2

## What it checks

A LEVEL_TAGS reference imported before override_settings() should expose the active MESSAGE_TAGS values. Rebinding only the storage module attribute leaves such references stale.

## Expected behavior

LEVEL_TAGS[constants.INFO] returns 'custom-info' while the override is active.

## Test command

`cd /testbed && python tests/runtests.py messages_tests.tests.MessageTests.test_imported_level_tags_track_override`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
