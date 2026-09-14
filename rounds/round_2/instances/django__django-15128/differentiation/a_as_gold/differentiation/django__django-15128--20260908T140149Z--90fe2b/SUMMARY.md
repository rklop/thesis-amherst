# Differentiating-test run: `django__django-15128`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-15128:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-02/bidirectional-differentiation/instances/django__django-15128/a_as_gold/differentiation/django__django-15128--20260908T140149Z--90fe2b`
- Test: `test_or_with_multichar_alias_prefix`
- Test command: `cd /testbed && ./tests/runtests.py queries.tests.QuerySetBitwiseOperationTests.test_or_with_multichar_alias_prefix`

## Specification gap

QuerySet combination must avoid alias collisions when Django uses a multi-character alias prefix, not only its initial one-character prefix.

## Input/output contract

Input: Create two BaseUser rows connected by a Task's owner and creator foreign keys. OR a StaffUser-related query with a query traversing both reverse Task paths, using Django's valid multi-character alias prefix `AA`.

Expected output: The combined QuerySet evaluates without an alias-generation exception and returns exactly the two created BaseUser primary keys.

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
