You are designing one high-signal differentiating input/output
test for SWE-Bench Verified instance scikit-learn__scikit-learn-13439.

Work read-only and use only the supplied local files. Do not use web search, GitHub or any other connector, upstream commits, pull requests, historical runs, or gold/reference patches. Treat all external sources as prohibited, even if a tool offers them. Inspect
these files:

- `00_inputs/instance.json`: issue statement and original benchmark metadata.
- `00_inputs/candidate_a.patch`: first independently generated solution.
- `00_inputs/candidate_b.patch`: second independently generated solution.
- `workspace/repo`: pristine repository at the instance base commit.

Both candidate patches may pass the existing benchmark tests. Find a semantic
disagreement that reveals a missing part of the intended public specification.
Prefer boundary cases, alternate public entry points, arity/type variants, or
invariants that should hold generally. Avoid implementation-detail assertions,
mock-only checks, timing thresholds, formatting trivia, and tests that merely
encode the gold patch.

Return a minimal test-only unified git diff relative to `workspace/repo`, plus
one targeted shell command that runs that test inside the standard SWE-Bench
container from `/testbed`. The command must return 0 when the test passes and
nonzero when it fails. Do not modify source code. Do not execute either
candidate yourself; the outer pipeline will apply and run the proposal in two
fresh isolated containers.

Explain the concrete input and externally observable expected output. Predict
which candidate should pass, but do not force a split if the evidence is weak.
The final response must conform exactly to the supplied JSON schema.

This run has a strict directional acceptance rule. Produce only a test whose
defensible expected behavior is:

- `candidate_b` passes.
- `candidate_a` fails.

The outer pipeline will reject an opposite-direction split even if it is a
real separation. Set `expected_passing_candidate` to
`candidate_b`. Do not weaken or reverse the oracle merely to
force the requested direction.


This is retry 3. The previous proposal did not establish a valid
pass/fail separation. Use this execution feedback and produce a materially
different or corrected test:

<prior_execution_feedback>
## candidate_a
returncode=92 passed=False test_ran=False
log=02_execution/attempt_02/candidate_a.log
```
[pipeline] checking candidate patch
[pipeline] checking generated test patch
error: corrupt patch at line 16

```

## candidate_b
returncode=92 passed=False test_ran=False
log=02_execution/attempt_02/candidate_b.log
```
[pipeline] checking candidate patch
[pipeline] checking generated test patch
error: corrupt patch at line 16

```
</prior_execution_feedback>

Run artifact root: /home/rocky/SWE-ABS/mini-swe-agent/evaluation_records/differentiating_gold_pass_current_candidate_fail_random100_20260904/runs/scikit-learn__scikit-learn-13439--20260904T231149Z--7faeb9
