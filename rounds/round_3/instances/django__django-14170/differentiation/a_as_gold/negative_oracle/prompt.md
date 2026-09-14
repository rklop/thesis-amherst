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
Query optimization in YearLookup breaks filtering by "__iso_year"
Description
	 
		(last modified by Florian Demmer)
	 
The optimization to use BETWEEN instead of the EXTRACT operation in ​YearLookup is also registered for the ​"__iso_year" lookup, which breaks the functionality provided by ​ExtractIsoYear when used via the lookup.
This has unfortunately been broken ever since ExtractIsoYear was introduced in ​Django 2.2 via #28649 and wasn't easy to track down since ExtractIsoYear when used by itself eg. in an annotation works perfectly fine. Just when using the lookup in a filter, the optimization is used (even when explicitly using an annotation):
# annotation works
>>> qs = DTModel.objects.annotate(extracted=ExtractIsoYear('start_date')).only('id')
>>> print(qs.query)
SELECT "db_functions_dtmodel"."id", EXTRACT('isoyear' FROM "db_functions_dtmodel"."start_date") AS "extracted" FROM "db_functions_dtmodel"
# explicit annotation used in filter does not use "extracted" and adds BETWEEN
>>> print(qs.filter(extracted=2020).query)
SELECT "db_functions_dtmodel"."id", EXTRACT('isoyear' FROM "db_functions_dtmodel"."start_date") AS "extracted" FROM "db_functions_dtmodel" WHERE "db_functions_dtmodel"."start_date" BETWEEN 2020-01-01 AND 2020-12-31
# implicit lookup uses BETWEEN
>>> print(DTModel.objects.filter(start_date__iso_year=2020).only('id').query)
SELECT "db_functions_dtmodel"."id" FROM "db_functions_dtmodel" WHERE "db_functions_dtmodel"."start_date" BETWEEN 2020-01-01 AND 2020-12-31
This results in the wrong data being returned by filters using iso_year.
This PR fixes the behaviour, reverts the invalid changes to the tests and extends one test to catch this problem: ​https://github.com/django/django/pull/14157

</issue_statement>
<candidate_a_patch>
diff --git a/django/db/backends/base/operations.py b/django/db/backends/base/operations.py
index 0fcc607bcf..e7467d1493 100644
--- a/django/db/backends/base/operations.py
+++ b/django/db/backends/base/operations.py
@@ -558,6 +558,58 @@ class BaseDatabaseOperations:
         second = self.adapt_datetimefield_value(second)
         return [first, second]
 
