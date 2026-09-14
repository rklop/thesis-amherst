# Differentiating-test run: `django__django-15127`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-15127:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-03/bidirectional-differentiation/instances/django__django-15127/b_as_gold/differentiation/django__django-15127--20260908T180310Z--c70645`
- Test: `test_update_level_tags_rejects_positional_setting`
- Test command: `python tests/runtests.py messages_tests.tests.MessageTests.test_update_level_tags_rejects_positional_setting`

## Specification gap

The MESSAGE_TAGS refresh hook is a signal receiver whose payload is keyword-only. A positional `setting` argument must not be treated as a valid alternate calling convention.

## Input/output contract

Input: Call the registered MESSAGE_TAGS update receiver directly with `'MESSAGE_TAGS'` as one positional argument.

Expected output: The call raises TypeError because the receiver accepts only keyword signal payloads. The supplied gold receiver uses only `**kwargs`; the generated candidate incorrectly accepts `setting` positionally and returns normally.

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
