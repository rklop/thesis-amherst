# Differentiating-test run: `django__django-15128`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-15128:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-02/bidirectional-differentiation/instances/django__django-15128/b_as_gold/differentiation/django__django-15128--20260908T140252Z--dac5ae`
- Test: `test_or_with_custom_query_join_override`
- Test command: `python tests/runtests.py queries.tests.QuerySetBitwiseOperationTests.test_or_with_custom_query_join_override`

## Specification gap

Alias-collision handling for QuerySet OR combinations must preserve the established Query.join(join, reuse=None) override contract. A custom Query subclass using that signature should still support combinations whose left and right sides create offset, overlapping sequential aliases.

## Input/output contract

Input: Create one student attached to Room 1. Build two Classroom querysets backed by a Query subclass that overrides join(join, reuse=None): the left side filters through the students many-to-many relation, while the right side traverses Classroom -> School -> Classroom -> School -> Classroom and selects Room 2. OR the querysets and apply distinct().

Expected output: Evaluating the combined queryset succeeds and returns each of the four classrooms exactly once. candidate_a instead passes a new reuse_with_aliases keyword to the existing join override and raises TypeError during the OR operation.

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
