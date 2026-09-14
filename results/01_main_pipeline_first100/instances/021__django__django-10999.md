# 021 — django__django-10999

## Comparison

- Manual reference: **Very different**
- Pipeline-derived bucket: **Very different**
- Exact agreement: **yes**
- Ordinal distance: **0**

## Why each side classified it this way

**Manual review:** They recognize different input languages; gold expresses a single-sign grammar while the candidate repairs component signs after parsing.

**Pipeline mapping:** A real pass/fail separation was found and MiniMax rated it high-signal.

## Differentiating test

- Test: `test_negative_sign_after_colon`
- Candidate that passed: **candidate_b**
- Specification gap tested: Only a leading minus sign may negate a standard duration. Candidate A also accepts a minus sign on an individual component after a colon.
- Input: Call parse_duration('00:01:-01'), where the seconds component has an internal sign.
- Expected behavior: parse_duration() returns None because the input is not a valid standard, ISO 8601, or PostgreSQL duration. Candidate A instead returns timedelta(seconds=59).

## MiniMax assessment

- Rating: **high_signal**
- Confidence: **0.9**
- Summary: The test correctly identifies a semantic difference between candidates. Candidate A's logic to propagate leading negative signs to components also inadvertently accepts individually-signed components like '00:01:-01', whereas Candidate B's refactored regex captures a single leading sign at the start and rejects component-internal signs. The test input '00:01:-01' has an internal sign on seconds that should not be valid under the intended semantics—negative durations should have a single leading minus applied to the entire duration, not per-component signs. Candidate B correctly returns None for this case, aligning with the original test patch expectation that '-01:-01' (both minutes and seconds with signs) returns None.

## Interpretation boundary

Agreement is measured against a human-produced reference review, not an objective ground truth. This note summarizes recorded evidence and does not expose private model chain-of-thought.
