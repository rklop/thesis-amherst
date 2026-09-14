# test_get_language_from_path_language_info_fallback

- **Instance:** `django__django-15098`
- **Test ID:** `django__django-15098--a2647c7f8e7fa7db`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 1

## What it checks

Expanding recognized language-tag shapes in URL paths must preserve Django's existing registered language aliases and fallbacks, not restrict resolution to textual parent/child prefixes.

## Expected behavior

The function returns 'zh-hans', using Django's registered zh-cn fallback.

## Test command

`cd /testbed && python tests/runtests.py i18n.tests.MiscTests.test_get_language_from_path_language_info_fallback`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
