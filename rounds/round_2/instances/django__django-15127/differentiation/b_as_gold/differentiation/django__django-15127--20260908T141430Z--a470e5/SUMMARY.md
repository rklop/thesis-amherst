# Differentiating-test run: `django__django-15127`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-15127:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-02/bidirectional-differentiation/instances/django__django-15127/b_as_gold/differentiation/django__django-15127--20260908T141430Z--a470e5`
- Test: `test_level_tags_dict_union_after_override`
- Test command: `cd /testbed && python tests/runtests.py messages_tests.tests.MessageTests.test_level_tags_dict_union_after_override`

## Specification gap

After MESSAGE_TAGS changes, LEVEL_TAGS must remain a coherent dictionary under ordinary dict operations. Union with an empty dict must preserve the refreshed tags, not expose stale or empty backing storage.

## Input/output contract

Input: Override MESSAGE_TAGS so custom level 29 maps to "custom", form `base.LEVEL_TAGS | {}`, and read key 29 from the resulting dictionary.

Expected output: The dictionary union contains key 29 with value "custom".

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
