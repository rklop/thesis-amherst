Judge the quality of a validated differentiating test for a
SWE-Bench task. Return exactly one JSON object and no Markdown.

Allowed ratings:

- `high_signal`: the test exercises intended public behavior, has a defensible
  oracle, reveals a meaningful missing specification, and is reasonably
  general/minimal rather than tailored to implementation internals.
- `low_signal`: the split is real but primarily reflects brittle internals,
  incidental formatting, undefined behavior, a contrived exploit, unrelated
  regressions, or an oracle not supported by the issue/API contract.
- `ambiguous`: the available evidence does not justify either quality judgment
  without a human deciding an underspecified semantic question.

Do not rate a test high merely because one candidate passes and one fails.
Assess whether the expected output follows from the issue and established API
semantics. Explicitly identify which candidate passed and whether that winner
appears more specification-conformant.

Required JSON keys: `rating`, `confidence`, `summary`, `specification_signal`,
`oracle_quality`, `concerns`, `human_review_questions`. `rating` must be one of
the three allowed values. `confidence` must be a number from 0 to 1. The last
three keys may contain arrays of strings.

<issue_statement>
Reduce Add/RemoveIndex migration operations.
Description
	
We should reduce AddIndex/RemoveIndex operations when optimizing migration operations.

</issue_statement>

<original_test_patch>
diff --git a/tests/migrations/test_optimizer.py b/tests/migrations/test_optimizer.py
--- a/tests/migrations/test_optimizer.py
+++ b/tests/migrations/test_optimizer.py
@@ -1158,3 +1158,17 @@ def test_rename_index(self):
                 ),
             ]
         )
+
+    def test_add_remove_index(self):
+        self.assertOptimizesTo(
+            [
+                migrations.AddIndex(
+                    "Pony",
+                    models.Index(
+                        fields=["weight", "pink"], name="idx_pony_weight_pink"
+                    ),
+                ),
+                migrations.RemoveIndex("Pony", "idx_pony_weight_pink"),
+            ],
+            [],
+        )

</original_test_patch>

<candidate_a_patch>
diff --git a/django/db/migrations/operations/models.py b/django/db/migrations/operations/models.py
index 967a91fdc8..03636540d7 100644
--- a/django/db/migrations/operations/models.py
+++ b/django/db/migrations/operations/models.py
@@ -861,6 +861,15 @@ class AddIndex(IndexOperation):
     def migration_name_fragment(self):
         return "%s_%s" % (self.model_name_lower, self.index.name.lower())
 
+    def reduce(self, operation, app_label):
+        if (
+            isinstance(operation, RemoveIndex)
+            and self.model_name_lower == operation.model_name_lower
+            and self.index.name == operation.name
+        ):
+            return []
+        return super().reduce(operation, app_label)
+
 
 class RemoveIndex(IndexOperation):
     """Remove an index from a model."""

</candidate_a_patch>

<candidate_b_patch>
diff --git a/django/db/migrations/operations/models.py b/django/db/migrations/operations/models.py
--- a/django/db/migrations/operations/models.py
+++ b/django/db/migrations/operations/models.py
@@ -861,6 +861,11 @@ def describe(self):
     def migration_name_fragment(self):
         return "%s_%s" % (self.model_name_lower, self.index.name.lower())
 
+    def reduce(self, operation, app_label):
+        if isinstance(operation, RemoveIndex) and self.index.name == operation.name:
+            return []
+        return super().reduce(operation, app_label)
+
 
 class RemoveIndex(IndexOperation):
     """Remove an index from a model."""

</candidate_b_patch>

<generated_test_proposal>
{
  "test_name": "test_add_remove_index_different_models",
  "specification_gap": "Django requires index names to be unique across models (models.E030), so AddIndex/RemoveIndex reduction should use the globally unique index name rather than additionally requiring identical model-name strings.",
  "input_description": "Pass MigrationOptimizer an AddIndex for Pony and a subsequent RemoveIndex for Horse, both using the exact index name shared_idx.",
  "expected_output": "The optimizer returns an empty operation list because the globally identified index is added and then removed.",
  "expected_passing_candidate": "candidate_b",
  "why_interesting": "This boundary case exposes candidate A's extra model-name guard. Candidate B follows the repository's cross-model index-name uniqueness invariant and removes the inverse pair.",
  "test_patch": "diff --git a/tests/migrations/test_optimizer.py b/tests/migrations/test_optimizer.py\n--- a/tests/migrations/test_optimizer.py\n+++ b/tests/migrations/test_optimizer.py\n@@ -1119,6 +1119,19 @@ class OptimizerTests(SimpleTestCase):\n                     \"Phou\", [(\"name\", models.CharField(max_length=255))]\n                 ),\n             ],\n         )\n \n+    def test_add_remove_index_different_models(self):\n+        # Index names are unique across models.\n+        index_name = \"shared_idx\"\n+        self.assertOptimizesTo(\n+            [\n+                migrations.AddIndex(\n+                    \"Pony\", models.Index(fields=[\"weight\"], name=index_name)\n+                ),\n+                migrations.RemoveIndex(\"Horse\", index_name),\n+            ],\n+            [],\n+        )\n+\n     def test_rename_index(self):\n         self.assertOptimizesTo(\n             [\n",
  "test_command": "cd /testbed && PYTHONPATH=. python tests/runtests.py migrations.test_optimizer.OptimizerTests.test_add_remove_index_different_models"
}
</generated_test_proposal>

<validated_execution>
## candidate_a
returncode=1 passed=False test_ran=True
log=02_execution/attempt_01/candidate_a.log
```
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
Testing against Django installed in '/testbed/django' with up to 24 processes
Found 1 test(s).
System check identified no issues (0 silenced).
F
======================================================================
FAIL: test_add_remove_index_different_models (migrations.test_optimizer.OptimizerTests.test_add_remove_index_different_models)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/testbed/tests/migrations/test_optimizer.py", line 1127, in test_add_remove_index_different_models
    self.assertOptimizesTo(
  File "/testbed/tests/migrations/test_optimizer.py", line 31, in assertOptimizesTo
    self.assertEqual(expected, result)
AssertionError: Lists differ: [] != ["migrations.AddIndex(\n    model_name='Po[146 chars]\n)"]

Second list contains 2 additional elements.
First extra element 0:
"migrations.AddIndex(\n    model_name='Pony',\n    index=models.Index(fields=['weight'], name='shared_idx'),\n)"

- []
+ ['migrations.AddIndex(\n'
+  "    model_name='Pony',\n"
+  "    index=models.Index(fields=['weight'], name='shared_idx'),\n"
+  ')',
+  "migrations.RemoveIndex(\n    model_name='Horse',\n    name='shared_idx',\n)"]

----------------------------------------------------------------------
Ran 1 test in 0.005s

FAILED (failures=1)
[pipeline] test_exit_code=1

```

## candidate_b
returncode=0 passed=True test_ran=True
log=02_execution/attempt_01/candidate_b.log
```
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
Testing against Django installed in '/testbed/django' with up to 24 processes
Found 1 test(s).
System check identified no issues (0 silenced).
.
----------------------------------------------------------------------
Ran 1 test in 0.000s

OK
[pipeline] test_exit_code=0

```
</validated_execution>
