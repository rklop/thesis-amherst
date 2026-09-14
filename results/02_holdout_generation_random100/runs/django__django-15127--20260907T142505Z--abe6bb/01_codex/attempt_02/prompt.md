You are designing one high-signal differentiating input/output
test for SWE-Bench Verified instance django__django-15127.

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


This is retry 2. The previous proposal did not establish a valid
pass/fail separation. Use this execution feedback and produce a materially
different or corrected test:

<prior_execution_feedback>
PipelineError: Codex attempt 1 exited 1; see /home/rocky/SWE-ABS/mini-swe-agent/evaluation_records/differentiating_gold_pass_current_candidate_fail_random100_20260904/runs/django__django-15127--20260907T142505Z--abe6bb/01_codex/attempt_01/events.jsonl
</prior_execution_feedback>

Run artifact root: /home/rocky/SWE-ABS/mini-swe-agent/evaluation_records/differentiating_gold_pass_current_candidate_fail_random100_20260904/runs/django__django-15127--20260907T142505Z--abe6bb