+    def iso_year_lookup_bounds_for_date_field(self, value):
+        """
+        Return a two-elements list with the lower and upper bound to be used
+        with a BETWEEN operator to query a DateField value using an iso_year
+        lookup.
+
+        `value` is an int, containing the looked-up iso_year.
+        """
+        import datetime
+        # January 4 is always in the first ISO week of the year
+        # Find the Thursday of that week, then go back to Monday
+        jan4 = datetime.date(value, 1, 4)
+        thursday = jan4 + datetime.timedelta(days=(3 - jan4.weekday()))
+        first = thursday - datetime.timedelta(days=3)
+        # Find the last day of the ISO year
+        dec28 = datetime.date(value, 12, 28)
+        sunday = dec28 + datetime.timedelta(days=(6 - dec28.weekday()))
+        second = sunday
+        first = self.adapt_datefield_value(first)
+        second = self.adapt_datefield_value(second)
+        return [first, second]
+
+
+    def iso_year_lookup_bounds_for_datetime_field(self, value):
+        """
+        Return a two-elements list with the lower and upper bound to be used
+        with a BETWEEN operator to query a DateTimeField value using an iso_year
+        lookup.
+
+        `value` is an int, containing the looked-up iso_year.
+        """
+        import datetime
+        from django.utils import timezone
+        # January 4 is always in the first ISO week of the year
+        # Find the Thursday of that week, then go back to Monday
+        jan4 = datetime.datetime(value, 1, 4, 0, 0, 0)
+        thursday = jan4 + datetime.timedelta(days=(3 - jan4.weekday()))
+        first = thursday - datetime.timedelta(days=3)
+        # Find the last day of the ISO year
+        dec28 = datetime.datetime(value, 12, 28, 23, 59, 59, 999999)
+        sunday = dec28 + datetime.timedelta(days=(6 - dec28.weekday()))
+        second = sunday
+        if settings.USE_TZ:
+            tz = timezone.get_current_timezone()
+            first = timezone.make_aware(first, tz)
+            second = timezone.make_aware(second, tz)
+        first = self.adapt_datetimefield_value(first)
+        second = self.adapt_datetimefield_value(second)
+        return [first, second]
+
+
+
     def get_db_converters(self, expression):
         """
         Return a list of functions needed to convert field data.
diff --git a/django/db/models/functions/datetime.py b/django/db/models/functions/datetime.py
index 90e6f41be0..a0fb6de839 100644
--- a/django/db/models/functions/datetime.py
+++ b/django/db/models/functions/datetime.py
@@ -7,6 +7,7 @@ from django.db.models.fields import (
 )
 from django.db.models.lookups import (
     Transform, YearExact, YearGt, YearGte, YearLt, YearLte,
+    IsoYearExact, IsoYearGt, IsoYearGte, IsoYearLt, IsoYearLte,
 )
 from django.utils import timezone
 
@@ -164,11 +165,11 @@ ExtractYear.register_lookup(YearGte)
 ExtractYear.register_lookup(YearLt)
 ExtractYear.register_lookup(YearLte)
 
-ExtractIsoYear.register_lookup(YearExact)
-ExtractIsoYear.register_lookup(YearGt)
-ExtractIsoYear.register_lookup(YearGte)
-ExtractIsoYear.register_lookup(YearLt)
-ExtractIsoYear.register_lookup(YearLte)
+ExtractIsoYear.register_lookup(IsoYearExact)
+ExtractIsoYear.register_lookup(IsoYearGt)
+ExtractIsoYear.register_lookup(IsoYearGte)
+ExtractIsoYear.register_lookup(IsoYearLt)
+ExtractIsoYear.register_lookup(IsoYearLte)
 
 
 class Now(Func):
diff --git a/django/db/models/lookups.py b/django/db/models/lookups.py
index 916478d075..4757b6f190 100644
--- a/django/db/models/lookups.py
+++ b/django/db/models/lookups.py
@@ -597,6 +597,65 @@ class YearLte(YearLookup, LessThanOrEqual):
         return (finish,)
 
 
+class IsoYearLookup(Lookup):
+    def iso_year_lookup_bounds(self, connection, year):
+        output_field = self.lhs.lhs.output_field
+        if isinstance(output_field, DateTimeField):
+            bounds = connection.ops.iso_year_lookup_bounds_for_datetime_field(year)
+        else:
+            bounds = connection.ops.iso_year_lookup_bounds_for_date_field(year)
+        return bounds
+
+    def as_sql(self, compiler, connection):
+        # Avoid the extract operation if the rhs is a direct value to allow
+        # indexes to be used.
+        if self.rhs_is_direct_value():
+            # Skip the extract part by directly using the originating field,
+            # that is self.lhs.lhs.
+            lhs_sql, params = self.process_lhs(compiler, connection, self.lhs.lhs)
+            rhs_sql, _ = self.process_rhs(compiler, connection)
+            rhs_sql = self.get_direct_rhs_sql(connection, rhs_sql)
+            start, finish = self.iso_year_lookup_bounds(connection, self.rhs)
+            params.extend(self.get_bound_params(start, finish))
+            return '%s %s' % (lhs_sql, rhs_sql), params
+        return super().as_sql(compiler, connection)
+
+    def get_direct_rhs_sql(self, connection, rhs):
+        return connection.operators[self.lookup_name] % rhs
+
+    def get_bound_params(self, start, finish):
+        raise NotImplementedError(
+            'subclasses of IsoYearLookup must provide a get_bound_params() method'
+        )
+
+
+class IsoYearExact(IsoYearLookup, Exact):
+    def get_direct_rhs_sql(self, connection, rhs):
+        return 'BETWEEN %s AND %s'
+
+    def get_bound_params(self, start, finish):
+        return (start, finish)
+
+
+class IsoYearGt(IsoYearLookup, GreaterThan):
+    def get_bound_params(self, start, finish):
+        return (finish,)
+
+
+class IsoYearGte(IsoYearLookup, GreaterThanOrEqual):
+    def get_bound_params(self, start, finish):
+        return (start,)
+
+
+class IsoYearLt(IsoYearLookup, LessThan):
+    def get_bound_params(self, start, finish):
+        return (start,)
+
+
+class IsoYearLte(IsoYearLookup, LessThanOrEqual):
+    def get_bound_params(self, start, finish):
+        return (finish,)
+
 class UUIDTextMixin:
     """
     Strip hyphens from a value when filtering a UUIDField on backends without

