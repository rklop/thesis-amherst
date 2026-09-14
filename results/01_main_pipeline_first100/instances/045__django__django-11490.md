# 045 — django__django-11490

## Comparison

- Manual reference: **Very different**
- Pipeline-derived bucket: **Very different**
- Exact agreement: **yes**
- Ordinal distance: **0**

## Why each side classified it this way

**Manual review:** The candidate handles an aliasing shape such as self-union, while gold protects the general combined-query case.

**Pipeline mapping:** A real pass/fail separation was found and MiniMax rated it high-signal.

## Differentiating test

- Test: `test_union_projection_reuse_with_distinct_operands`
- Candidate that passed: **candidate_b**
- Specification gap tested: Re-projecting a composed queryset must not retain a previous projection on a distinct non-first operand. The existing test only covers reusing the same QuerySet object as both operands.
- Input: Create one ReservedName row, union two separately constructed ReservedName.objects.all() querysets, evaluate a two-column values_list(), then evaluate a one-column values_list() on the same compound queryset.
- Expected behavior: The first evaluation returns ('a', 2), and the second returns (2,) without a database column-count error.

## MiniMax assessment

- Rating: **high_signal**
- Confidence: **0.95**
- Summary: The test exercises the exact bug reported in the issue: re-projecting a composed queryset with different column counts. Candidate B (the fix) correctly clones all combined queries before propagating the projection, while candidate A only clones when the query object identity matches the first combined query. The test passes with candidate B and fails with candidate A, demonstrating that the fix must handle distinct operand objects, not just the same object reused twice. The oracle is based on the issue's explicit example behavior and Django's documented QuerySet semantics.

## Interpretation boundary

Agreement is measured against a human-produced reference review, not an objective ground truth. This note summarizes recorded evidence and does not expose private model chain-of-thought.
