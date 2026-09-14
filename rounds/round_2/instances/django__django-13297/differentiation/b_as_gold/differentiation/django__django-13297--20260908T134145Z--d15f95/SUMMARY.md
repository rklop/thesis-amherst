# Differentiating-test run: `django__django-13297`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-13297:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-02/bidirectional-differentiation/instances/django__django-13297/b_as_gold/differentiation/django__django-13297--20260908T134145Z--d15f95`
- Test: `test_template_params_can_be_used_in_query`
- Test command: `./tests/runtests.py generic_views.test_base.DeprecationTests.test_template_params_can_be_used_in_query`

## Specification gap

A URL kwarg exposed in TemplateView's final context must remain usable as an ORM filter value, not merely arrive unwrapped inside an overridden get_context_data().

## Input/output contract

Input: Create an Author with slug "bar", request the ordinary TemplateView route with URL kwarg foo="bar", then pass response.context['foo'] directly to Author.objects.filter().

Expected output: The ORM query completes without a SQLite parameter-binding error and reports that the matching Author exists.

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
