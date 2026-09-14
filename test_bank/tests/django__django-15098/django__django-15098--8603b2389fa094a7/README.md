# test_get_language_from_path_rejects_overlong_subtag

- **Instance:** `django__django-15098`
- **Test ID:** `django__django-15098--8603b2389fa094a7`
- **Status:** Rejected and not used in later generation
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 2

## What it checks

Language subtags parsed from URL prefixes are limited to eight characters. A malformed longer subtag must be rejected before generic-language fallback occurs.

## Expected behavior

The function returns None. It must not interpret the malformed prefix as the supported generic language 'en'.

## Test command

`PYTHONDONTWRITEBYTECODE=1 ./tests/runtests.py i18n.tests.MiscTests.test_get_language_from_path_rejects_overlong_subtag`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
