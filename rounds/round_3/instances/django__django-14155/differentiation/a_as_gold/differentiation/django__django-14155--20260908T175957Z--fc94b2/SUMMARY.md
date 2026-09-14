# Differentiating-test run: `django__django-14155`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-14155:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-03/bidirectional-differentiation/instances/django__django-14155/a_as_gold/differentiation/django__django-14155--20260908T175957Z--fc94b2`
- Test: `test_repr_partial_subclass`
- Test command: `cd /testbed && ./tests/runtests.py urlpatterns_reverse.tests.ResolverMatchTests.test_repr_partial_subclass`

## Specification gap

ResolverMatch must expose the underlying callable and pre-bound arguments for any functools.partial instance, including subclasses whose own repr is opaque.

## Input/output contract

Input: Create an opaque-repr functools.partial subclass wrapping empty_view with positional argument 'preset' and keyword argument template_name='template.html', then pass it to ResolverMatch.

Expected output: repr(ResolverMatch(...)) contains 'empty_view', 'preset', and "template_name='template.html'" despite the partial subclass hiding those details in its own repr.

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
