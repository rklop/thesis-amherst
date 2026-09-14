# Differentiating-test run: `django__django-15127`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-15127:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-04/bidirectional-differentiation/instances/django__django-15127/a_as_gold/differentiation/django__django-15127--20260908T224645Z--f3015e`
- Test: `test_level_tags_existing_key_setdefault`
- Test command: `cd /testbed && python tests/runtests.py messages_tests.tests.MessageTests.test_level_tags_existing_key_setdefault`

## Specification gap

LEVEL_TAGS must remain a coherent dictionary after MESSAGE_TAGS changes: every configured key visible through membership and lookup must also be treated as existing by standard dict operations such as setdefault().

## Input/output contract

Input: Use override_settings() to map the standard INFO message level to 'custom-info', then call LEVEL_TAGS.setdefault(INFO, 'fallback') and read Message.level_tag.

Expected output: INFO is present in LEVEL_TAGS, setdefault() returns the existing value 'custom-info' without substituting 'fallback', and Message.level_tag is 'custom-info'.

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
