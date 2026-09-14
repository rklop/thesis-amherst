# 064 — django__django-12125

## Comparison

- Manual reference: **Not different**
- Pipeline-derived bucket: **Very different**
- Exact agreement: **no**
- Ordinal distance: **2**

## Why each side classified it this way

**Manual review:** The nested-class implementation is the same; the additional branch does not create a useful alternative.

**Pipeline mapping:** A real pass/fail separation was found and MiniMax rated it high-signal.

## Differentiating test

- Test: `test_serialize_nested_builtin_type`
- Candidate that passed: **candidate_a**
- Specification gap tested: The specification does not state whether qualified-name serialization applies when a nested class reports the public `builtins` module. Candidate A applies `__qualname__` uniformly, while candidate B special-cases builtins and emits only `__name__`, producing an unresolvable reference for a nested builtins type.
- Input: Create `MigrationTypes.Nested`, expose `MigrationTypes` through the builtins namespace, and set the nested class's public module and qualified name to `builtins` and `MigrationTypes.Nested`. Serialize the nested class through `MigrationWriter` and execute the generated expression.
- Expected behavior: The generated expression must evaluate to the identical `MigrationTypes.Nested` class. Candidate A emits `MigrationTypes.Nested` with no imports; candidate B emits `Nested`, which cannot be resolved.

## MiniMax assessment

- Rating: **high_signal**
- Confidence: **0.95**
- Summary: The test correctly distinguishes the two candidates based on a meaningful semantic difference: whether the serializer preserves the qualified name path for nested classes even when their public module is 'builtins'. Candidate A passes because it emits `MigrationTypes.Nested` which can resolve via the mocked builtins entry point, while candidate B emits just `Nested' which cannot be resolved. This reveals the correct behavior for handling nested classes that have a public module of 'builtins' but need their full qualified name preserved for proper resolution.

## Severe-disagreement adjudication

- Category: **Questionable or out-of-domain test**
- Assessment: The test monkeypatches an unusual nested type into builtins, which is far from the reported nested model-field scenario.

## Interpretation boundary

Agreement is measured against a human-produced reference review, not an objective ground truth. This note summarizes recorded evidence and does not expose private model chain-of-thought.
