Act as a careful human software engineer reviewing a potential
negative specification test. Return exactly one JSON object and no Markdown.

The generated test separates two plausible patches, but the actual SWE-bench
gold patch executes and fails it. The question is not whether the assertion
should pass. The question is whether this failure reliably specifies behavior
that a final patch must NOT be required to implement. Extract only a narrow,
implementation-independent negative constraint supported by the issue and gold
evidence. Do not infer that every gold failure is useful.

Use this policy:
- `accept` only when the target test demonstrably ran, its failure is behavioral
  (not collection/setup/infrastructure), and the assertion captures a plausible,
  issue-relevant behavior that the gold evidence specifically declines to require.
- `reject` for infrastructure or collection failures, incidental output, private
  implementation coupling, unrelated behavior, unstable/undefined behavior, or
  when failure does not justify a reusable negative constraint.
- `human_review` only for a consequential ambiguity not resolvable from evidence.

eligible_failure_fingerprint=true. If false, do not accept.
An arbitrary nonzero exit is never sufficient.

Required JSON keys: `decision`, `confidence`, `negative_specification`,
`forbidden_behavior`, `gold_failure_interpretation`, `specification_value`,
`decision_rationale`, `concerns`, `human_review_questions`.

<issue_statement>
Optimize multiple AlterFooTogether operations into one
Description
	
Hi,
In #31503 we split the AlterFooTogether (AlterUniqueTogether and AlterIndexTogether) operations into two types of operations.
First, a migration will have operations to remove constraints, and then other operations adds the new constraints. This allows field alterations to work as expected during in between operations.
In some cases, this introduced two operations that can actually easily be reduced to one.
See for instance the test case: ​https://github.com/django/django/pull/14722/files#diff-506caa00017053ff8278de6efc2e59cc0c5cea22da9461482bdf16a9fc50af9eR1573-R1592
Example:
 operations = [
	 migrations.AlterUniqueTogether(
		 name='mymodel',
		 unique_together=set(),
	 ),
	 migrations.AlterIndexTogether(
		 name='mymodel',
		 index_together=set(),
	 ),
	 migrations.AlterUniqueTogether(
		 name='mymodel',
		 unique_together={("col",)},
	 ),
	 migrations.AlterIndexTogether(
		 name='mymodel',
		 index_together={("col",)},
	 ),
 ]
should be optimized to
 operations = [
	 migrations.AlterUniqueTogether(
		 name='mymodel',
		 unique_together={("col",)},
	 ),
	 migrations.AlterIndexTogether(
		 name='mymodel',
		 index_together={("col",)},
	 ),
 ]
So that we don't do two operations on each constraint, but only one.

</issue_statement>
<candidate_a_patch>
diff --git a/django/db/migrations/operations/models.py b/django/db/migrations/operations/models.py
index c120f5d32d..e23ac38b18 100644
--- a/django/db/migrations/operations/models.py
+++ b/django/db/migrations/operations/models.py
@@ -528,6 +528,16 @@ class AlterTogetherOptionOperation(ModelOptionOperation):
     def migration_name_fragment(self):
         return 'alter_%s_%s' % (self.name_lower, self.option_name)
 
