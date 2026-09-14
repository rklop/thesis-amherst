# Differentiating-test run: `django__django-14170`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-14170:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-04/bidirectional-differentiation/instances/django__django-14170/b_as_gold/differentiation/django__django-14170--20260908T223552Z--1359d2`
- Test: `test_extract_iso_year_renamed_subclass_lookup`
- Test command: `python tests/runtests.py db_functions.datetime.test_extract_trunc.DateFunctionTests.test_extract_iso_year_renamed_subclass_lookup`

## Specification gap

ISO-year comparison semantics belong to an ExtractIsoYear expression, not to the literal name under which its subclass is exposed. A subclass that retains ISO-year extraction but uses another lookup name must still use ISO-year boundaries when filtered.

## Input/output contract

Input: Create a row dated 2014-12-31, annotate it with an ExtractIsoYear subclass named `renamed_iso_year`, and filter the annotation for 2015. The date is in calendar year 2014 but ISO year 2015.

Expected output: The filtered values are exactly `[(date(2014, 12, 31), 2015)]`. candidate_b selects ISO-specific comparison lookups by expression type. candidate_a checks whether `lookup_name == 'iso_year'`, incorrectly applies calendar-year bounds to the renamed ISO-year expression, and returns no rows.

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
