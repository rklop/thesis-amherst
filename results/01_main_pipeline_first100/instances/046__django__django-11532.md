# 046 — django__django-11532

## Comparison

- Manual reference: **Not different**
- Pipeline-derived bucket: **Very different**
- Exact agreement: **no**
- Ordinal distance: **2**

## Why each side classified it this way

**Manual review:** At the failing Message-ID path the implementation is the same; gold's extra files consolidate existing conversions.

**Pipeline mapping:** A real pass/fail separation was found and MiniMax rated it high-signal.

## Differentiating test

- Test: `test_ascii_fqdn_is_not_idna_validated`
- Candidate that passed: **candidate_a**
- Specification gap tested: IDNA conversion should fix non-ASCII hostnames without newly validating or rejecting hostnames that are already ASCII. CachedDnsName previously returned every ASCII socket.getfqdn() result unchanged.
- Input: Mock socket.getfqdn() to return a single 64-character ASCII label and call CachedDnsName.get_fqdn() on a fresh cache instance.
- Expected behavior: The exact 64-character hostname is returned unchanged without raising an exception.

## MiniMax assessment

- Rating: **high_signal**
- Confidence: **0.95**
- Summary: The test successfully differentiates between two legitimate implementation approaches for handling non-ASCII hostnames in email Message-ID headers. Candidate A preserves ASCII hostnames unchanged (original behavior), while Candidate B unconditionally applies IDNA encoding which breaks valid ASCII hostnames exceeding 63 characters. The test correctly identifies this behavioral difference and has a defensible oracle based on the principle that the fix should only convert non-ASCII domains, not alter ASCII ones. Candidate A passes and appears more specification-conformant as it maintains backward compatibility.

## Severe-disagreement adjudication

- Category: **Questionable or out-of-domain test**
- Assessment: The differentiator uses a 64-character DNS label, which exceeds the DNS per-label limit, so its backward-compatibility oracle is debatable.

## Interpretation boundary

Agreement is measured against a human-produced reference review, not an objective ground truth. This note summarizes recorded evidence and does not expose private model chain-of-thought.
