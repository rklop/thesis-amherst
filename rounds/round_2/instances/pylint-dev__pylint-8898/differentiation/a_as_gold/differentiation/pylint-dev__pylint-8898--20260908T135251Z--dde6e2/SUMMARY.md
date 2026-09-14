# Differentiating-test run: `pylint-dev__pylint-8898`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.pylint-dev_1776_pylint-8898:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-02/bidirectional-differentiation/instances/pylint-dev__pylint-8898/a_as_gold/differentiation/pylint-dev__pylint-8898--20260908T135251Z--dde6e2`
- Test: `test_bad_name_regex_csv_separator_after_literal_closing_brace`
- Test command: `cd /testbed && python -m pytest -q tests/config/test_regexp_csv.py::test_bad_name_regex_csv_separator_after_literal_closing_brace`

## Specification gap

A valid regex may contain a literal unmatched closing brace. That brace must not prevent a subsequent top-level comma from separating the next regex in the comma-separated option.

## Input/output contract

Input: Run Pylint on a module defining `first` and `target_name`, with `--bad-names-rgxs=(?:first|}),target_name`. The first regex validly includes a literal `}`; the comma separates it from the second regex.

Expected output: Pylint emits `Disallowed name "first"` and `Disallowed name "target_name"`. Candidate A preserves zero brace depth after the literal `}` and parses two regexes. Candidate B decrements brace depth below zero, parses one combined regex, and emits neither diagnostic.

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
