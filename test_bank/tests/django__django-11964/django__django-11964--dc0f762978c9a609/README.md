# test_custom_choices_str

- **Instance:** `django__django-11964`
- **Test ID:** `django__django-11964--dc0f762978c9a609`
- **Status:** Rejected and not used in later generation
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 1

## What it checks

Value-style stringification is limited to TextChoices and IntegerChoices. The generic Choices extension point for other concrete types retains standard Enum member stringification.

## Expected behavior

str(MoonLandings.APOLLO_11) returns "MoonLandings.APOLLO_11", preserving the member-identifying Enum representation rather than returning the underlying date string "1969-07-20".

## Test command

`cd /testbed && python tests/runtests.py model_enums.tests.CustomChoicesTests.test_custom_choices_str`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
