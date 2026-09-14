# test_or_with_custom_query_join_override

- **Instance:** `django__django-15128`
- **Test ID:** `django__django-15128--8a4d22b44c420b66`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 2

## What it checks

Alias-collision handling for QuerySet OR combinations must preserve the established Query.join(join, reuse=None) override contract. A custom Query subclass using that signature should still support combinations whose left and right sides create offset, overlapping sequential aliases.

## Expected behavior

Evaluating the combined queryset succeeds and returns each of the four classrooms exactly once. candidate_a instead passes a new reuse_with_aliases keyword to the existing join override and raises TypeError during the OR operation.

## Test command

`python tests/runtests.py queries.tests.QuerySetBitwiseOperationTests.test_or_with_custom_query_join_override`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
