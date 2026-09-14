# Differentiating-test run: `matplotlib__matplotlib-24026`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.matplotlib_1776_matplotlib-24026:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-03/bidirectional-differentiation/instances/matplotlib__matplotlib-24026/a_as_gold/differentiation/matplotlib__matplotlib-24026--20260908T175133Z--96e17e`
- Test: `test_stackplot_rejects_single_color_string`
- Test command: `python -m pytest -q lib/matplotlib/tests/test_axes.py::test_stackplot_rejects_single_color_string`

## Specification gap

The `colors` parameter is documented as a sequence of colors, not a single color value. A scalar color string such as `"red"` must therefore remain invalid rather than being silently accepted as a one-element color sequence.

## Input/output contract

Input: Create an Axes and call `ax.stackplot([0, 1], [1, 1], colors="red")`, passing a scalar color string instead of a sequence of color specifications.

Expected output: The public call raises `ValueError`. The targeted pytest command exits with status 0 only when that rejection occurs.

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
