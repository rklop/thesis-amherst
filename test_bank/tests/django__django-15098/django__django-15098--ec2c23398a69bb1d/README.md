# test_get_language_from_request_path_gettext_modifier

- **Instance:** `django__django-15098`
- **Test ID:** `django__django-15098--ec2c23398a69bb1d`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 2

## What it checks

Adding support for three-part language prefixes must preserve Django's existing gettext-style @modifier syntax. The generated candidate accepts only hyphen-separated components, although the repository already recognizes sr-RS@latin as a valid language code.

## Expected behavior

get_language_from_request(request, check_path=True) returns the exact configured language code 'sr-RS@latin'.

## Test command

`cd /testbed && python tests/runtests.py i18n.tests.CountrySpecificLanguageTests.test_get_language_from_request_path_gettext_modifier`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
