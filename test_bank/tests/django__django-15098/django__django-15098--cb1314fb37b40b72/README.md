# test_language_prefix_with_variant

- **Instance:** `django__django-15098`
- **Test ID:** `django__django-15098--cb1314fb37b40b72`
- **Status:** Rejected and not used in later generation
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 1

## What it checks

An exact language tag configured in LANGUAGES remains a valid i18n URL prefix when it includes a BCP 47 variant after the language, script, and region subtags.

## Expected behavior

The i18n-prefixed endpoint resolves successfully with HTTP status 200.

## Test command

`python tests/runtests.py i18n.patterns.tests.URLPrefixTests.test_language_prefix_with_variant --verbosity 2`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
