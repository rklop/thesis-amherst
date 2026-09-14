# Differentiating-test run: `matplotlib__matplotlib-14623`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.matplotlib_1776_matplotlib-14623:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-04/bidirectional-differentiation/instances/matplotlib__matplotlib-14623/a_as_gold/differentiation/matplotlib__matplotlib-14623--20260908T223315Z--0b1c24`
- Test: `test_equal_positive_log_limits_do_not_invert_axis`
- Test command: `cd /testbed && python -m pytest -q lib/matplotlib/tests/test_logscale_limits.py::test_equal_positive_log_limits_do_not_invert_axis`

## Specification gap

Equal positive limits have no descending direction to preserve. When a singular log-axis range is automatically expanded, it should retain the default non-inverted orientation rather than inventing an inversion.

## Input/output contract

Input: Create a log-scaled y-axis and call the public Axes.set_ylim(10, 10) with identical positive endpoints.

Expected output: A singular-limit warning is emitted, and the expanded limits are ordered so that bottom < 10 < top; the resulting y-axis is not inverted.

## Execution

| Candidate | Ran | Passed | Exit |
| --- | --- | --- | --- |
| candidate_a | True | True | 0 |
| candidate_b | True | False | 1 |

## Main artifacts

- `selected_test.patch`
- `selected_proposal.json`
- `02_execution/selected/result.json`
- `03_minimax/verdict.json`
- `pipeline.log`
