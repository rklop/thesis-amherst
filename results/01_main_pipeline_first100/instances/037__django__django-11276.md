# 037 — django__django-11276

## Comparison

- Manual reference: **Not different**
- Pipeline-derived bucket: **Very different**
- Exact agreement: **no**
- Ordinal distance: **2**

## Why each side classified it this way

**Manual review:** The requested escape implementation is the same; gold includes an adjacent standard-library cleanup.

**Pipeline mapping:** A real pass/fail separation was found and MiniMax rated it high-signal.

## Differentiating test

- Test: `test_urlize_escaped_apostrophe`
- Candidate that passed: **candidate_b**
- Specification gap tested: Switching escape() to html.escape() changes apostrophes from &#39; to &#x27;. urlize() promises to handle HTML-escaped URLs, so it must recognize the new escape() output when constructing href values. Candidate A leaves urlize()'s decoder unable to decode &#x27;, causing double escaping; candidate B updates it.
- Input: Pass the URL "http://example.com/it's/" through django.utils.html.escape(), then pass that escaped SafeString to django.utils.html.urlize().
- Expected behavior: urlize() returns exactly <a href="http://example.com/it&#x27;s/">http://example.com/it&#x27;s/</a>. In particular, the href contains &#x27;, not the double-escaped &amp;#x27; produced when urlize() fails to decode escape()'s output.

## MiniMax assessment

- Rating: **high_signal**
- Confidence: **0.85**
- Summary: The test exercises a legitimate composition of two public Django HTML utilities (escape() and urlize()) at the specific apostrophe-format boundary introduced by the requested change. Candidate A fails because it leaves urlize() unable to decode the new &#x27; format, resulting in visible double-escaping (&amp;#x27;). Candidate B correctly updates urlize() to use html.unescape() which handles both &#39; and &#x27;. The issue explicitly acknowledges the &#39;→&#x27; change as a backwards incompatible change, making this test a valid specification check.

## Severe-disagreement adjudication

- Category: **Strong new behavioral evidence**
- Assessment: Composing escape() and urlize() exposes a concrete compatibility difference caused by the requested apostrophe escape change.

## Interpretation boundary

Agreement is measured against a human-produced reference review, not an objective ground truth. This note summarizes recorded evidence and does not expose private model chain-of-thought.
