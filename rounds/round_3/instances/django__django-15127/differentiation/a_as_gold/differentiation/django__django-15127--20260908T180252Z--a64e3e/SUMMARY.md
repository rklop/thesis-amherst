# Differentiating-test run: `django__django-15127`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-15127:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-03/bidirectional-differentiation/instances/django__django-15127/a_as_gold/differentiation/django__django-15127--20260908T180252Z--a64e3e`
- Test: `test_update_level_tags_accepts_positional_setting`
- Test command: `cd /testbed && python tests/runtests.py messages_tests.tests.MessageTests.test_update_level_tags_accepts_positional_setting`

## Specification gap

The MESSAGE_TAGS refresh hook must accept the changed setting name as an explicit positional argument, not only as a keyword supplied by override_settings.

## Input/output contract

Input: Temporarily disconnect the automatic receiver, override MESSAGE_TAGS from INFO='outer' to INFO='inner', and directly invoke update_level_tags('MESSAGE_TAGS').

Expected output: A newly constructed INFO Message has the externally visible level_tag 'inner'. candidate_a accepts the positional setting argument and refreshes the tags; candidate_b raises TypeError.

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
