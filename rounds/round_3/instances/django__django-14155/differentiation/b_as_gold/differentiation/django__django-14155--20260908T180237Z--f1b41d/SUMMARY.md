# Differentiating-test run: `django__django-14155`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-14155:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-03/bidirectional-differentiation/instances/django__django-14155/b_as_gold/differentiation/django__django-14155--20260908T180237Z--f1b41d`
- Test: `test_partial_repr_uses_current_arguments`
- Test command: `./tests/runtests.py urlpatterns_reverse.tests.ResolverMatchTests.test_partial_repr_uses_current_arguments`

## Specification gap

ResolverMatch.__repr__() should describe the actual partial callback and its current bound arguments, rather than a stale snapshot captured when ResolverMatch was initialized.

## Input/output contract

Input: Construct a ResolverMatch with functools.partial(empty_view, template_name='initial.html'), mutate the partial's public keywords dictionary to template_name='changed.html', and render the ResolverMatch.

Expected output: repr(match) contains repr(callback), including template_name='changed.html'. The supplied gold reads the live partial; the generated candidate reconstructs it from stale copied keywords.

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
