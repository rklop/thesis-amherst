# 063 — django__django-12050

## Comparison

- Manual reference: **Somewhat different**
- Pipeline-derived bucket: **Very different**
- Exact agreement: **no**
- Ordinal distance: **1**

## Why each side classified it this way

**Manual review:** The reported top-level coercion is fixed by both, but gold defines recursive container behavior.

**Pipeline mapping:** A real pass/fail separation was found and MiniMax rated it high-signal.

## Differentiating test

- Test: `test_nested_iterable_lookup_value`
- Candidate that passed: **candidate_b**
- Specification gap tested: Iterable lookup values must resolve ORM expressions at every nested list/tuple level while preserving each container's input type. The existing test covers only a flat list, so a shallow fix is indistinguishable from recursive resolution.
- Input: Pass Query.resolve_lookup_value() a list containing a top-level F('name') and a nested one-element tuple containing F('created').
- Expected behavior: The result remains a list, its nested container remains a tuple, and both F() references are resolved to SimpleCol expressions.

## MiniMax assessment

- Rating: **high_signal**
- Confidence: **0.95**
- Summary: The test validates that nested F() expressions inside iterables are recursively resolved while preserving container types. Candidate B passes by correctly handling both type preservation and recursive resolution. Candidate A fails because it only preserves the outer container type but doesn't recursively resolve F() expressions nested inside inner tuples. This reveals a meaningful missing specification: the resolve_lookup_value method must handle arbitrarily nested iterables, not just flat ones. The test is general and minimal—it checks the core semantic requirement that both type preservation and recursive expression resolution apply at all nesting levels, which is essential for field types like PickledField that depend on exact type matching.

## Interpretation boundary

Agreement is measured against a human-produced reference review, not an objective ground truth. This note summarizes recorded evidence and does not expose private model chain-of-thought.