+    def reduce(self, operation, app_label):
+        if isinstance(operation, AlterTogetherOptionOperation) and self.name_lower == operation.name_lower:
+            if self.__class__ is operation.__class__:
+                # Same class, same model - return the later operation
+                return [operation]
+            elif self.option_name != operation.option_name:
+                # Different option_name - allow to optimize through
+                return True
+        return super().reduce(operation, app_label)
+
 
 class AlterUniqueTogether(AlterTogetherOptionOperation):
     """

</candidate_a_patch>
<candidate_b_patch>
diff --git a/django/db/migrations/operations/models.py b/django/db/migrations/operations/models.py
index c120f5d32d..1790626ffc 100644
--- a/django/db/migrations/operations/models.py
+++ b/django/db/migrations/operations/models.py
@@ -470,6 +470,26 @@ class AlterModelTable(ModelOptionOperation):
 class AlterTogetherOptionOperation(ModelOptionOperation):
     option_name = None
 
+    def reduce(self, operation, app_label):
+        if (
+            isinstance(operation, AlterTogetherOptionOperation) and
+            self.option_name == operation.option_name and
+            self.name_lower == operation.name_lower and
+            # Only reduce if the operation class is the same or a subclass.
+            # This ensures a custom subclass isn't reduced to a built-in sibling.
+            isinstance(operation, self.__class__)
+        ):
+            return [operation]
+        # Allow passing through if the other operation has a different option_name
+        # (e.g., AlterUniqueTogether can pass through AlterIndexTogether)
+        elif (
+            isinstance(operation, AlterTogetherOptionOperation) and
+            self.option_name != operation.option_name and
+            self.name_lower == operation.name_lower
+        ):
+            return True
+        return super().reduce(operation, app_label)
+
     def __init__(self, name, option_value):
         if option_value:
             option_value = set(normalize_together(option_value))

</candidate_b_patch>
<generated_test_proposal>
{
  "test_name": "test_alter_together_different_options_custom_operation",
  "specification_gap": "For custom AlterTogetherOptionOperation subclasses, operation class equality must not override option identity. Two same-class operations targeting different together options are independent and must not absorb each other.",
  "input_description": "Pass MigrationOptimizer two instances of one reusable custom operation class for model Book. The first sets unique_together to {('title',)} and the second sets index_together to {('author',)}.",
  "expected_output": "The optimized operations expose option_name values ['unique_together', 'index_together'] in that order, showing that both independent changes were retained.",
  "expected_passing_candidate": "candidate_b",
  "why_interesting": "candidate_a checks exact class equality first and incorrectly replaces the unique_together operation with the later index_together operation. candidate_b checks the option names first and allows optimization to pass across different options without discarding either operation.",
  "test_patch": "diff --git a/tests/migrations/test_optimizer.py b/tests/migrations/test_optimizer.py\n--- a/tests/migrations/test_optimizer.py\n+++ b/tests/migrations/test_optimizer.py\n@@ -1,5 +1,6 @@\n from django.db import migrations, models\n from django.db.migrations import operations\n+from django.db.migrations.operations.models import AlterTogetherOptionOperation\n from django.db.migrations.optimizer import MigrationOptimizer\n from django.db.migrations.serializer import serializer_factory\n from django.test import SimpleTestCase\n@@ -209,7 +210,30 @@ class OptimizerTests(SimpleTestCase):\n     def test_alter_alter_index_model(self):\n         self._test_alter_alter_model(\n             migrations.AlterIndexTogether(\"Foo\", [[\"a\", \"b\"]]),\n             migrations.AlterIndexTogether(\"Foo\", [[\"a\", \"c\"]]),\n         )\n \n+    def test_alter_together_different_options_custom_operation(self):\n+        \"\"\"Different together options must remain independent.\"\"\"\n+        class CustomAlterTogetherOperation(AlterTogetherOptionOperation):\n+            def __init__(self, name, option_name, option_value):\n+                self.option_name = option_name\n+                super().__init__(name, option_value)\n+\n+        unique_operation = CustomAlterTogetherOperation(\n+            \"Book\", \"unique_together\", {(\"title\",)},\n+        )\n+        index_operation = CustomAlterTogetherOperation(\n+            \"Book\", \"index_together\", {(\"author\",)},\n+        )\n+\n+        optimized, _ = self.optimize(\n+            [unique_operation, index_operation],\n+            \"migrations\",\n+        )\n+        self.assertEqual(\n+            [operation.option_name for operation in optimized],\n+            [\"unique_together\", \"index_together\"],\n+        )\n+\n     def test_alter_alter_owrt_model(self):\n",
  "test_command": "cd /testbed && ./tests/runtests.py migrations.test_optimizer.OptimizerTests.test_alter_together_different_options_custom_operation"
}
</generated_test_proposal>
<candidate_execution_summary>
{
  "attempt": 1,
  "separates": true,
  "required_passing_candidate": "candidate_b",
  "meets_acceptance_rule": true,
  "results": [
    {
      "label": "candidate_a",
      "returncode": 1,
      "passed": false,
      "test_ran": true,
      "duration_seconds": 1.352,
      "log_path": "02_execution/attempt_01/candidate_a.log"
    },
    {
      "label": "candidate_b",
      "returncode": 0,
      "passed": true,
      "test_ran": true,
      "duration_seconds": 1.335,
      "log_path": "02_execution/attempt_01/candidate_b.log"
    }
  ]
}
</candidate_execution_summary>
<candidate_a_execution_log>
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
F
======================================================================
FAIL: test_alter_together_different_options_custom_operation (migrations.test_optimizer.OptimizerTests)
Different together options must remain independent.
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/testbed/tests/migrations/test_optimizer.py", line 234, in test_alter_together_different_options_custom_operation
    self.assertEqual(
AssertionError: Lists differ: ['index_together'] != ['unique_together', 'index_together']

First differing element 0:
'index_together'
'unique_together'

Second list contains 1 additional elements.
First extra element 1:
'index_together'

- ['index_together']
+ ['unique_together', 'index_together']

----------------------------------------------------------------------
Ran 1 test in 0.000s

FAILED (failures=1)
Testing against Django installed in '/testbed/django' with up to 24 processes
Found 1 test(s).
System check identified no issues (0 silenced).
[pipeline] test_exit_code=1

</candidate_a_execution_log>
<candidate_b_execution_log>
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
.
----------------------------------------------------------------------
Ran 1 test in 0.000s

OK
Testing against Django installed in '/testbed/django' with up to 24 processes
Found 1 test(s).
System check identified no issues (0 silenced).
[pipeline] test_exit_code=0

</candidate_b_execution_log>
<official_gold_patch>
diff --git a/django/db/migrations/operations/models.py b/django/db/migrations/operations/models.py
--- a/django/db/migrations/operations/models.py
+++ b/django/db/migrations/operations/models.py
@@ -34,9 +34,12 @@ def references_model(self, name, app_label):
     def reduce(self, operation, app_label):
         return (
             super().reduce(operation, app_label) or
-            not operation.references_model(self.name, app_label)
+            self.can_reduce_through(operation, app_label)
         )
 
+    def can_reduce_through(self, operation, app_label):
+        return not operation.references_model(self.name, app_label)
+
 
 class CreateModel(ModelOperation):
     """Create a model's table."""
