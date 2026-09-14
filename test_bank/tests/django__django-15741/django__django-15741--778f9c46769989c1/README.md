# test_get_format_has_no_docstring

- **Instance:** `django__django-15741`
- **Test ID:** `django__django-15741--778f9c46769989c1`
- **Status:** Rejected and not used in later generation
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 2

## What it checks

The authoritative patch places format_type coercion before the function's descriptive string literal. Consequently, that literal is no longer get_format's Python docstring and public introspection returns None.

## Expected behavior

get_format.__doc__ is None.

## Test command

`./tests/runtests.py i18n.tests.FormattingTests.test_get_format_has_no_docstring`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
