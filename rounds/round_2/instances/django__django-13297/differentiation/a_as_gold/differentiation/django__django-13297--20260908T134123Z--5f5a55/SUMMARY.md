# Differentiating-test run: `django__django-13297`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-13297:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-02/bidirectional-differentiation/instances/django__django-13297/a_as_gold/differentiation/django__django-13297--20260908T134123Z--5f5a55`
- Test: `test_get_context_data_url_kwargs_are_native_values`
- Test command: `cd /testbed && ./tests/runtests.py generic_views.test_base.TemplateViewTest.test_get_context_data_url_kwargs_are_native_values`

## Specification gap

URL kwargs passed to an overridden TemplateView.get_context_data() must remain their resolver-produced native values. Deprecation-warning wrappers may be added to the final template context, but must not replace the values received by the override.

## Input/output contract

Input: Invoke a TemplateView subclass with offer_slug='summer-sale'. Its get_context_data() JSON-serializes and deserializes the received kwargs, then exposes the result through response.context_data.

Expected output: response.context_data['received_url_kwargs'] is {'offer_slug': 'summer-sale'} without a JSON serialization error.

## Execution

| Candidate | Ran | Passed | Exit |
| --- | --- | --- | --- |
| candidate_a | True | True | 0 |
| candidate_b | True | False | 1 |

## Main artifacts

- `selected_test.patch`
- `selected_proposal.json`
- `02_execution/selected/result.json`
- `03_minimax/verdict.json`
- `pipeline.log`
