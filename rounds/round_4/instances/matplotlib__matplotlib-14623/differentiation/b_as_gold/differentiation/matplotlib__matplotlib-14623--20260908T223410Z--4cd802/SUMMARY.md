# Differentiating-test run: `matplotlib__matplotlib-14623`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.matplotlib_1776_matplotlib-14623:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-04/bidirectional-differentiation/instances/matplotlib__matplotlib-14623/b_as_gold/differentiation/matplotlib__matplotlib-14623--20260908T223410Z--4cd802`
- Test: `test_log_ylim_partial_update_preserves_inversion`
- Test command: `cd /testbed && python -m pytest -q lib/matplotlib/tests/test_axes.py::test_log_ylim_partial_update_preserves_inversion`

## Specification gap

An already inverted log axis should remain inverted when a one-sided limit update retains the other endpoint, produces equal limits, and triggers automatic singular-limit expansion.

## Input/output contract

Input: Create a log-scaled y-axis, invert it with set_ylim(100, 1), then set only the bottom limit to 1 while leaving the top limit unchanged.

Expected output: The equal limits are automatically expanded, and both the returned and stored y-limits remain in descending order.

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
