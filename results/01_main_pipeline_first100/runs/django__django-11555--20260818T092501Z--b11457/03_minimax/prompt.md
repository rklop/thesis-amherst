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
order_by() a parent model crash when Meta.ordering contains expressions.
Description
	 
		(last modified by Jonny Fuller)
	 
Hi friends,
During testing I discovered a strange bug when using a query expression for ordering during multi-table inheritance. You can find the full write up as well as reproducible test repository ​https://github.com/JonnyWaffles/djangoordermetabug. The bug occurs because the field is an OrderBy object, not a string, during get_order_dir. The linked stacktrace should make the issue obvious, but what I don't understand is why it only fails during test db setup, not during repl or script use. I wish I could help more and come up with a real solution. Hopefully, this is enough for someone wiser to find the culprit.

</issue_statement>

<original_test_patch>
diff --git a/tests/ordering/models.py b/tests/ordering/models.py
--- a/tests/ordering/models.py
+++ b/tests/ordering/models.py
@@ -54,6 +54,10 @@ class Meta:
         ordering = (models.F('author').asc(nulls_first=True), 'id')
 
 
+class ChildArticle(Article):
+    pass
+
+
 class Reference(models.Model):
     article = models.ForeignKey(OrderedByAuthorArticle, models.CASCADE)
 
diff --git a/tests/ordering/tests.py b/tests/ordering/tests.py
--- a/tests/ordering/tests.py
+++ b/tests/ordering/tests.py
@@ -9,7 +9,7 @@
 from django.test import TestCase
 from django.utils.deprecation import RemovedInDjango31Warning
 
-from .models import Article, Author, OrderedByFArticle, Reference
+from .models import Article, Author, ChildArticle, OrderedByFArticle, Reference
 
 
 class OrderingTests(TestCase):
@@ -462,6 +462,26 @@ def test_default_ordering_by_f_expression(self):
             attrgetter('headline')
         )
 
