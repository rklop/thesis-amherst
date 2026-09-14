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
Composed queries cannot change the list of columns with values()/values_list().
Description
	
Composed queries cannot change the list of columns when values()/values_list() is evaluated multiple times, e.g.
>>> ReservedName.objects.create(name='a', order=2)
>>> qs1 = ReservedName.objects.all()
>>> print(qs1.union(qs1).values_list('name', 'order').get())
('a', 2)
>>> print(qs1.union(qs1).values_list('order').get())
('a', 2)
(see ​compiler.py#L428-L433).

</issue_statement>

<original_test_patch>
diff --git a/tests/queries/test_qs_combinators.py b/tests/queries/test_qs_combinators.py
--- a/tests/queries/test_qs_combinators.py
+++ b/tests/queries/test_qs_combinators.py
@@ -123,6 +123,9 @@ def test_union_with_values(self):
         self.assertEqual(reserved_name['order'], 2)
         reserved_name = qs1.union(qs1).values_list('name', 'order', 'id').get()
         self.assertEqual(reserved_name[:2], ('a', 2))
+        # List of columns can be changed.
+        reserved_name = qs1.union(qs1).values_list('order').get()
+        self.assertEqual(reserved_name, (2,))
 
     def test_union_with_two_annotated_values_list(self):
         qs1 = Number.objects.filter(num=1).annotate(

</original_test_patch>

<candidate_a_patch>
diff --git a/django/db/models/sql/compiler.py b/django/db/models/sql/compiler.py
index a44adfc760..0964970693 100644
--- a/django/db/models/sql/compiler.py
+++ b/django/db/models/sql/compiler.py
@@ -426,6 +426,11 @@ class SQLCompiler:
                 # must have the same columns list. Set the selects defined on
                 # the query on all combined queries, if not already set.
                 if not compiler.query.values_select and self.query.values_select:
+                    # If the combined query is the same object as the original
+                    # query (e.g., qs.union(qs)), we need to clone it to avoid
+                    # modifying the original query's values_select.
+                    if compiler.query is self.query.combined_queries[0]:
+                        compiler.query = compiler.query.clone()
                     compiler.query.set_values((
                         *self.query.extra_select,
                         *self.query.values_select,

</candidate_a_patch>

<candidate_b_patch>
diff --git a/django/db/models/sql/compiler.py b/django/db/models/sql/compiler.py
--- a/django/db/models/sql/compiler.py
+++ b/django/db/models/sql/compiler.py
@@ -426,6 +426,7 @@ def get_combinator_sql(self, combinator, all):
                 # must have the same columns list. Set the selects defined on
                 # the query on all combined queries, if not already set.
                 if not compiler.query.values_select and self.query.values_select:
+                    compiler.query = compiler.query.clone()
                     compiler.query.set_values((
                         *self.query.extra_select,
                         *self.query.values_select,

</candidate_b_patch>

<generated_test_proposal>
{
  "test_name": "test_union_projection_reuse_with_distinct_operands",
  "specification_gap": "Re-projecting a composed queryset must not retain a previous projection on a distinct non-first operand. The existing test only covers reusing the same QuerySet object as both operands.",
  "input_description": "Create one ReservedName row, union two separately constructed ReservedName.objects.all() querysets, evaluate a two-column values_list(), then evaluate a one-column values_list() on the same compound queryset.",
  "expected_output": "The first evaluation returns ('a', 2), and the second returns (2,) without a database column-count error.",
  "expected_passing_candidate": "candidate_b",
  "why_interesting": "Candidate A clones only branches identical to combined_queries[0], so the first evaluation mutates the distinct second operand. Candidate B clones every branch before propagating the projection. This tests public QuerySet reuse and changing projection arity without asserting compiler internals.",
  "test_patch": "diff --git a/tests/queries/test_qs_combinators.py b/tests/queries/test_qs_combinators.py\n--- a/tests/queries/test_qs_combinators.py\n+++ b/tests/queries/test_qs_combinators.py\n@@ -125,6 +125,15 @@ class QuerySetSetOperationTests(TestCase):\n         reserved_name = qs1.union(qs1).values_list('name', 'order', 'id').get()\n         self.assertEqual(reserved_name[:2], ('a', 2))\n \n+    def test_union_projection_reuse_with_distinct_operands(self):\n+        ReservedName.objects.create(name='a', order=2)\n+        qs1 = ReservedName.objects.all()\n+        qs2 = ReservedName.objects.all()\n+        combined = qs1.union(qs2)\n+\n+        self.assertEqual(combined.values_list('name', 'order').get(), ('a', 2))\n+        self.assertEqual(combined.values_list('order').get(), (2,))\n+\n     def test_union_with_two_annotated_values_list(self):\n         qs1 = Number.objects.filter(num=1).annotate(\n             count=Value(0, IntegerField()),\n",
  "test_command": "./tests/runtests.py queries.test_qs_combinators.QuerySetSetOperationTests.test_union_projection_reuse_with_distinct_operands"
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
Creating test database for alias 'default'...
E
======================================================================
ERROR: test_union_projection_reuse_with_distinct_operands (queries.test_qs_combinators.QuerySetSetOperationTests)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/testbed/django/db/backends/utils.py", line 86, in _execute
    return self.cursor.execute(sql, params)
  File "/testbed/django/db/backends/sqlite3/base.py", line 391, in execute
    return Database.Cursor.execute(self, query, params)
sqlite3.OperationalError: SELECTs to the left and right of UNION do not have the same number of result columns

The above exception was the direct cause of the following exception:

Traceback (most recent call last):
  File "/testbed/tests/queries/test_qs_combinators.py", line 134, in test_union_projection_reuse_with_distinct_operands
    self.assertEqual(combined.values_list('order').get(), (2,))
  File "/testbed/django/db/models/query.py", line 408, in get
    num = len(clone)
  File "/testbed/django/db/models/query.py", line 258, in __len__
    self._fetch_all()
  File "/testbed/django/db/models/query.py", line 1240, in _fetch_all
    self._result_cache = list(self._iterable_class(self))
  File "/testbed/django/db/models/query.py", line 146, in __iter__
    return compiler.results_iter(tuple_expected=True, chunked_fetch=self.chunked_fetch, chunk_size=self.chunk_size)
  File "/testbed/django/db/models/sql/compiler.py", line 1046, in results_iter
    results = self.execute_sql(MULTI, chunked_fetch=chunked_fetch, chunk_size=chunk_size)
  File "/testbed/django/db/models/sql/compiler.py", line 1094, in execute_sql
    cursor.execute(sql, params)
  File "/testbed/django/db/backends/utils.py", line 68, in execute
    return self._execute_with_wrappers(sql, params, many=False, executor=self._execute)
  File "/testbed/django/db/backends/utils.py", line 77, in _execute_with_wrappers
    return executor(sql, params, many, context)
  File "/testbed/django/db/backends/utils.py", line 86, in _execute
    return self.cursor.execute(sql, params)
  File "/testbed/django/db/utils.py", line 89, in __exit__
    raise dj_exc_value.with_traceback(traceback) from exc_value
  File "/testbed/django/db/backends/utils.py", line 86, in _execute
    return self.cursor.execute(sql, params)
  File "/testbed/django/db/backends/sqlite3/base.py", line 391, in execute
    return Database.Cursor.execute(self, query, params)
django.db.utils.OperationalError: SELECTs to the left and right of UNION do not have the same number of result columns

----------------------------------------------------------------------
Ran 1 test in 0.002s

FAILED (errors=1)
Destroying test database for alias 'default'...
Testing against Django installed in '/testbed/django' with up to 24 processes
System check identified no issues (1 silenced).
[pipeline] test_exit_code=1

```

## candidate_b
returncode=0 passed=True test_ran=True
log=02_execution/attempt_01/candidate_b.log
```
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
Creating test database for alias 'default'...
.
----------------------------------------------------------------------
Ran 1 test in 0.001s

OK
Destroying test database for alias 'default'...
Testing against Django installed in '/testbed/django' with up to 24 processes
System check identified no issues (1 silenced).
[pipeline] test_exit_code=0

```
</validated_execution>
