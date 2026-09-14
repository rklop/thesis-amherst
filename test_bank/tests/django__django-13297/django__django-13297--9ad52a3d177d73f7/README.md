# test_get_context_data_url_kwargs_are_native_values

- **Instance:** `django__django-13297`
- **Test ID:** `django__django-13297--9ad52a3d177d73f7`
- **Status:** Rejected and not used in later generation
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 2

## What it checks

URL kwargs passed to an overridden TemplateView.get_context_data() must remain their resolver-produced native values. Deprecation-warning wrappers may be added to the final template context, but must not replace the values received by the override.

## Expected behavior

response.context_data['received_url_kwargs'] is {'offer_slug': 'summer-sale'} without a JSON serialization error.

## Test command

`cd /testbed && ./tests/runtests.py generic_views.test_base.TemplateViewTest.test_get_context_data_url_kwargs_are_native_values`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