@@ -528,6 +531,14 @@ def describe(self):
     def migration_name_fragment(self):
         return 'alter_%s_%s' % (self.name_lower, self.option_name)
 
+    def can_reduce_through(self, operation, app_label):
+        return (
+            super().can_reduce_through(operation, app_label) or (
+                isinstance(operation, AlterTogetherOptionOperation) and
+                type(operation) is not type(self)
+            )
+        )
+
 
 class AlterUniqueTogether(AlterTogetherOptionOperation):
     """

</official_gold_patch>
<gold_execution_summary>
{
  "label": "gold",
  "returncode": 1,
  "passed": false,
  "test_ran": true,
  "duration_seconds": 1.355,
  "log_path": "gold_execution/gold.log"
}
</gold_execution_summary>
<gold_failure_contract>
{
  "kind": "behavioral_test_failure",
  "required_substrings": [
    "test_alter_together_different_options_custom_operation"
  ],
  "required_any_substrings": [
    "AssertionError",
    "FAILED",
    "FAIL:"
  ],
  "forbidden_substrings": [
    "collected 0 items",
    "no tests ran",
    "ERROR collecting",
    "command not found",
    "No such file or directory",
    "Could not find a version that satisfies the requirement",
    "Temporary failure in name resolution"
  ],
  "reference_returncode": 1,
  "reference_log_sha256": "9bb94b88a1358c08167b24503c34b0f0af41ebd6a51a0ad45a9eef73d48b2e01"
}
</gold_failure_contract>
<gold_execution_log>
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
F
======================================================================
FAIL: test_alter_together_different_options_custom_operation (migrations.test_optimizer.OptimizerTests)
Different together options must remain independent.
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/testbed/tests/migrations/test_optimizer.py", line 234, in test_alter_together_different_options_custom_operation
    self.assertEqual(
AssertionError: Lists differ: ['index_together'] != ['unique_together', 'index_together']

First differing element 0:
'index_together'
'unique_together'

Second list contains 1 additional elements.
First extra element 1:
'index_together'

- ['index_together']
+ ['unique_together', 'index_together']

----------------------------------------------------------------------
Ran 1 test in 0.000s

FAILED (failures=1)
Testing against Django installed in '/testbed/django' with up to 24 processes
Found 1 test(s).
System check identified no issues (0 silenced).
[pipeline] test_exit_code=1

</gold_execution_log>
