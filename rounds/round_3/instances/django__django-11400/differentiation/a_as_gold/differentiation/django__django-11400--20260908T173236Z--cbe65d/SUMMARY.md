# Differentiating-test run: `django__django-11400`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-11400:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-03/bidirectional-differentiation/instances/django__django-11400/a_as_gold/differentiation/django__django-11400--20260908T173236Z--cbe65d`
- Test: `test_get_choices_ordering_is_stable_during_filtering`
- Test command: `cd /testbed && ./tests/runtests.py model_fields.tests.GetChoicesOrderingTests.test_get_choices_ordering_is_stable_during_filtering`

## Specification gap

A truthy explicit ordering iterable passed to Field.get_choices() is the ordering for that call and must remain stable while limit_choices_to is processed.

## Input/output contract

Input: Create related Foo objects with a='a' and a='b', pass ordering=['a'], and pass a mapping-compatible limit_choices_to whose key enumeration changes the original list to ['-a'].

Expected output: get_choices() returns the two choices in ascending a order: foo1 ('a') followed by foo2 ('b'). The later mutation of the caller-owned list must not reverse this call's ordering.

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
