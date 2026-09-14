# 030 — django__django-11149

## Comparison

- Manual reference: **Not different**
- Pipeline-derived bucket: **Somewhat different**
- Exact agreement: **no**
- Ordinal distance: **1**

## Why each side classified it this way

**Manual review:** The edit rule is the same; gold mainly factors the target-model lookup for reuse in the existing view check.

**Pipeline mapping:** A real pass/fail separation was found but MiniMax rated it low_signal.

## Differentiating test

- Test: `test_inline_change_m2m_private_method_name_collision`
- Candidate that passed: **candidate_b**
- Specification gap tested: Fixing auto-created many-to-many permissions must preserve compatibility with existing InlineModelAdmin subclasses. A subclass-private method that Django previously ignored must not unexpectedly become an internal permission hook.
- Input: Create a TabularInline for Author.books.through whose subclass already defines `_has_change_permission(self, request)`. Grant the user change_book permission, then call the public `has_change_permission(request)` entry point.
- Expected behavior: The permission check returns True without invoking the unrelated subclass-private method. Candidate A instead dispatches to that method with an additional obj argument and raises TypeError.

## MiniMax assessment

- Rating: **low_signal**
- Confidence: **0.75**
- Summary: The test exposes a real backward-compatibility issue in candidate A (method name collision with user subclasses), but it tests implementation internals rather than the core permission bug described in the issue. The test is narrowly focused on a specific helper method name conflict that would be rare in practice, rather than validating the intended permission semantics for auto-created M2M inlines.

## Interpretation boundary

Agreement is measured against a human-produced reference review, not an objective ground truth. This note summarizes recorded evidence and does not expose private model chain-of-thought.
