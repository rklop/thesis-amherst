# test_prepare_value_preserves_encoder_option_order

- **Instance:** `django__django-13512`
- **Test ID:** `django__django-13512--e01e7b7586e4f228`
- **Status:** Rejected and not used in later generation
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 1

## What it checks

When adding Unicode-preserving serialization, JSONField.prepare_value() should retain the existing custom-encoder option first and append ensure_ascii=False. This preserves the observable keyword order for forwarding wrappers.

## Expected behavior

prepare_value() returns '{"country": "中国"}', and the wrapper observes keyword options in the order ['cls', 'ensure_ascii'].

## Test command

`./tests/runtests.py forms_tests.field_tests.test_jsonfield.JSONFieldTest.test_prepare_value_preserves_encoder_option_order`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
