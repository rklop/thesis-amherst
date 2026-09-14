# Differentiating-test run: `django__django-13297`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-13297:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-04/bidirectional-differentiation/instances/django__django-13297/b_as_gold/differentiation/django__django-13297--20260908T222606Z--e27263`
- Test: `test_url_kwargs_are_serializable_in_get_context_data`
- Test command: `cd /testbed && python tests/runtests.py generic_views.test_base.TemplateViewTest.test_url_kwargs_are_serializable_in_get_context_data`

## Specification gap

URL kwargs passed to an overridden TemplateView.get_context_data() must remain their ordinary URL-converter values for use by Python code. Deprecation handling for exposing those values in template context must not replace the arguments received by the override with lazy wrapper objects.

## Input/output contract

Input: Invoke a TemplateView with offer_slug='summer-sale'. Its get_context_data() override serializes the received kwargs mapping with json.dumps() and stores the result in the response context.

Expected output: The serialized response-context value decodes to {'offer_slug': 'summer-sale'} without requiring an explicit str() conversion.

## Execution

| Candidate | Ran | Passed | Exit |
| --- | --- | --- | --- |
| candidate_a | True | False | 1 |
| candidate_b | True | True | 0 |

## Main artifacts

- `selected_test.patch`
- `selected_proposal.json`
- `02_execution/selected/result.json`
- `03_minimax/verdict.json`
- `pipeline.log`
