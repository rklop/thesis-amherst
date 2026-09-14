# 084 — django__django-12965

## Comparison

- Manual reference: **Very different**
- Pipeline-derived bucket: **Very different**
- Exact agreement: **yes**
- Ordinal distance: **0**

## Why each side classified it this way

**Manual review:** The candidate adds local fallback states; gold restores the query invariant expected by compiler logic.

**Pipeline mapping:** A real pass/fail separation was found and MiniMax rated it high-signal.

## Differentiating test

- Test: `test_fast_delete_all_after_evaluation`
- Candidate that passed: **candidate_b**
- Specification gap tested: Deleting all rows must remain a direct single-table DELETE even when the same QuerySet was previously evaluated. Candidate A misses the case where evaluation has created a base-table alias whose reference count was subsequently reset to zero.
- Input: Create one User, evaluate User.objects.all(), then call delete() on that same all-rows QuerySet while capturing the generated SQL.
- Expected behavior: delete() reports one deleted row and executes exactly one DELETE statement containing no SELECT subquery.

## MiniMax assessment

- Rating: **high_signal**
- Confidence: **0.9**
- Summary: The test effectively differentiates between candidates by checking that delete() on an evaluated all-rows QuerySet still produces a simple DELETE without SELECT subquery. Candidate B passes, producing 'DELETE FROM table' without subquery, while candidate A fails producing the problematic subquery. The test aligns with the issue's core complaint about performance regression and LOCK TABLES incompatibility, verifying the expected behavior from a public API perspective rather than checking internal alias state.

## Interpretation boundary

Agreement is measured against a human-produced reference review, not an objective ground truth. This note summarizes recorded evidence and does not expose private model chain-of-thought.
