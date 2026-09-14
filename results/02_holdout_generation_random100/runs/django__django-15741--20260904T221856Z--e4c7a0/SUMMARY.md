# Differentiating-test run: `django__django-15741`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-15741:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/evaluation_records/differentiating_gold_pass_current_candidate_fail_random100_20260904/runs/django__django-15741--20260904T221856Z--e4c7a0`
- Test: `test_lazy_format_type_uses_call_language`
- Test command: `./tests/runtests.py i18n.test_lazy_format_type.FormattingTests.test_lazy_format_type_uses_call_language`

## Specification gap

get_format() should select the localization language active when the call begins before forcing a deferred format name. Evaluating the lazy argument must not retroactively change the locale selected for that call.

## Input/output contract

Input: Call get_format() while English is active with use_l10n=True and a django.utils.functional.lazy string that activates German when evaluated, then resolves to "DATE_FORMAT".

Expected output: The call returns the English DATE_FORMAT, "N j, Y". Candidate B captures English before evaluating the lazy value; candidate A evaluates it first and instead returns the German format, "j. F Y".

## Execution

| Candidate | Ran | Passed | Exit |
| --- | --- | --- | --- |
| candidate_a | True | False | 1 |
| candidate_b | True | True | 0 |

## MiniMax judgment

- Rating: **high_signal**
- Confidence: 0.95
- Summary: The generated test exercises a meaningful semantic distinction between the patches: candidate B captures the active language at call time before evaluating the lazy string, while candidate A evaluates the lazy string first (which activates German internally), resulting in the German format being returned. The test verifies this ambient-language invariant at the lazy-evaluation boundary without inspecting internals. Candidate B passes and returns the English format 'N j, Y', which aligns with the expected behavior that the language should be determined when get_format() is invoked, not when the lazy argument is later evaluated. This is a defensible oracle based on API semantics: the use_l10n parameter should apply to the language active during the call, not be affected by side effects from lazy evaluation.

## Main artifacts

- `selected_test.patch`
- `selected_proposal.json`
- `02_execution/selected/result.json`
- `03_minimax/verdict.json`
- `pipeline.log`