</candidate_a_patch>
<candidate_b_patch>
diff --git a/django/db/backends/base/operations.py b/django/db/backends/base/operations.py
index 0fcc607bcf..5ad58b2527 100644
--- a/django/db/backends/base/operations.py
+++ b/django/db/backends/base/operations.py
@@ -558,6 +558,46 @@ class BaseDatabaseOperations:
         second = self.adapt_datetimefield_value(second)
         return [first, second]
 
+    def iso_year_lookup_bounds_for_date_field(self, value):
+        """
+        Return a two-elements list with the lower and upper bound to be used
+        with a BETWEEN operator to query a DateField value using an iso_year
+        lookup.
+
+        `value` is an int, containing the looked-up iso_year.
+        """
+        first = datetime.date(value, 1, 4)
+        # Find the Monday of the first ISO week
+        first = first - datetime.timedelta(days=first.weekday())
+        second = datetime.date(value, 12, 28)
+        # Find the Sunday of the last ISO week
+        second = second + datetime.timedelta(days=6 - second.weekday())
+        first = self.adapt_datefield_value(first)
+        second = self.adapt_datefield_value(second)
+        return [first, second]
+
+    def iso_year_lookup_bounds_for_datetime_field(self, value):
+        """
+        Return a two-elements list with the lower and upper bound to be used
+        with a BETWEEN operator to query a DateTimeField value using an iso_year
+        lookup.
+
+        `value` is an int, containing the looked-up iso_year.
+        """
+        first = datetime.datetime(value, 1, 4)
+        # Find the Monday of the first ISO week
+        first = first - datetime.timedelta(days=first.weekday())
+        second = datetime.datetime(value, 12, 28, 23, 59, 59, 999999)
+        # Find the Sunday of the last ISO week
+        second = second + datetime.timedelta(days=6 - second.weekday())
+        if settings.USE_TZ:
+            tz = timezone.get_current_timezone()
+            first = timezone.make_aware(first, tz)
+            second = timezone.make_aware(second, tz)
+        first = self.adapt_datetimefield_value(first)
+        second = self.adapt_datetimefield_value(second)
+        return [first, second]
+
     def get_db_converters(self, expression):
         """
         Return a list of functions needed to convert field data.
diff --git a/django/db/models/functions/datetime.py b/django/db/models/functions/datetime.py
index 90e6f41be0..a0fb6de839 100644
--- a/django/db/models/functions/datetime.py
+++ b/django/db/models/functions/datetime.py
@@ -7,6 +7,7 @@ from django.db.models.fields import (
 )
 from django.db.models.lookups import (
     Transform, YearExact, YearGt, YearGte, YearLt, YearLte,
+    IsoYearExact, IsoYearGt, IsoYearGte, IsoYearLt, IsoYearLte,
 )
 from django.utils import timezone
 
@@ -164,11 +165,11 @@ ExtractYear.register_lookup(YearGte)
 ExtractYear.register_lookup(YearLt)
 ExtractYear.register_lookup(YearLte)
 
-ExtractIsoYear.register_lookup(YearExact)
-ExtractIsoYear.register_lookup(YearGt)
-ExtractIsoYear.register_lookup(YearGte)
-ExtractIsoYear.register_lookup(YearLt)
-ExtractIsoYear.register_lookup(YearLte)
+ExtractIsoYear.register_lookup(IsoYearExact)
+ExtractIsoYear.register_lookup(IsoYearGt)
+ExtractIsoYear.register_lookup(IsoYearGte)
+ExtractIsoYear.register_lookup(IsoYearLt)
+ExtractIsoYear.register_lookup(IsoYearLte)
 
 
 class Now(Func):
diff --git a/django/db/models/lookups.py b/django/db/models/lookups.py
index 916478d075..223629b414 100644
--- a/django/db/models/lookups.py
+++ b/django/db/models/lookups.py
@@ -597,6 +597,66 @@ class YearLte(YearLookup, LessThanOrEqual):
         return (finish,)
 
 
+class IsoYearLookup(Lookup):
+    def iso_year_lookup_bounds(self, connection, iso_year):
+        output_field = self.lhs.lhs.output_field
+        if isinstance(output_field, DateTimeField):
+            bounds = connection.ops.iso_year_lookup_bounds_for_datetime_field(iso_year)
+        else:
+            bounds = connection.ops.iso_year_lookup_bounds_for_date_field(iso_year)
+        return bounds
+
+    def as_sql(self, compiler, connection):
+        # Avoid the extract operation if the rhs is a direct value to allow
+        # indexes to be used.
+        if self.rhs_is_direct_value():
+            # Skip the extract part by directly using the originating field,
+            # that is self.lhs.lhs.
+            lhs_sql, params = self.process_lhs(compiler, connection, self.lhs.lhs)
+            rhs_sql, _ = self.process_rhs(compiler, connection)
+            rhs_sql = self.get_direct_rhs_sql(connection, rhs_sql)
+            start, finish = self.iso_year_lookup_bounds(connection, self.rhs)
+            params.extend(self.get_bound_params(start, finish))
+            return '%s %s' % (lhs_sql, rhs_sql), params
+        return super().as_sql(compiler, connection)
+
+    def get_direct_rhs_sql(self, connection, rhs):
+        return connection.operators[self.lookup_name] % rhs
+
+    def get_bound_params(self, start, finish):
+        raise NotImplementedError(
+            'subclasses of IsoYearLookup must provide a get_bound_params() method'
+        )
+
+
+class IsoYearExact(IsoYearLookup, Exact):
+    def get_direct_rhs_sql(self, connection, rhs):
+        return 'BETWEEN %s AND %s'
+
+    def get_bound_params(self, start, finish):
+        return (start, finish)
+
+
+class IsoYearGt(IsoYearLookup, GreaterThan):
+    def get_bound_params(self, start, finish):
+        return (finish,)
+
+
+class IsoYearGte(IsoYearLookup, GreaterThanOrEqual):
+    def get_bound_params(self, start, finish):
+        return (start,)
+
+
+class IsoYearLt(IsoYearLookup, LessThan):
+    def get_bound_params(self, start, finish):
+        return (start,)
+
+
+class IsoYearLte(IsoYearLookup, LessThanOrEqual):
+    def get_bound_params(self, start, finish):
+        return (finish,)
+
+
 class UUIDTextMixin:
     """
     Strip hyphens from a value when filtering a UUIDField on backends without

