# Differentiating-test run: `matplotlib__matplotlib-22871`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.matplotlib_1776_matplotlib-22871:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-02/bidirectional-differentiation/instances/matplotlib__matplotlib-22871/b_as_gold/differentiation/matplotlib__matplotlib-22871--20260908T133154Z--949e93`
- Test: `test_concise_formatter_rejects_nonscalar_show_offset_consistently`
- Test command: `python -m pytest -q lib/matplotlib/tests/test_dates.py::test_concise_formatter_rejects_nonscalar_show_offset_consistently`

## Specification gap

`show_offset` is documented as a single boolean. An ambiguous non-scalar value must not be accepted or rejected depending on whether January happens to occur among the ticks.

## Input/output contract

Input: Create a `ConciseDateFormatter` with `show_offset=np.array([True, False])`, then format month-level ticks for February–March and January–February 2021.

Expected output: Both calls raise `ValueError` because the multi-element array has no unambiguous boolean value.

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
