# Differentiating-test run: `django__django-11885`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-11885:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/evaluation_records/differentiating_pipeline_vs_manual_first100_20260818/runs/django__django-11885--20260818T093948Z--16c9ad`
- Test: `test_fast_delete_combined_relationships_respects_batch_size`
- Test command: `python tests/runtests.py delete.tests.FastDeleteTests.test_fast_delete_combined_relationships_respects_batch_size --verbosity 2`

## Specification gap

Combining cascade fast-delete predicates must preserve backend parameter-limit batching. Queries for different relationship columns may be OR-combined within a batch, but batches must not all be collapsed into one oversized DELETE.

## Input/output contract

Input: Create one Origin, 500 Referrer rows, and 500 SecondReferrer rows where both cascading foreign keys reference the same Referrer. Delete the Origin. On SQLite, each relationship separately fits one 500-object batch, while a combined two-column predicate is limited to 499 objects per batch.

Expected output: Deletion returns a total of 1001 deleted rows, no SecondReferrer rows remain, and the SecondReferrer table is deleted through exactly two backend-sized DELETE statements on SQLite.

## Execution

| Candidate | Ran | Passed | Exit |
| --- | --- | --- | --- |
| candidate_a | True | False | 1 |
| candidate_b | True | True | 0 |

## MiniMax judgment

- Rating: **high_signal**
- Confidence: 0.95
- Summary: The test validates that combined fast-delete queries must preserve backend parameter-limit batching, a critical invariant of the deletion system. Candidate A fails by collapsing all predicates into one oversized DELETE statement (1 query instead of 2), violating batching limits. Candidate B correctly groups relationship fields before calculating batch size, producing the expected number of safe combined DELETE statements (2 batches). The test targets genuine specification requirements derived from the issue's goal of combining queries while maintaining existing system invariants.

## Main artifacts

- `selected_test.patch`
- `selected_proposal.json`
- `02_execution/selected/result.json`
- `03_minimax/verdict.json`
- `pipeline.log`
