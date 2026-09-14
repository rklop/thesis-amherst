# 094 — django__django-13158

## Comparison

- Manual reference: **Somewhat different**
- Pipeline-derived bucket: **Very different**
- Exact agreement: **no**
- Ordinal distance: **1**

## Why each side classified it this way

**Manual review:** Both return no rows, but query-state propagation and the lifecycle at which emptiness is represented differ.

**Pipeline mapping:** A real pass/fail separation was found and MiniMax rated it high-signal.

## Differentiating test

- Test: `test_combined_queryset_clone_independence`
- Candidate that passed: **candidate_b**
- Specification gap tested: QuerySet.all() is documented to return a copy, but combined querysets also contain mutable branch queries. Those branches must be copied so changing a cloned queryset cannot silently change the source queryset. This is especially relevant when none() mutates a cloned combined query.
- Input: Create a union producing numbers [0, 1, 8, 9], clone it with all(), and add num=0 to the first branch of the clone. Evaluate both querysets.
- Expected behavior: The source union remains [0, 1, 8, 9], while the modified clone returns [0, 8, 9].

## MiniMax assessment

- Rating: **high_signal**
- Confidence: **0.9**
- Summary: The test correctly identifies that candidate B properly implements the copy isolation invariant for combined querysets, while candidate A fails to deep-copy combined_queries during cloning. Candidate B passes because it clones combined_queries in the clone() method and propagates set_empty() to them, which aligns with QuerySet.all() returning an independent copy. The test reveals a real specification gap: QuerySet documentation implies all() returns a copy, but combined querysets shared mutable branch queries without this fix. Candidate B is more specification-conformant.

## Interpretation boundary

Agreement is measured against a human-produced reference review, not an objective ground truth. This note summarizes recorded evidence and does not expose private model chain-of-thought.
