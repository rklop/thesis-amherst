# Differentiating-test run: `django__django-13158`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-13158:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/evaluation_records/differentiating_pipeline_vs_manual_first100_20260818/runs/django__django-13158--20260818T101721Z--4e13ca`
- Test: `test_combined_queryset_clone_independence`
- Test command: `python tests/runtests.py queries.test_qs_combinators.QuerySetSetOperationTests.test_combined_queryset_clone_independence --verbosity 2`

## Specification gap

QuerySet.all() is documented to return a copy, but combined querysets also contain mutable branch queries. Those branches must be copied so changing a cloned queryset cannot silently change the source queryset. This is especially relevant when none() mutates a cloned combined query.

## Input/output contract

Input: Create a union producing numbers [0, 1, 8, 9], clone it with all(), and add num=0 to the first branch of the clone. Evaluate both querysets.

Expected output: The source union remains [0, 1, 8, 9], while the modified clone returns [0, 8, 9].

## Execution

| Candidate | Ran | Passed | Exit |
| --- | --- | --- | --- |
| candidate_a | True | False | 1 |
| candidate_b | True | True | 0 |

## MiniMax judgment

- Rating: **high_signal**
- Confidence: 0.9
- Summary: The test correctly identifies that candidate B properly implements the copy isolation invariant for combined querysets, while candidate A fails to deep-copy combined_queries during cloning. Candidate B passes because it clones combined_queries in the clone() method and propagates set_empty() to them, which aligns with QuerySet.all() returning an independent copy. The test reveals a real specification gap: QuerySet documentation implies all() returns a copy, but combined querysets shared mutable branch queries without this fix. Candidate B is more specification-conformant.

## Main artifacts

- `selected_test.patch`
- `selected_proposal.json`
- `02_execution/selected/result.json`
- `03_minimax/verdict.json`
- `pipeline.log`
