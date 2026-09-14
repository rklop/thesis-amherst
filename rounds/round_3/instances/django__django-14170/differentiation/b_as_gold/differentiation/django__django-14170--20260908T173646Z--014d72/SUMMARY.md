# Differentiating-test run: `django__django-14170`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-14170:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-03/bidirectional-differentiation/instances/django__django-14170/b_as_gold/differentiation/django__django-14170--20260908T173646Z--014d72`
- Test: `test_bounds_accept_iso_year_keyword`
- Test command: `./tests/runtests.py lookup.test_lookups.IsoYearLookupTests.test_bounds_accept_iso_year_keyword --verbosity 2`

## Specification gap

The ISO-year lookup bounds entry point accepts the semantically named `iso_year` keyword and computes ISO-week-year boundaries, which may extend beyond the corresponding calendar year.

## Input/output contract

Input: Construct an exact ISO-year lookup over a DateField expression and request bounds with `iso_year=2020`.

Expected output: The helper returns the backend-adapted inclusive interval from Monday 2019-12-30 through Sunday 2021-01-03.

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
