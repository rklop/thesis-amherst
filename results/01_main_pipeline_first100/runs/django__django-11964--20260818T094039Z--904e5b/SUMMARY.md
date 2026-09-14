# Differentiating-test run: `django__django-11964`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-11964:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/evaluation_records/differentiating_pipeline_vs_manual_first100_20260818/runs/django__django-11964--20260818T094039Z--904e5b`
- Test: `custom_date_choices_str_uses_value`
- Test command: `cd /testbed && python tests/runtests.py model_enums.tests.CustomChoicesTests.test_str_custom_concrete_type`

## Specification gap

The issue’s conversion invariant is unspecified for Django’s documented custom `models.Choices` subclasses. These are a public way to use concrete field types other than `str` and `int`, so a custom choice should stringify like its underlying concrete value too.

## Input/output contract

Input: Call `str()` on `MoonLandings.APOLLO_11`, an existing `datetime.date`/`models.Choices` member whose underlying value is 1969-07-20.

Expected output: The result is the externally usable date string `1969-07-20`, rather than the enum identity `MoonLandings.APOLLO_11`.

## Execution

| Candidate | Ran | Passed | Exit |
| --- | --- | --- | --- |
| candidate_a | True | False | 1 |
| candidate_b | True | True | 0 |

## MiniMax judgment

- Rating: **high_signal**
- Confidence: 0.9
- Summary: The test correctly differentiates between candidates by checking whether the fix generalizes to custom Choices subclasses with non-standard concrete types (datetime.date). Candidate B passes because it adds __str__ to the base Choices class, while Candidate A fails because it only adds it to IntegerChoices and TextChoices. The test's oracle (str(enum) should equal str(value)) is directly derived from the issue's complaint that str() returns the enum identity instead of the value. While the test doesn't directly exercise model field retrieval, testing the enum's __str__ is sufficient since that's where the fix is applied and it determines the behavior when accessing model fields.

## Main artifacts

- `selected_test.patch`
- `selected_proposal.json`
- `02_execution/selected/result.json`
- `03_minimax/verdict.json`
- `pipeline.log`
