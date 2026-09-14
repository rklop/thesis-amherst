# 023 — django__django-11087

## Comparison

- Manual reference: **Very different**
- Pipeline-derived bucket: **Very different**
- Exact agreement: **yes**
- Ordinal distance: **0**

## Why each side classified it this way

**Manual review:** The optimization is applied at a different layer and uses different safety conditions and field discovery.

**Pipeline mapping:** A real pass/fail separation was found and MiniMax rated it high-signal.

## Differentiating test

- Test: `test_cascade_with_select_related_base_manager`
- Candidate that passed: **candidate_b**
- Specification gap tested: Field deferral during cascade collection must preserve a related model's customized base-manager query. In particular, adding only() must not conflict with select_related() already applied by that public manager entry point.
- Input: Create an origin, a cascading child whose configured base manager selects the origin relation, and a cascading grandchild that prevents the child from being fast-deleted. Delete the origin.
- Expected behavior: origin.delete() completes without an exception, and database existence checks for the origin, child, and grandchild all return False.

## MiniMax assessment

- Rating: **high_signal**
- Confidence: **0.95**
- Summary: The test correctly identifies that candidate A breaks when deletion field optimization interacts with custom base managers using select_related(), while candidate B handles this case properly by avoiding field deferral when select_related is present. This is a meaningful specification gap: the optimization should not break queries that already use select_related, as such usage is legitimate and documented.

## Interpretation boundary

Agreement is measured against a human-produced reference review, not an objective ground truth. This note summarizes recorded evidence and does not expose private model chain-of-thought.
