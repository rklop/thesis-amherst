# Differentiating-test run: `sympy__sympy-22456`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.sympy_1776_sympy-22456:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-02/bidirectional-differentiation/instances/sympy__sympy-22456/b_as_gold/differentiation/sympy__sympy-22456--20260908T141131Z--4d0197`
- Test: `test_String_subclass_argument_invariance`
- Test command: `cd /testbed && /opt/miniconda3/envs/testbed/bin/python -c "from sympy.codegen.tests.test_ast import test_String_subclass_argument_invariance; test_String_subclass_argument_invariance()"`

## Specification gap

String argument invariance must preserve every declared constructor field for subclasses, not assume String-derived objects always have arity one.

## Input/output contract

Input: Define a String subclass using Token's slot convention with text and tag fields, then construct TaggedString('status', Integer(7)) and rebuild it through value.func(*value.args).

Expected output: Reconstruction returns an equal TaggedString with text == 'status' and tag == Integer(7).

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
