# test_setting_changed_signal_class

- **Instance:** `django__django-15127`
- **Test ID:** `django__django-15127--235e2f79942d07a0`
- **Status:** Rejected and not used in later generation
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 1

## What it checks

The supplied reference exposes the dispatch Signal class from django.contrib.messages.storage.base when installing its setting_changed receiver; the generated candidate omits that alternate import entry point.

## Expected behavior

The import succeeds, and type(setting_changed) is the imported Signal class. candidate_a instead raises ImportError because its storage.base module doesn't expose Signal.

## Test command

`python tests/runtests.py messages_tests.tests.MessageTests.test_setting_changed_signal_class --verbosity 2`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
