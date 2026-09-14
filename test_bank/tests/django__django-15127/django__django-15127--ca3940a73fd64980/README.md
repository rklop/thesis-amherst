# test_level_tags_dict_union_after_override

- **Instance:** `django__django-15127`
- **Test ID:** `django__django-15127--ca3940a73fd64980`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 2

## What it checks

After MESSAGE_TAGS changes, LEVEL_TAGS must remain a coherent dictionary under ordinary dict operations. Union with an empty dict must preserve the refreshed tags, not expose stale or empty backing storage.

## Expected behavior

The dictionary union contains key 29 with value "custom".

## Test command

`cd /testbed && python tests/runtests.py messages_tests.tests.MessageTests.test_level_tags_dict_union_after_override`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
