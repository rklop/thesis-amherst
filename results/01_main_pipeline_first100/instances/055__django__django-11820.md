# 055 — django__django-11820

## Comparison

- Manual reference: **Very different**
- Pipeline-derived bucket: **Very different**
- Exact agreement: **yes**
- Ordinal distance: **0**

## Why each side classified it this way

**Manual review:** Gold preserves normal field traversal after alias resolution; the candidate introduces a separate path with different behavior for following segments.

**Pipeline mapping:** A real pass/fail separation was found and MiniMax rated it high-signal.

## Differentiating test

- Test: `test_ordering_rejects_lookup_after_registered_transform`
- Candidate that passed: **candidate_b**
- Specification gap tested: Meta.ordering validation must inspect the entire lookup path. Recognizing a registered transform must not cause validation to ignore later nonexistent components.
- Input: Define a model with a CharField and Meta.ordering = ('test__lower__missing',), while registering the public Lower transform on CharField.
- Expected behavior: Model.check() returns exactly one models.E015 Error naming 'test__lower__missing'. The 'lower' prefix is valid, but the trailing 'missing' lookup is not.

## MiniMax assessment

- Rating: **high_signal**
- Confidence: **0.95**
- Summary: The test correctly identifies that candidate_b properly validates the entire lookup path after a registered transform, while candidate_a stops validation after encountering a valid transform and incorrectly accepts nonexistent suffixes. The test uses Django's public Lower transform API and tests a legitimate validation requirement: after a valid transform, subsequent path components must still be validated. Candidate_b passes and demonstrates specification-conformant behavior.

## Interpretation boundary

Agreement is measured against a human-produced reference review, not an objective ground truth. This note summarizes recorded evidence and does not expose private model chain-of-thought.
