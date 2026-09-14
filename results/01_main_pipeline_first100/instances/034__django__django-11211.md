# 034 — django__django-11211

## Comparison

- Manual reference: **Very different**
- Pipeline-derived bucket: **Very different**
- Exact agreement: **yes**
- Ordinal distance: **0**

## Why each side classified it this way

**Manual review:** The candidate patches one consumer; gold corrects the field-level invariant used across ORM paths.

**Pipeline mapping:** A real pass/fail separation was found and MiniMax rated it high-signal.

## Differentiating test

- Test: `model_fields.test_promises.PromiseTest.test_UUIDField`
- Candidate that passed: **candidate_b**
- Specification gap tested: UUIDField.get_prep_value() should normalize accepted UUID representations, including lazy Promise values, into uuid.UUID objects. Fixing conversion only inside GenericForeignKey leaves this general field-preparation contract unsatisfied.
- Input: Pass UUIDField.get_prep_value() a Django lazy Promise that evaluates to the canonical string "550e8400-e29b-41d4-a716-446655440000".
- Expected behavior: The result equals uuid.UUID("550e8400-e29b-41d4-a716-446655440000"), rather than remaining a string or lazy proxy.

## MiniMax assessment

- Rating: **high_signal**
- Confidence: **0.95**
- Summary: The generated test validates a fundamental field contract: UUIDField.get_prep_value() should normalize input values (including lazy Promise objects) into proper uuid.UUID instances. Candidate B correctly implements this by overriding get_prep_value to call to_python, while candidate A only special-cases the GFK prefetch path without fixing the underlying field behavior. The test exposes that candidate A leaves the general field contract unsatisfied.

## Interpretation boundary

Agreement is measured against a human-produced reference review, not an objective ground truth. This note summarizes recorded evidence and does not expose private model chain-of-thought.