+    def test_order_by_ptr_field_with_default_ordering_by_expression(self):
+        ca1 = ChildArticle.objects.create(
+            headline='h2',
+            pub_date=datetime(2005, 7, 27),
+            author=self.author_2,
+        )
+        ca2 = ChildArticle.objects.create(
+            headline='h2',
+            pub_date=datetime(2005, 7, 27),
+            author=self.author_1,
+        )
+        ca3 = ChildArticle.objects.create(
+            headline='h3',
+            pub_date=datetime(2005, 7, 27),
+            author=self.author_1,
+        )
+        ca4 = ChildArticle.objects.create(headline='h1', pub_date=datetime(2005, 7, 28))
+        articles = ChildArticle.objects.order_by('article_ptr')
+        self.assertSequenceEqual(articles, [ca4, ca2, ca1, ca3])
+
     def test_deprecated_values_annotate(self):
         msg = (
             "Article QuerySet won't use Meta.ordering in Django 3.1. Add "

</original_test_patch>

<candidate_a_patch>
diff --git a/django/db/models/sql/compiler.py b/django/db/models/sql/compiler.py
index eaccc96d7d..2a89eee21c 100644
--- a/django/db/models/sql/compiler.py
+++ b/django/db/models/sql/compiler.py
@@ -722,8 +722,16 @@ class SQLCompiler:
 
             results = []
             for item in opts.ordering:
-                results.extend(self.find_ordering_name(item, opts, alias,
-                                                       order, already_seen))
+                if hasattr(item, 'resolve_expression'):
+                    if not isinstance(item, OrderBy):
+                        item = item.asc()
+                    if not self.query.standard_ordering:
+                        item = item.copy()
+                        item.reverse_ordering()
+                    results.append((item, False))
+                else:
+                    results.extend(self.find_ordering_name(item, opts, alias,
+                                                           order, already_seen))
             return results
         targets, alias, _ = self.query.trim_joins(targets, joins, path)
         return [(OrderBy(transform_function(t, alias), descending=descending), False) for t in targets]

</candidate_a_patch>

<candidate_b_patch>
diff --git a/django/db/models/sql/compiler.py b/django/db/models/sql/compiler.py
--- a/django/db/models/sql/compiler.py
+++ b/django/db/models/sql/compiler.py
@@ -722,6 +722,9 @@ def find_ordering_name(self, name, opts, alias=None, default_order='ASC',
 
             results = []
             for item in opts.ordering:
+                if isinstance(item, OrderBy):
+                    results.append((item, False))
+                    continue
                 results.extend(self.find_ordering_name(item, opts, alias,
                                                        order, already_seen))
             return results

</candidate_b_patch>

<generated_test_proposal>
{
  "test_name": "test_unwrapped_function_in_parent_ordering",
  "specification_gap": "Related or parent-pointer ordering must support any valid Meta.ordering expression, including a bare database function, not only expressions already wrapped in OrderBy.",
  "input_description": "Create multi-table child rows named 'Zebra' and 'alpha'. Order them by the inherited parent pointer, whose parent model declares Meta.ordering=(Lower('name'),).",
  "expected_output": "Evaluating the queryset returns ['alpha', 'Zebra'] without raising a type error.",
  "expected_passing_candidate": "candidate_a",
  "why_interesting": "This exercises the bare-expression variant identified in the issue discussion through an alternate public entry point. Candidate A normalizes any resolvable expression to OrderBy; candidate B special-cases OrderBy only and passes Lower into string-only parsing. The standalone added file also avoids the prior patch-context failure.",
  "test_patch": "diff --git a/tests/ordering/test_related_expression_ordering.py b/tests/ordering/test_related_expression_ordering.py\nnew file mode 100644\n--- /dev/null\n+++ b/tests/ordering/test_related_expression_ordering.py\n@@ -0,0 +1,26 @@\n+from django.db import models\n+from django.db.models.functions import Lower\n+from django.test import TestCase\n+\n+\n+class LowerOrderedParent(models.Model):\n+    name = models.CharField(max_length=20)\n+\n+    class Meta:\n+        ordering = (Lower('name'),)\n+\n+\n+class LowerOrderedChild(LowerOrderedParent):\n+    pass\n+\n+\n+class RelatedExpressionOrderingTests(TestCase):\n+    def test_unwrapped_function_in_parent_ordering(self):\n+        LowerOrderedChild.objects.create(name='Zebra')\n+        LowerOrderedChild.objects.create(name='alpha')\n+        names = (\n+            LowerOrderedChild.objects\n+            .order_by('lowerorderedparent_ptr')\n+            .values_list('name', flat=True)\n+        )\n+        self.assertEqual(list(names), ['alpha', 'Zebra'])\n",
  "test_command": "./tests/runtests.py ordering.test_related_expression_ordering.RelatedExpressionOrderingTests.test_unwrapped_function_in_parent_ordering"
}
</generated_test_proposal>

<validated_execution>
## candidate_a
returncode=0 passed=True test_ran=True
log=02_execution/attempt_02/candidate_a.log
```
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
Creating test database for alias 'default'...
.Testing against Django installed in '/testbed/django' with up to 24 processes
System check identified no issues (0 silenced).

----------------------------------------------------------------------
Ran 1 test in 0.001s

OK
Destroying test database for alias 'default'...
[pipeline] test_exit_code=0

```

## candidate_b
returncode=1 passed=False test_ran=True
log=02_execution/attempt_02/candidate_b.log
```
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
Creating test database for alias 'default'...
ETesting against Django installed in '/testbed/django' with up to 24 processes
System check identified no issues (0 silenced).

======================================================================
ERROR: test_unwrapped_function_in_parent_ordering (ordering.test_related_expression_ordering.RelatedExpressionOrderingTests)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/testbed/tests/ordering/test_related_expression_ordering.py", line 26, in test_unwrapped_function_in_parent_ordering
    self.assertEqual(list(names), ['alpha', 'Zebra'])
  File "/testbed/django/db/models/query.py", line 276, in __iter__
    self._fetch_all()
  File "/testbed/django/db/models/query.py", line 1240, in _fetch_all
    self._result_cache = list(self._iterable_class(self))
  File "/testbed/django/db/models/query.py", line 184, in __iter__
    for row in compiler.results_iter(chunked_fetch=self.chunked_fetch, chunk_size=self.chunk_size):
  File "/testbed/django/db/models/sql/compiler.py", line 1050, in results_iter
    results = self.execute_sql(MULTI, chunked_fetch=chunked_fetch, chunk_size=chunk_size)
  File "/testbed/django/db/models/sql/compiler.py", line 1085, in execute_sql
    sql, params = self.as_sql()
  File "/testbed/django/db/models/sql/compiler.py", line 480, in as_sql
    extra_select, order_by, group_by = self.pre_sql_setup()
  File "/testbed/django/db/models/sql/compiler.py", line 53, in pre_sql_setup
    order_by = self.get_order_by()
  File "/testbed/django/db/models/sql/compiler.py", line 330, in get_order_by
    field, self.query.get_meta(), default_order=asc))
  File "/testbed/django/db/models/sql/compiler.py", line 729, in find_ordering_name
    order, already_seen))
  File "/testbed/django/db/models/sql/compiler.py", line 707, in find_ordering_name
    name, order = get_order_dir(name, default_order)
  File "/testbed/django/db/models/sql/query.py", line 2221, in get_order_dir
    if field[0] == '-':
TypeError: 'Lower' object does not support indexing

----------------------------------------------------------------------
Ran 1 test in 0.002s

FAILED (errors=1)
Destroying test database for alias 'default'...
[pipeline] test_exit_code=1

```
</validated_execution>
