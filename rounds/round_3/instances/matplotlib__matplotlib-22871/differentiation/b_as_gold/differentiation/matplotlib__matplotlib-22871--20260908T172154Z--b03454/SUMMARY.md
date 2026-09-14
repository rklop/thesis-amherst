# Differentiating-test run: `matplotlib__matplotlib-22871`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.matplotlib_1776_matplotlib-22871:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-03/bidirectional-differentiation/instances/matplotlib__matplotlib-22871/b_as_gold/differentiation/matplotlib__matplotlib-22871--20260908T172154Z--b03454`
- Test: `test_concise_formatter_months_without_january_show_year`
- Test command: `cd /testbed && python -m pytest -q lib/matplotlib/tests/test_dates_year_offset.py`

## Specification gap

The direct ConciseDateFormatter.format_ticks entry point must retain the common year as its offset when month-level ticks omit January. The fix must also remain within the repository's configured 79-column source-line limit.

## Input/output contract

Input: Format two monthly ticks, February 1 and March 1 of 2021, using ConciseDateFormatter.format_ticks.

Expected output: The tick labels are ['Feb', 'Mar'] and get_offset() returns '2021'; dates.py contains no source line longer than the repository-configured 79 columns.

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
