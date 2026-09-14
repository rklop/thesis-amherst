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
Query.resolve_lookup_value coerces value of type list to tuple
Description
	
Changes introduced in #30687 cause an input value list to be coerced to tuple breaking exact value queries. This affects ORM field types that are dependent on matching input types such as PickledField.
The expected iterable return type should match input iterable type.

</issue_statement>

<original_test_patch>
diff --git a/tests/queries/test_query.py b/tests/queries/test_query.py
--- a/tests/queries/test_query.py
+++ b/tests/queries/test_query.py
@@ -113,3 +113,10 @@ def test_clone_select_related(self):
         clone = query.clone()
         clone.add_select_related(['note', 'creator__extra'])
         self.assertEqual(query.select_related, {'creator': {}})
+
+    def test_iterable_lookup_value(self):
+        query = Query(Item)
+        where = query.build_where(Q(name=['a', 'b']))
+        name_exact = where.children[0]
+        self.assertIsInstance(name_exact, Exact)
+        self.assertEqual(name_exact.rhs, "['a', 'b']")

</original_test_patch>

<candidate_a_patch>
diff --git a/django/db/models/sql/query.py b/django/db/models/sql/query.py
index ea51912e8b..90475d6fb9 100644
--- a/django/db/models/sql/query.py
+++ b/django/db/models/sql/query.py
@@ -1073,7 +1073,7 @@ class Query(BaseExpression):
                         ))
                 else:
                     resolved_values.append(sub_value)
-            value = tuple(resolved_values)
+            value = type(value)(resolved_values)
         return value
 
     def solve_lookup_type(self, lookup):

</candidate_a_patch>

<candidate_b_patch>
diff --git a/django/db/models/sql/query.py b/django/db/models/sql/query.py
--- a/django/db/models/sql/query.py
+++ b/django/db/models/sql/query.py
@@ -1059,21 +1059,10 @@ def resolve_lookup_value(self, value, can_reuse, allow_joins, simple_col):
         elif isinstance(value, (list, tuple)):
             # The items of the iterable may be expressions and therefore need
             # to be resolved independently.
-            resolved_values = []
-            for sub_value in value:
-                if hasattr(sub_value, 'resolve_expression'):
-                    if isinstance(sub_value, F):
-                        resolved_values.append(sub_value.resolve_expression(
-                            self, reuse=can_reuse, allow_joins=allow_joins,
-                            simple_col=simple_col,
-                        ))
-                    else:
-                        resolved_values.append(sub_value.resolve_expression(
-                            self, reuse=can_reuse, allow_joins=allow_joins,
-                        ))
-                else:
-                    resolved_values.append(sub_value)
-            value = tuple(resolved_values)
+            return type(value)(
+                self.resolve_lookup_value(sub_value, can_reuse, allow_joins, simple_col)
+                for sub_value in value
+            )
         return value
 
     def solve_lookup_type(self, lookup):

</candidate_b_patch>

<generated_test_proposal>
{
  "test_name": "test_nested_iterable_lookup_value",
  "specification_gap": "Iterable lookup values must resolve ORM expressions at every nested list/tuple level while preserving each container's input type. The existing test covers only a flat list, so a shallow fix is indistinguishable from recursive resolution.",
  "input_description": "Pass Query.resolve_lookup_value() a list containing a top-level F('name') and a nested one-element tuple containing F('created').",
  "expected_output": "The result remains a list, its nested container remains a tuple, and both F() references are resolved to SimpleCol expressions.",
  "expected_passing_candidate": "candidate_b",
  "why_interesting": "Candidate A preserves the outer list and resolves its direct F() child, but leaves the F() inside the tuple unresolved. Candidate B recursively applies expression resolution and type preservation, satisfying the invariant for structured lookup values.",
  "test_patch": "diff --git a/tests/queries/test_query.py b/tests/queries/test_query.py\n--- a/tests/queries/test_query.py\n+++ b/tests/queries/test_query.py\n@@ -113,3 +113,14 @@ class TestQuery(SimpleTestCase):\n         clone = query.clone()\n         clone.add_select_related(['note', 'creator__extra'])\n         self.assertEqual(query.select_related, {'creator': {}})\n+\n+    def test_nested_iterable_lookup_value(self):\n+        query = Query(Item)\n+        value = [F('name'), (F('created'),)]\n+        resolved = query.resolve_lookup_value(\n+            value, can_reuse=set(), allow_joins=False, simple_col=True,\n+        )\n+        self.assertIsInstance(resolved, list)\n+        self.assertIsInstance(resolved[0], SimpleCol)\n+        self.assertIsInstance(resolved[1], tuple)\n+        self.assertIsInstance(resolved[1][0], SimpleCol)\n",
  "test_command": "cd /testbed && ./tests/runtests.py queries.test_query.TestQuery.test_nested_iterable_lookup_value"
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
System check identified no issues (1 silenced).
F
======================================================================
FAIL: test_nested_iterable_lookup_value (queries.test_query.TestQuery)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/testbed/tests/queries/test_query.py", line 126, in test_nested_iterable_lookup_value
    self.assertIsInstance(resolved[1][0], SimpleCol)
AssertionError: F(created) is not an instance of <class 'django.db.models.expressions.SimpleCol'>

----------------------------------------------------------------------
Ran 1 test in 0.000s

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
.
----------------------------------------------------------------------
Ran 1 test in 0.000s

OK
Testing against Django installed in '/testbed/django' with up to 24 processes
System check identified no issues (1 silenced).
[pipeline] test_exit_code=0

```
</validated_execution>
