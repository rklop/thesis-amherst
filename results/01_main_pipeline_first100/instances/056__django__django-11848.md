# 056 — django__django-11848

## Comparison

- Manual reference: **Very different**
- Pipeline-derived bucket: **Very different**
- Exact agreement: **yes**
- Ordinal distance: **0**

## Why each side classified it this way

**Manual review:** The candidate is correct in the current century but hard-codes behavior that fails after a century boundary.

**Pipeline mapping:** A real pass/fail separation was found and MiniMax rated it high-signal.

## Differentiating test

- Test: `test_parsing_rfc850_at_next_century_boundary`
- Candidate that passed: **candidate_b**
- Specification gap tested: Two-digit RFC 850 years must be interpreted relative to the current century, including after a century rollover. A year exactly 50 years in the future must remain in the future because only dates more than 50 years ahead are rolled back.
- Input: Freeze the current UTC time at 2100-11-06 08:49:37 and parse the RFC 850 date "Sunday, 06-Nov-50 08:49:37 GMT".
- Expected behavior: parse_http_date() returns an epoch timestamp that converts to 2150-11-06 08:49:37 UTC. Candidate A instead resolves the year against a hard-coded 2000 base and produces 2050.

## MiniMax assessment

- Rating: **high_signal**
- Confidence: **0.95**
- Summary: The test correctly exercises the RFC 7231 specification for RFC 850 two-digit year parsing. It validates that the year calculation must be relative to the current century (not hard-coded to 2000), specifically testing the 50-year boundary at a century rollover (year 2100). Candidate B correctly computes the current century and applies the >50 year rule, while Candidate A incorrectly adds 2000 as a base, failing when the current year is 2100. The test winner (candidate_b) is specification-conformant.

## Interpretation boundary

Agreement is measured against a human-produced reference review, not an objective ground truth. This note summarizes recorded evidence and does not expose private model chain-of-thought.