</candidate_b_patch>
<generated_test_proposal>
{
  "test_name": "test_extract_iso_year_lookup_bounds_keyword",
  "specification_gap": "The ISO-year lookup-bound helper should accept the requested year through the same `year` keyword convention as the existing year lookup helper, while calculating ISO-calendar rather than calendar-year bounds.",
  "input_description": "Obtain the lookup generated by `start_date__iso_year=2015` and request its optimized date bounds with `year=2015`.",
  "expected_output": "The returned backend-adapted bounds represent 2014-12-29 through 2016-01-03, the first and last dates of ISO year 2015. Candidate_b instead raises `TypeError` because it renamed the accepted keyword to `iso_year`.",
  "expected_passing_candidate": "candidate_a",
  "why_interesting": "The candidates otherwise compute identical ISO-year ranges. This exercises an alternate callable entry point and verifies concrete boundary values, exposing their narrow signature incompatibility without inspecting source text.",
  "test_patch": "diff --git a/tests/db_functions/datetime/test_extract_trunc.py b/tests/db_functions/datetime/test_extract_trunc.py\n--- a/tests/db_functions/datetime/test_extract_trunc.py\n+++ b/tests/db_functions/datetime/test_extract_trunc.py\n@@ -10,6 +10,7 @@ except ImportError:\n         zoneinfo = None\n \n from django.conf import settings\n+from django.db import connection\n from django.db.models import (\n     DateField, DateTimeField, F, IntegerField, Max, OuterRef, Subquery,\n     TimeField,\n@@ -346,7 +347,20 @@ class DateFunctionTests(TestCase):\n         )\n         # Both dates are from the same week year.\n         self.assertEqual(DTModel.objects.filter(start_datetime__iso_year=ExtractIsoYear('start_datetime')).count(), 2)\n \n+    def test_extract_iso_year_lookup_bounds_keyword(self):\n+        lookup = (\n+            DTModel.objects.filter(start_date__iso_year=2015)\n+            .query.where.children[0]\n+        )\n+        self.assertEqual(\n+            lookup.iso_year_lookup_bounds(connection, year=2015),\n+            [\n+                connection.ops.adapt_datefield_value(datetime(2014, 12, 29).date()),\n+                connection.ops.adapt_datefield_value(datetime(2016, 1, 3).date()),\n+            ],\n+        )\n+\n     def test_extract_iso_year_func_boundaries(self):\n         end_datetime = datetime(2016, 6, 15, 14, 10, 50, 123)\n         if settings.USE_TZ:\n",
  "test_command": "cd /testbed && ./tests/runtests.py db_functions.datetime.test_extract_trunc.DateFunctionTests.test_extract_iso_year_lookup_bounds_keyword"
}
</generated_test_proposal>
<candidate_execution_summary>
{
  "attempt": 2,
  "separates": true,
  "required_passing_candidate": "candidate_a",
  "meets_acceptance_rule": true,
  "results": [
    {
      "label": "candidate_a",
      "returncode": 0,
      "passed": true,
      "test_ran": true,
      "duration_seconds": 1.565,
      "log_path": "02_execution/attempt_02/candidate_a.log"
    },
    {
      "label": "candidate_b",
      "returncode": 1,
      "passed": false,
      "test_ran": true,
      "duration_seconds": 1.511,
      "log_path": "02_execution/attempt_02/candidate_b.log"
    }
  ]
}
</candidate_execution_summary>
<candidate_a_execution_log>
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
Creating test database for alias 'default'...
Testing against Django installed in '/testbed/django' with up to 24 processes
System check identified no issues (0 silenced).
.
----------------------------------------------------------------------
Ran 1 test in 0.001s

