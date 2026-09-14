# Differentiating-test run: `django__django-14155`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-14155:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-02/bidirectional-differentiation/instances/django__django-14155/b_as_gold/differentiation/django__django-14155--20260908T141035Z--5c0a45`
- Test: `test_repr_partial_callable_object`
- Test command: `cd /testbed && python tests/runtests.py urlpatterns_reverse.tests.ResolverMatchTests.test_repr_partial_callable_object`

## Specification gap

A partial view must preserve ResolverMatch's existing support for callable objects, even when the underlying callable has no function-style __name__ attribute.

## Input/output contract

Input: Construct ResolverMatch with partial(CallableView(), setting='value'), where CallableView is a valid callable view with a stable repr and no __name__.

Expected output: Construction succeeds and repr(match) is exactly "ResolverMatch(func=functools.partial(callable_view, setting='value'), args=(), kwargs={}, url_name=None, app_names=[], namespaces=[], route=None)".

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
