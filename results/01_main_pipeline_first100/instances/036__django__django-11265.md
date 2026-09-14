# 036 — django__django-11265

## Comparison

- Manual reference: **Very different**
- Pipeline-derived bucket: **Very different**
- Exact agreement: **yes**
- Ordinal distance: **0**

## Why each side classified it this way

**Manual review:** One repairs query aliases after trimming logic has acted; the other preserves the filtered-join invariant before trimming.

**Pipeline mapping:** A real pass/fail separation was found and MiniMax rated it high-signal.

## Differentiating test

- Test: `test_negated_q_on_filtered_relation_is_order_independent`
- Candidate that passed: **candidate_b**
- Specification gap tested: Combining positive and negated Q() lookups on the same filtered reverse relation must exclude against the entire filtered relation and must not depend on AND operand order. Candidate A instead appears to correlate the negated subquery to the previously joined child row.
- Input: Jane has two books admitted by the FilteredRelation, one ending in “A” and one ending in “B”. Query for suffix A while negating suffix B, in both operand orders.
- Expected behavior: Both query orderings return an empty sequence. Candidate A is expected to return Jane for the positive-then-negated ordering, while candidate B returns empty for both.

## MiniMax assessment

- Rating: **high_signal**
- Confidence: **0.95**
- Summary: The test exercises a genuine public API behavior: the logical equivalence of Q-expression operands regardless of order when applied to FilteredRelation. The test uses standard Django QuerySet filtering with Q objects, testing a semantic invariant rather than implementation details. Candidate B correctly returns empty for both orderings, preserving the boolean-algebra property that (A AND NOT B) equals (NOT B AND A). Candidate A fails this invariant by returning Jane for one ordering, demonstrating incorrect handling of negated lookups on multi-valued FilteredRelation.

## Interpretation boundary

Agreement is measured against a human-produced reference review, not an objective ground truth. This note summarizes recorded evidence and does not expose private model chain-of-thought.
