# Differentiating-test run: `pydata__xarray-7229`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.pydata_1776_xarray-7229:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/evaluation_records/differentiating_gold_pass_current_candidate_fail_random100_20260904/runs/pydata__xarray-7229--20260904T224907Z--455a73`
- Test: `test_where_keep_attrs_with_array_valued_attrs`
- Test command: `cd /testbed && python -m pytest -q xarray/tests/test_computation.py::test_where_keep_attrs_with_array_valued_attrs`

## Specification gap

xr.where(..., keep_attrs=True) must preserve coordinate attributes independently of the DataArray's own attributes, including when both layers contain NumPy-array-valued metadata under the same key.

## Input/output contract

Input: A DataArray with values [1, 2], a sample coordinate whose actual_range attribute is array([0, 1]), and a parent actual_range attribute of array([1, 2]); it is passed as x to xr.where(True, x, 0, keep_attrs=True).

Expected output: The result is identical to x: values remain [1, 2], the parent attribute remains array([1, 2]), and the sample coordinate retains array([0, 1]) without raising an ambiguous-truth-value error.

## Execution

| Candidate | Ran | Passed | Exit |
| --- | --- | --- | --- |
| candidate_a | True | False | 1 |
| candidate_b | True | True | 0 |

## MiniMax judgment

- Rating: **high_signal**
- Confidence: 0.95
- Summary: The test exercises the core issue: preserving coordinate attributes independently from variable attributes when using xr.where(..., keep_attrs=True). Candidate_a fails with a ValueError when attrs contain NumPy array values, revealing that its approach of using identity/membership checks ('x_attrs in attrs') cannot handle array-valued attributes. Candidate_b correctly handles this by directly copying attrs from x to each layer of the result. The test has a clear oracle (assert_identical(x, actual)) and exercises real supported functionality (attrs can contain arbitrary objects including arrays). The winner (candidate_b) is more specification-conformant because it correctly preserves both variable and coordinate attributes as required by the issue.

## Main artifacts

- `selected_test.patch`
- `selected_proposal.json`
- `02_execution/selected/result.json`
- `03_minimax/verdict.json`
- `pipeline.log`
