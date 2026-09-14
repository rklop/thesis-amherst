# Differentiating-test run: `matplotlib__matplotlib-25479`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.matplotlib_1776_matplotlib-25479:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-02/bidirectional-differentiation/instances/matplotlib__matplotlib-25479/b_as_gold/differentiation/matplotlib__matplotlib-25479--20260908T135938Z--946a7f`
- Test: `test_set_cmap_runtime_type_hints`
- Test command: `python -m pytest -q lib/matplotlib/tests/test_pyplot.py::test_set_cmap_runtime_type_hints`

## Specification gap

The public `pyplot.set_cmap` API advertises that its argument accepts either a `Colormap` or a registered-name string. That annotation should also be resolvable through Python's standard runtime type-introspection entry point.

## Input/output contract

Input: Call `typing.get_type_hints` on `matplotlib.pyplot.set_cmap` and inspect the resolved type for its `cmap` parameter.

Expected output: Type-hint resolution succeeds and returns `matplotlib.colors.Colormap | str` for `cmap`. Candidate_a instead raises `NameError` because `Colormap` is absent from `pyplot`'s runtime globals.

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
