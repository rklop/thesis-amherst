# 057 — django__django-11880

## Comparison

- Manual reference: **Somewhat different**
- Pipeline-derived bucket: **Very different**
- Exact agreement: **no**
- Ordinal distance: **1**

## Why each side classified it this way

**Manual review:** Both isolate the dictionary, but custom nested or mutable message values are copied differently.

**Pipeline mapping:** A real pass/fail separation was found and MiniMax rated it high-signal.

## Differentiating test

- Test: `test_error_message_values_independence`
- Candidate that passed: **candidate_a**
- Specification gap tested: The existing test requires only a distinct error_messages mapping, so a shallow dictionary copy passes. It doesn't specify whether mutable message values accepted by Django's ValidationError path are also isolated between form instances.
- Input: Define a Form whose required-field error message is a two-item list, instantiate two bound forms, and append a third message through only the first form's field.
- Expected behavior: The first form reports ['First error.', 'Second error.', 'Third error.'], while the second still reports ['First error.', 'Second error.'].

## MiniMax assessment

- Rating: **high_signal**
- Confidence: **0.95**
- Summary: This is an excellent differentiating test that exercises the core semantic issue: error_messages values that are mutable (like lists) must be deep-copied to ensure form instance independence. Candidate A passes because it uses copy.deepcopy for recursive copying, while candidate B fails because .copy() only shallow-copies the dictionary, causing the list value to be shared between form instances. The test legitimately uses Django's supported feature of list-valued error messages and verifies the expected isolation behavior described in the issue.

## Interpretation boundary

Agreement is measured against a human-produced reference review, not an objective ground truth. This note summarizes recorded evidence and does not expose private model chain-of-thought.