OK
Destroying test database for alias 'default'...
[pipeline] test_exit_code=0

</candidate_a_execution_log>
<candidate_b_execution_log>
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
Creating test database for alias 'default'...
Testing against Django installed in '/testbed/django' with up to 24 processes
System check identified no issues (0 silenced).
E
======================================================================
ERROR: test_extract_iso_year_lookup_bounds_keyword (db_functions.datetime.test_extract_trunc.DateFunctionTests)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/testbed/tests/db_functions/datetime/test_extract_trunc.py", line 357, in test_extract_iso_year_lookup_bounds_keyword
    lookup.iso_year_lookup_bounds(connection, year=2015),
TypeError: iso_year_lookup_bounds() got an unexpected keyword argument 'year'

----------------------------------------------------------------------
Ran 1 test in 0.001s

FAILED (errors=1)
Destroying test database for alias 'default'...
[pipeline] test_exit_code=1

</candidate_b_execution_log>
<official_gold_patch>
diff --git a/django/db/backends/base/operations.py b/django/db/backends/base/operations.py
--- a/django/db/backends/base/operations.py
+++ b/django/db/backends/base/operations.py
@@ -526,30 +526,46 @@ def adapt_ipaddressfield_value(self, value):
         """
         return value or None
 
-    def year_lookup_bounds_for_date_field(self, value):
+    def year_lookup_bounds_for_date_field(self, value, iso_year=False):
         """
         Return a two-elements list with the lower and upper bound to be used
         with a BETWEEN operator to query a DateField value using a year
         lookup.
 
         `value` is an int, containing the looked-up year.
