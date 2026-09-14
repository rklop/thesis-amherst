# Differentiating-test run: `django__django-14170`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-14170:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/evaluation_records/differentiating_gold_pass_current_candidate_fail_random100_20260904/runs/django__django-14170--20260904T232455Z--dd13ec`
- Test: `test_iso_year_lookup_bound_params`
- Test command: `python tests/runtests.py backends.sqlite.test_operations.SQLiteOperationsTests.test_iso_year_lookup_bound_params`

## Specification gap

Optimized ISO-year lookup bounds must pass through the backend's DateField value adapter, just like ordinary year bounds. Otherwise generated SQL parameters may have types unsupported by a database driver.

## Input/output contract

Input: Compile a SQLite queryset filtering an Item DateField with date__iso_year=2020 and inspect the parameters returned by Query.sql_with_params().

Expected output: The two bound parameters are the SQLite-adapted strings ('2019-12-30', '2021-01-03').

## Execution

| Candidate | Ran | Passed | Exit |
| --- | --- | --- | --- |
| candidate_a | True | False | 1 |
| candidate_b | True | True | 0 |

## MiniMax judgment

- Rating: **high_signal**
- Confidence: 0.95
- Summary: The generated test correctly identifies that candidate_a fails to apply backend value adaptation to ISO year bounds, returning raw datetime.date objects instead of properly adapted strings. Candidate_b passes because it extends the existing year_lookup_bounds methods with an iso_year parameter, preserving the adaptation step that candidate_a's new methods skip. This is a specification-conformant difference: Django's ORM should adapt bound parameters for the database backend, consistent with how regular year lookups work.

## Main artifacts

- `selected_test.patch`
- `selected_proposal.json`
- `02_execution/selected/result.json`
- `03_minimax/verdict.json`
- `pipeline.log`
