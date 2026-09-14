# Differentiating-test run: `matplotlib__matplotlib-25479`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.matplotlib_1776_matplotlib-25479:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-02/bidirectional-differentiation/instances/matplotlib__matplotlib-25479/a_as_gold/differentiation/matplotlib__matplotlib-25479--20260908T135925Z--2b4634`
- Test: `test_set_cmap_uses_current_colormap_class_after_reload`
- Test command: `cd /testbed && python -m pytest -q lib/matplotlib/tests/test_pyplot.py::test_set_cmap_uses_current_colormap_class_after_reload`

## Specification gap

`pyplot.set_cmap` must recognize instances of the currently loaded public `matplotlib.colors.Colormap` class, rather than relying on a stale class binding captured when `pyplot` was imported.

## Input/output contract

Input: Reload `matplotlib.colors`, create a `ListedColormap` named `internal_name`, register it under `test_registered_alias`, retrieve it through the public registry, and pass that current `Colormap` instance to `pyplot.set_cmap`.

Expected output: `set_cmap` completes without exception and `matplotlib.rcParams['image.cmap']` becomes `test_registered_alias`.

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
