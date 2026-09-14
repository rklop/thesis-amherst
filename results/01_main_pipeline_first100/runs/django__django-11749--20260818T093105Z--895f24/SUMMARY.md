# Differentiating-test run: `django__django-11749`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-11749:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/evaluation_records/differentiating_pipeline_vs_manual_first100_20260818/runs/django__django-11749--20260818T093105Z--895f24`
- Test: `test_call_command_required_group_preserves_action_order`
- Test command: `python tests/runtests.py user_commands.tests.CommandTests.test_call_command_required_group_preserves_action_order`

## Specification gap

call_command() should preserve the parser-visible ordering of options when translating keyword arguments, including options from required mutually exclusive groups. Candidate A appends ordinary required options first and group options afterward, reordering custom Action execution; candidate B keeps parser action order.

## Input/output contract

Input: A command declares a required mutually exclusive option before an ordinary required option. A public argparse.Action records invocation order. The command is invoked once with ordered CLI arguments and once with the equivalent ordered keyword arguments.

Expected output: Both public entry points return exactly "group_value,required_value". The keyword invocation must be observationally equivalent to the CLI invocation.

## Execution

| Candidate | Ran | Passed | Exit |
| --- | --- | --- | --- |
| candidate_a | True | False | 1 |
| candidate_b | True | True | 0 |

## MiniMax judgment

- Rating: **low_signal**
- Confidence: 0.75
- Summary: The test checks the internal order of argparse Action execution, which is an implementation detail not part of the public API or the issue's specification. Candidate B happens to preserve parser action order while candidate A partitions by category, causing different action ordering. However, the core bug fix is about making mutually exclusive group arguments work with kwargs—not about preserving any particular action execution order. The test uses a custom argparse.Action to record internal execution order, which is not part of Django's documented management command contract.

## Main artifacts

- `selected_test.patch`
- `selected_proposal.json`
- `02_execution/selected/result.json`
- `03_minimax/verdict.json`
- `pipeline.log`
