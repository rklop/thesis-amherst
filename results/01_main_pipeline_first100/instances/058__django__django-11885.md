# 058 — django__django-11885

## Comparison

- Manual reference: **Very different**
- Pipeline-derived bucket: **Very different**
- Exact agreement: **yes**
- Ordinal distance: **0**

## Why each side classified it this way

**Manual review:** The candidate reverse-engineers and combines SQL late; gold changes deletion planning and batching before SQL is built.

**Pipeline mapping:** A real pass/fail separation was found and MiniMax rated it high-signal.

## Differentiating test

- Test: `test_fast_delete_combined_relationships_respects_batch_size`
- Candidate that passed: **candidate_b**
- Specification gap tested: Combining cascade fast-delete predicates must preserve backend parameter-limit batching. Queries for different relationship columns may be OR-combined within a batch, but batches must not all be collapsed into one oversized DELETE.
- Input: Create one Origin, 500 Referrer rows, and 500 SecondReferrer rows where both cascading foreign keys reference the same Referrer. Delete the Origin. On SQLite, each relationship separately fits one 500-object batch, while a combined two-column predicate is limited to 499 objects per batch.
- Expected behavior: Deletion returns a total of 1001 deleted rows, no SecondReferrer rows remain, and the SecondReferrer table is deleted through exactly two backend-sized DELETE statements on SQLite.

## MiniMax assessment

- Rating: **high_signal**
- Confidence: **0.95**
- Summary: The test validates that combined fast-delete queries must preserve backend parameter-limit batching, a critical invariant of the deletion system. Candidate A fails by collapsing all predicates into one oversized DELETE statement (1 query instead of 2), violating batching limits. Candidate B correctly groups relationship fields before calculating batch size, producing the expected number of safe combined DELETE statements (2 batches). The test targets genuine specification requirements derived from the issue's goal of combining queries while maintaining existing system invariants.

## Interpretation boundary

Agreement is measured against a human-produced reference review, not an objective ground truth. This note summarizes recorded evidence and does not expose private model chain-of-thought.
