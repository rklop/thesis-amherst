# Differentiating-test run: `django__django-11490`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-11490:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/evaluation_records/differentiating_pipeline_vs_manual_first100_20260818/runs/django__django-11490--20260818T092125Z--13019c`
- Test: `test_union_projection_reuse_with_distinct_operands`
- Test command: `./tests/runtests.py queries.test_qs_combinators.QuerySetSetOperationTests.test_union_projection_reuse_with_distinct_operands`

## Specification gap

Re-projecting a composed queryset must not retain a previous projection on a distinct non-first operand. The existing test only covers reusing the same QuerySet object as both operands.

## Input/output contract

Input: Create one ReservedName row, union two separately constructed ReservedName.objects.all() querysets, evaluate a two-column values_list(), then evaluate a one-column values_list() on the same compound queryset.

Expected output: The first evaluation returns ('a', 2), and the second returns (2,) without a database column-count error.

## Execution

| Candidate | Ran | Passed | Exit |
| --- | --- | --- | --- |
| candidate_a | True | False | 1 |
| candidate_b | True | True | 0 |

## MiniMax judgment

- Rating: **high_signal**
- Confidence: 0.95
- Summary: The test exercises the exact bug reported in the issue: re-projecting a composed queryset with different column counts. Candidate B (the fix) correctly clones all combined queries before propagating the projection, while candidate A only clones when the query object identity matches the first combined query. The test passes with candidate B and fails with candidate A, demonstrating that the fix must handle distinct operand objects, not just the same object reused twice. The oracle is based on the issue's explicit example behavior and Django's documented QuerySet semantics.

## Main artifacts

- `selected_test.patch`
- `selected_proposal.json`
- `02_execution/selected/result.json`
- `03_minimax/verdict.json`
- `pipeline.log`
