# test_storage_base_does_not_expose_signal

- **Instance:** `django__django-15127`
- **Test ID:** `django__django-15127--4d54de7a2d230a17`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 1

## What it checks

Refreshing message level tags on setting changes must not expose the unrelated dispatch Signal class through django.contrib.messages.storage.base. Local documentation assigns Signal to django.dispatch while documenting Message and BaseStorage as the storage.base interfaces.

## Expected behavior

hasattr(base, 'Signal') returns False. candidate_a imports only receiver, so the assertion passes; candidate_b also imports Signal into the module namespace, so it fails.

## Test command

`./tests/runtests.py messages_tests.tests.MessageTests.test_storage_base_does_not_expose_signal`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
