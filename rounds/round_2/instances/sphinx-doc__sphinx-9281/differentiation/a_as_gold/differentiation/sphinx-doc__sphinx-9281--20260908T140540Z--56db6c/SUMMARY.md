# Differentiating-test run: `sphinx-doc__sphinx-9281`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.sphinx-doc_1776_sphinx-9281:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-02/bidirectional-differentiation/instances/sphinx-doc__sphinx-9281/a_as_gold/differentiation/sphinx-doc__sphinx-9281--20260908T140540Z--56db6c`
- Test: `test_signature_enum_default_with_formattable_name`
- Test command: `cd /testbed && python -m pytest -q tests/test_util_inspect.py::test_signature_enum_default_with_formattable_name`

## Specification gap

When an Enum exposes a string-subclass name with custom formatting, the generated signature should use that name's standard formatting protocol rather than coercing it with str().

## Input/output contract

Input: A function whose default argument is an Enum member. The Enum overrides its public name property to return a str subclass whose underlying text is "internal-name" but whose __format__ result is "ValueA".

Expected output: stringify_signature() returns "(value=Choice.ValueA)". candidate_b instead renders "(value=Choice.internal-name)" because %s bypasses the custom __format__ behavior.

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
