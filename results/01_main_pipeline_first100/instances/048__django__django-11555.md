# 048 — django__django-11555

## Comparison

- Manual reference: **Somewhat different**
- Pipeline-derived bucket: **Very different**
- Exact agreement: **no**
- Ordinal distance: **1**

## Why each side classified it this way

**Manual review:** The candidate generalizes to more expression types and performs direction handling that gold does not add here.

**Pipeline mapping:** A real pass/fail separation was found and MiniMax rated it high-signal.

## Differentiating test

- Test: `test_unwrapped_function_in_parent_ordering`
- Candidate that passed: **candidate_a**
- Specification gap tested: Related or parent-pointer ordering must support any valid Meta.ordering expression, including a bare database function, not only expressions already wrapped in OrderBy.
- Input: Create multi-table child rows named 'Zebra' and 'alpha'. Order them by the inherited parent pointer, whose parent model declares Meta.ordering=(Lower('name'),).
- Expected behavior: Evaluating the queryset returns ['alpha', 'Zebra'] without raising a type error.

## MiniMax assessment

- Rating: **high_signal**
- Confidence: **0.95**
- Summary: The test validates a real specification gap: parent pointer ordering must handle any valid Meta.ordering expression (including bare functions like Lower), not just strings or OrderBy-wrapped expressions. Candidate A correctly resolves and normalizes expressions, while B only handles pre-wrapped OrderBy and crashes on bare functions. The test's oracle (checking for correct alphabetical ordering without type errors) follows directly from the issue's complaint about crashes when Meta.ordering contains expressions during multi-table inheritance.

## Interpretation boundary

Agreement is measured against a human-produced reference review, not an objective ground truth. This note summarizes recorded evidence and does not expose private model chain-of-thought.
