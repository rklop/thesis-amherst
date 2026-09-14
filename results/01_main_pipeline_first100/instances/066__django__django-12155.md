# 066 — django__django-12155

## Comparison

- Manual reference: **Somewhat different**
- Pipeline-derived bucket: **Very different**
- Exact agreement: **no**
- Ordinal distance: **1**

## Why each side classified it this way

**Manual review:** The intended indentation semantics overlap, but gold delegates all edge cases to the standard implementation across two paths.

**Pipeline mapping:** A real pass/fail separation was found and MiniMax rated it high-signal.

## Differentiating test

- Test: `test_parse_docstring_preserves_rst_escaped_space`
- Candidate that passed: **candidate_b**
- Specification gap tested: Docstring indentation cleanup must not strip meaningful interior content. In reStructuredText, a backslash followed by a trailing space is an escape sequence, but candidate A applies rstrip() to every line and removes that space. Candidate B uses inspect.cleandoc(), which removes indentation while preserving it.
- Input: Pass parse_docstring() the first-line docstring "Summary.\n\n    first\\ \n    second", whose body contains a reStructuredText escaped space immediately before a newline.
- Expected behavior: parse_docstring() returns exactly ('Summary.', 'first\\ \nsecond', {}), retaining the space after the backslash while removing the body's four-space indentation.

## MiniMax assessment

- Rating: **high_signal**
- Confidence: **0.85**
- Summary: The test successfully differentiates between candidates by exposing a semantic difference in docstring content preservation. Candidate A strips trailing whitespace from every line (including the escaped space), while Candidate B using inspect.cleandoc() preserves it. Candidate B correctly passes the test, indicating it handles RST escaped spaces appropriately for admindocs rendering.

## Interpretation boundary

Agreement is measured against a human-produced reference review, not an objective ground truth. This note summarizes recorded evidence and does not expose private model chain-of-thought.
