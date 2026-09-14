# 077 — django__django-12663

## Comparison

- Manual reference: **Very different**
- Pipeline-derived bucket: **Very different**
- Exact agreement: **yes**
- Ordinal distance: **0**

## Why each side classified it this way

**Manual review:** The candidate changes every lookup's input coercion; gold fixes the expression metadata that caused the one lookup to choose a wrong converter.

**Pipeline mapping:** A real pass/fail separation was found and MiniMax rated it high-signal.

## Differentiating test

- Test: `test_subquery_annotation_filter_lazy_to_field`
- Candidate that passed: **candidate_b**
- Specification gap tested: A scalar subquery selecting a ForeignKey must retain the relation’s configured target-field semantics when compared with a lazy model instance. The value must come from the ForeignKey’s `to_field`, which is not necessarily the related object’s primary key.
- Input: Create a Parent with primary key 7 and unique name `parent-key`, plus a ToFieldChild whose ForeignKey targets Parent.name. Annotate the child with a correlated scalar subquery selecting that ForeignKey, then retrieve it by comparing the annotation to a SimpleLazyObject wrapping the Parent.
- Expected behavior: The annotated lookup returns the created ToFieldChild. The comparison must use `parent-key`, not the Parent primary key `7`, and must not raise a lazy-object conversion error.

## MiniMax assessment

- Rating: **high_signal**
- Confidence: **0.95**
- Summary: The test reveals a genuine semantic gap: when filtering by a lazy model instance through a subquery annotation that selects a ForeignKey field, the comparison must use the ForeignKey's configured target field (to_field), not blindly convert to pk. Candidate B correctly preserves the ForeignKey metadata through the subquery, while Candidate A's approach of always using .pk fails for non-pk target fields. The test uses the to_field relationship correctly to demonstrate this distinction.

## Interpretation boundary

Agreement is measured against a human-produced reference review, not an objective ground truth. This note summarizes recorded evidence and does not expose private model chain-of-thought.