+        If `iso_year` is True, return bounds for ISO-8601 week-numbering years.
         """
-        first = datetime.date(value, 1, 1)
-        second = datetime.date(value, 12, 31)
+        if iso_year:
+            first = datetime.date.fromisocalendar(value, 1, 1)
+            second = (
+                datetime.date.fromisocalendar(value + 1, 1, 1) -
+                datetime.timedelta(days=1)
+            )
+        else:
+            first = datetime.date(value, 1, 1)
+            second = datetime.date(value, 12, 31)
         first = self.adapt_datefield_value(first)
         second = self.adapt_datefield_value(second)
         return [first, second]
 
-    def year_lookup_bounds_for_datetime_field(self, value):
+    def year_lookup_bounds_for_datetime_field(self, value, iso_year=False):
         """
         Return a two-elements list with the lower and upper bound to be used
         with a BETWEEN operator to query a DateTimeField value using a year
         lookup.
 
         `value` is an int, containing the looked-up year.
+        If `iso_year` is True, return bounds for ISO-8601 week-numbering years.
         """
-        first = datetime.datetime(value, 1, 1)
-        second = datetime.datetime(value, 12, 31, 23, 59, 59, 999999)
+        if iso_year:
+            first = datetime.datetime.fromisocalendar(value, 1, 1)
+            second = (
+                datetime.datetime.fromisocalendar(value + 1, 1, 1) -
+                datetime.timedelta(microseconds=1)
+            )
+        else:
+            first = datetime.datetime(value, 1, 1)
+            second = datetime.datetime(value, 12, 31, 23, 59, 59, 999999)
         if settings.USE_TZ:
             tz = timezone.get_current_timezone()
             first = timezone.make_aware(first, tz)
diff --git a/django/db/models/lookups.py b/django/db/models/lookups.py
--- a/django/db/models/lookups.py
+++ b/django/db/models/lookups.py
@@ -539,11 +539,17 @@ class IRegex(Regex):
 
 class YearLookup(Lookup):
     def year_lookup_bounds(self, connection, year):
+        from django.db.models.functions import ExtractIsoYear
+        iso_year = isinstance(self.lhs, ExtractIsoYear)
         output_field = self.lhs.lhs.output_field
         if isinstance(output_field, DateTimeField):
-            bounds = connection.ops.year_lookup_bounds_for_datetime_field(year)
+            bounds = connection.ops.year_lookup_bounds_for_datetime_field(
+                year, iso_year=iso_year,
+            )
         else:
-            bounds = connection.ops.year_lookup_bounds_for_date_field(year)
+            bounds = connection.ops.year_lookup_bounds_for_date_field(
+                year, iso_year=iso_year,
+            )
         return bounds
 
     def as_sql(self, compiler, connection):

</official_gold_patch>
<gold_execution_summary>
{
  "label": "gold",
  "returncode": 1,
  "passed": false,
  "test_ran": true,
  "duration_seconds": 1.583,
  "log_path": "gold_execution/gold.log"
}
</gold_execution_summary>
<gold_failure_contract>
{
  "kind": "behavioral_test_failure",
  "required_substrings": [
    "test_extract_iso_year_lookup_bounds_keyword"
  ],
  "required_any_substrings": [
    "FAILED"
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
  "reference_log_sha256": "80ffa824f73bcb1c19a0798340d3e51fe2bc65da1053cae93c2a5e06d0d4b62b"
}
</gold_failure_contract>
<gold_execution_log>
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
Testing against Django installed in '/testbed/django' with up to 24 processes
Creating test database for alias 'default'...
System check identified no issues (0 silenced).
E
======================================================================
ERROR: test_extract_iso_year_lookup_bounds_keyword (db_functions.datetime.test_extract_trunc.DateFunctionTests)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/testbed/tests/db_functions/datetime/test_extract_trunc.py", line 357, in test_extract_iso_year_lookup_bounds_keyword
    lookup.iso_year_lookup_bounds(connection, year=2015),
AttributeError: 'YearExact' object has no attribute 'iso_year_lookup_bounds'

----------------------------------------------------------------------
Ran 1 test in 0.001s

FAILED (errors=1)
Destroying test database for alias 'default'...
[pipeline] test_exit_code=1

</gold_execution_log>
