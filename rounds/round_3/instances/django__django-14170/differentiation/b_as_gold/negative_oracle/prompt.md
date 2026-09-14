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

eligible_failure_fingerprint=false. If false, do not accept.
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
  "test_name": "test_bounds_accept_iso_year_keyword",
  "specification_gap": "The ISO-year lookup bounds entry point accepts the semantically named `iso_year` keyword and computes ISO-week-year boundaries, which may extend beyond the corresponding calendar year.",
  "input_description": "Construct an exact ISO-year lookup over a DateField expression and request bounds with `iso_year=2020`.",
  "expected_output": "The helper returns the backend-adapted inclusive interval from Monday 2019-12-30 through Sunday 2021-01-03.",
  "expected_passing_candidate": "candidate_b",
  "why_interesting": "This tests a distinct callable entry point and a cross-calendar-year boundary. Candidate_b accepts `iso_year` and returns the expected interval; candidate_a names the parameter `year`, so the same semantic keyword call raises TypeError.",
  "test_patch": "diff --git a/tests/lookup/test_lookups.py b/tests/lookup/test_lookups.py\n--- a/tests/lookup/test_lookups.py\n+++ b/tests/lookup/test_lookups.py\n@@ -1,8 +1,10 @@\n-from datetime import datetime\n+from datetime import date, datetime\n from unittest import mock\n \n-from django.db.models import DateTimeField, Value\n-from django.db.models.lookups import Lookup, YearLookup\n+from django.db import connection\n+from django.db.models import DateField, DateTimeField, Value\n+from django.db.models.functions import ExtractIsoYear\n+from django.db.models.lookups import IsoYearExact, Lookup, YearLookup\n from django.test import SimpleTestCase\n \n \n@@ -38,3 +40,18 @@ class YearLookupTests(SimpleTestCase):\n         msg = 'subclasses of YearLookup must provide a get_bound_params() method'\n         with self.assertRaisesMessage(NotImplementedError, msg):\n             look_up.get_bound_params(datetime(2010, 1, 1, 0, 0, 0), datetime(2010, 1, 1, 23, 59, 59))\n+\n+\n+class IsoYearLookupTests(SimpleTestCase):\n+    def test_bounds_accept_iso_year_keyword(self):\n+        lookup = IsoYearExact(\n+            ExtractIsoYear(Value(date(2020, 1, 1), output_field=DateField())),\n+            2020,\n+        )\n+        self.assertEqual(\n+            lookup.iso_year_lookup_bounds(connection, iso_year=2020),\n+            [\n+                connection.ops.adapt_datefield_value(date(2019, 12, 30)),\n+                connection.ops.adapt_datefield_value(date(2021, 1, 3)),\n+            ],\n+        )\n",
  "test_command": "./tests/runtests.py lookup.test_lookups.IsoYearLookupTests.test_bounds_accept_iso_year_keyword --verbosity 2"
}
</generated_test_proposal>
<candidate_execution_summary>
{
  "attempt": 3,
  "separates": true,
  "required_passing_candidate": "candidate_b",
  "meets_acceptance_rule": true,
  "results": [
    {
      "label": "candidate_a",
      "returncode": 1,
      "passed": false,
      "test_ran": true,
      "duration_seconds": 1.557,
      "log_path": "02_execution/attempt_03/candidate_a.log"
    },
    {
      "label": "candidate_b",
      "returncode": 0,
      "passed": true,
      "test_ran": true,
      "duration_seconds": 1.53,
      "log_path": "02_execution/attempt_03/candidate_b.log"
    }
  ]
}
</candidate_execution_summary>
<candidate_a_execution_log>
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
Testing against Django installed in '/testbed/django' with up to 24 processes
Importing application lookup
Skipping setup of unused database(s): default, other.
System check identified no issues (0 silenced).
test_bounds_accept_iso_year_keyword (lookup.test_lookups.IsoYearLookupTests) ... ERROR

======================================================================
ERROR: test_bounds_accept_iso_year_keyword (lookup.test_lookups.IsoYearLookupTests)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/testbed/tests/lookup/test_lookups.py", line 52, in test_bounds_accept_iso_year_keyword
    lookup.iso_year_lookup_bounds(connection, iso_year=2020),
TypeError: iso_year_lookup_bounds() got an unexpected keyword argument 'iso_year'

----------------------------------------------------------------------
Ran 1 test in 0.000s

FAILED (errors=1)
[pipeline] test_exit_code=1

</candidate_a_execution_log>
<candidate_b_execution_log>
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
Testing against Django installed in '/testbed/django' with up to 24 processes
Importing application lookup
Skipping setup of unused database(s): default, other.
System check identified no issues (0 silenced).
test_bounds_accept_iso_year_keyword (lookup.test_lookups.IsoYearLookupTests) ... ok

----------------------------------------------------------------------
Ran 1 test in 0.000s

OK
[pipeline] test_exit_code=0

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
  "duration_seconds": 1.511,
  "log_path": "gold_execution/gold.log"
}
</gold_execution_summary>
<gold_failure_contract>
null
</gold_failure_contract>
<gold_execution_log>
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
Testing against Django installed in '/testbed/django' with up to 24 processes
Importing application lookup
Skipping setup of unused database(s): default, other.
System check identified no issues (0 silenced).
test_lookups (unittest.loader._FailedTest) ... ERROR

======================================================================
ERROR: test_lookups (unittest.loader._FailedTest)
----------------------------------------------------------------------
ImportError: Failed to import test module: test_lookups
Traceback (most recent call last):
  File "/opt/miniconda3/envs/testbed/lib/python3.8/unittest/loader.py", line 154, in loadTestsFromName
    module = __import__(module_name)
  File "/testbed/tests/lookup/test_lookups.py", line 7, in <module>
    from django.db.models.lookups import IsoYearExact, Lookup, YearLookup
ImportError: cannot import name 'IsoYearExact' from 'django.db.models.lookups' (/testbed/django/db/models/lookups.py)


----------------------------------------------------------------------
Ran 1 test in 0.000s

FAILED (errors=1)
[pipeline] test_exit_code=1

</gold_execution_log>
