# Differentiating-test run: `django__django-14155`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-14155:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-02/bidirectional-differentiation/instances/django__django-14155/a_as_gold/differentiation/django__django-14155--20260908T141007Z--88cd77`
- Test: `test_repr_partial_uses_current_func`
- Test command: `python tests/runtests.py urlpatterns_reverse.tests.ResolverMatchTests.test_repr_partial_uses_current_func`

## Specification gap

ResolverMatch.__repr__() should describe the partial currently exposed through its public func attribute, rather than a separate partial cached only during construction.

## Input/output contract

Input: Construct a ResolverMatch with partial(empty_view), replace its public func attribute with partial(absolute_kwargs_view), and call repr().

Expected output: The representation contains func=functools.partial(<the repr of absolute_kwargs_view>, ), identifying the replacement partial's underlying function.

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
