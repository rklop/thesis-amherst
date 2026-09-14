# Differentiating-test run: `pylint-dev__pylint-8898`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.pylint-dev_1776_pylint-8898:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-02/bidirectional-differentiation/instances/pylint-dev__pylint-8898/b_as_gold/differentiation/pylint-dev__pylint-8898--20260908T135342Z--cf854f`
- Test: `test_csv_regex_separator_after_literal_braces`
- Test command: `python -m pytest -q tests/config/test_config.py::test_csv_regex_separator_after_literal_braces`

## Specification gap

A comma must remain a list separator after a valid regex containing literal braces, even when `{` occurs outside a character class and `}` occurs inside one. Brace tracking must not collapse the following regex into the preceding entry.

## Input/output contract

Input: Lint a module declaring `bar` with default bad names cleared and `--bad-names-rgxs=foo{[}],bar`. This represents two valid regexes: `foo{[}]`, which matches `foo{}`, and `bar`.

Expected output: Pylint emits `C0104: Disallowed name "bar" (disallowed-name)`, proving that `bar` was parsed as the second bad-name regex.

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
