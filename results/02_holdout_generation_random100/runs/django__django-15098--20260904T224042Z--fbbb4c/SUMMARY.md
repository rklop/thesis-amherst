# Differentiating-test run: `django__django-15098`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-15098:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/evaluation_records/differentiating_gold_pass_current_candidate_fail_random100_20260904/runs/django__django-15098--20260904T224042Z--fbbb4c`
- Test: `test_get_language_from_path_case_insensitive_subsequent_fallback`
- Test command: `./tests/runtests.py i18n.tests.MiscTests.test_get_language_from_path_case_insensitive_subsequent_fallback`

## Specification gap

The path-based public API must preserve case-insensitive language fallback when parsing the newly supported language-script-region form. Candidate A adds a case-sensitive prefilter that rejects mixed-case tags before Django's normal fallback resolver runs.

## Input/output contract

Input: Configure only `zh-hans` and call `translation.get_language_from_path('/zh-Hans-CN/')`.

Expected output: The function returns `zh-hans`, falling back from the mixed-case language-script-region tag to the configured language-script variant.

## Execution

| Candidate | Ran | Passed | Exit |
| --- | --- | --- | --- |
| candidate_a | True | False | 1 |
| candidate_b | True | True | 0 |

## MiniMax judgment

- Rating: **high_signal**
- Confidence: 0.9
- Summary: The test verifies a genuine regression in case-insensitive fallback semantics when extending URL-prefix parsing to support language-script-region tags. Candidate A adds a case-sensitive prefilter that breaks Django's documented case-insensitive fallback behavior, while Candidate B preserves it. The test correctly identifies this semantic gap in the original issue, which focused on supporting the new format but did not explicitly address case-handling nuances.

## Main artifacts

- `selected_test.patch`
- `selected_proposal.json`
- `02_execution/selected/result.json`
- `03_minimax/verdict.json`
- `pipeline.log`
