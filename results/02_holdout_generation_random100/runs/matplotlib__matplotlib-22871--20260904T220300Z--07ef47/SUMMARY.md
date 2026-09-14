# Differentiating-test run: `matplotlib__matplotlib-22871`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.matplotlib_1776_matplotlib-22871:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/evaluation_records/differentiating_gold_pass_current_candidate_fail_random100_20260904/runs/matplotlib__matplotlib-22871--20260904T220300Z--07ef47`
- Test: `test_concise_formatter_custom_year_level_offset`
- Test command: `cd /testbed && python -m pytest -q lib/matplotlib/tests/test_dates.py::test_concise_formatter_custom_year_level_offset`

## Specification gap

ConciseDateFormatter should retain a requested year-level offset when customized two-digit year labels need that offset to supply century context. Candidate A unconditionally suppresses offsets whenever years vary, while candidate B preserves the configured offset for this case.

## Input/output contract

Input: Format June 1 in 2020 and 2021 using `%y` year-level tick labels, `century %C` as the year-level offset format, and `show_offset=True`.

Expected output: The public formatter returns tick labels `['20', '21']`, and `get_offset()` returns `century 20`.

## Execution

| Candidate | Ran | Passed | Exit |
| --- | --- | --- | --- |
| candidate_a | True | False | 1 |
| candidate_b | True | True | 0 |

## MiniMax judgment

- Rating: **high_signal**
- Confidence: 0.85
- Summary: The test validates that ConciseDateFormatter respects custom year-level format configurations (two-digit year labels with century offset) via public API. Candidate B correctly preserves the explicitly-requested offset while Candidate A unconditionally suppresses it whenever years vary, breaking valid use cases. The test uses public methods (format_ticks, get_offset) and tests a meaningful interaction between formats, offset_formats, and show_offset parameters that follows from the API contract.

## Main artifacts

- `selected_test.patch`
- `selected_proposal.json`
- `02_execution/selected/result.json`
- `03_minimax/verdict.json`
- `pipeline.log`
