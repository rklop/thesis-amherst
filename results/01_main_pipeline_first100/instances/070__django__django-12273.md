# 070 — django__django-12273

## Comparison

- Manual reference: **Very different**
- Pipeline-derived bucket: **Very different**
- Exact agreement: **yes**
- Ordinal distance: **0**

## Why each side classified it this way

**Manual review:** The candidate includes the core synchronization but adds save-path behavior and extra parent assignments outside gold's scope.

**Pipeline mapping:** A real pass/fail separation was found and MiniMax rated it high-signal.

## Differentiating test

- Test: `test_save_with_explicit_new_pk`
- Candidate that passed: **candidate_b**
- Specification gap tested: Assigning a valid non-None value through a child model's public pk property must propagate that value to the target primary-key field of a non-primary parent link. The existing tests cover only None and manually clear link fields, so they don't distinguish updating the parent's target field from incorrectly updating the child link column.
- Input: Create a Profile whose own primary key is distinct from its inherited User identity, assign an unused integer to profile.pk, change its username, and save it.
- Expected behavior: The save succeeds and creates a second Profile/User pair. The original profile still has username 'john'; the copied profile has the explicit new primary key, username 'bill', and a User parent with that same new identity.

## MiniMax assessment

- Rating: **high_signal**
- Confidence: **0.9**
- Summary: The test exercises a legitimate public API contract: setting pk on a child model instance should propagate the value to parent link fields to enable creating a new row on save. The test's oracle correctly verifies both the original and new records exist, and candidate_b's passing indicates it implements the correct semantics by propagating the pk to the parent id field rather than just the child link column.

## Interpretation boundary

Agreement is measured against a human-produced reference review, not an objective ground truth. This note summarizes recorded evidence and does not expose private model chain-of-thought.
