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

<original_test_patch>
diff --git a/tests/db_functions/datetime/test_extract_trunc.py b/tests/db_functions/datetime/test_extract_trunc.py
--- a/tests/db_functions/datetime/test_extract_trunc.py
+++ b/tests/db_functions/datetime/test_extract_trunc.py
@@ -359,9 +359,9 @@ def test_extract_iso_year_func_boundaries(self):
             week_52_day_2014 = timezone.make_aware(week_52_day_2014, is_dst=False)
             week_53_day_2015 = timezone.make_aware(week_53_day_2015, is_dst=False)
         days = [week_52_day_2014, week_1_day_2014_2015, week_53_day_2015]
-        self.create_model(week_53_day_2015, end_datetime)
-        self.create_model(week_52_day_2014, end_datetime)
-        self.create_model(week_1_day_2014_2015, end_datetime)
+        obj_1_iso_2014 = self.create_model(week_52_day_2014, end_datetime)
+        obj_1_iso_2015 = self.create_model(week_1_day_2014_2015, end_datetime)
+        obj_2_iso_2015 = self.create_model(week_53_day_2015, end_datetime)
         qs = DTModel.objects.filter(start_datetime__in=days).annotate(
             extracted=ExtractIsoYear('start_datetime'),
         ).order_by('start_datetime')
@@ -371,6 +371,19 @@ def test_extract_iso_year_func_boundaries(self):
             (week_53_day_2015, 2015),
         ], lambda m: (m.start_datetime, m.extracted))
 
+        qs = DTModel.objects.filter(
+            start_datetime__iso_year=2015,
+        ).order_by('start_datetime')
+        self.assertSequenceEqual(qs, [obj_1_iso_2015, obj_2_iso_2015])
+        qs = DTModel.objects.filter(
+            start_datetime__iso_year__gt=2014,
+        ).order_by('start_datetime')
+        self.assertSequenceEqual(qs, [obj_1_iso_2015, obj_2_iso_2015])
+        qs = DTModel.objects.filter(
+            start_datetime__iso_year__lte=2014,
+        ).order_by('start_datetime')
+        self.assertSequenceEqual(qs, [obj_1_iso_2014])
+
     def test_extract_month_func(self):
         start_datetime = datetime(2015, 6, 15, 14, 30, 50, 321)
         end_datetime = datetime(2016, 6, 15, 14, 10, 50, 123)

</original_test_patch>

<candidate_a_patch>
diff --git a/django/db/backends/base/operations.py b/django/db/backends/base/operations.py
index 0fcc607bcf..ab27ca696d 100644
--- a/django/db/backends/base/operations.py
+++ b/django/db/backends/base/operations.py
@@ -558,6 +558,42 @@ class BaseDatabaseOperations:
         second = self.adapt_datetimefield_value(second)
         return [first, second]
 
+    def iso_year_lookup_bounds_for_datetime_field(self, value):
+        """
+        Return a two-elements list with the lower and upper bound to be used
+        with a BETWEEN operator to query a DateTimeField value using an
+        iso_year lookup.
+
+        `value` is an int, containing the looked-up iso year.
+        """
+        first, second = self.iso_year_lookup_bounds_for_date_field(value)
+        first = datetime.datetime.combine(first, datetime.time.min)
+        second = datetime.datetime.combine(second, datetime.time.max)
+        if settings.USE_TZ:
+            tz = timezone.get_current_timezone()
+            first = timezone.make_aware(first, tz)
+            second = timezone.make_aware(second, tz)
+        return [first, second]
+
+    def iso_year_lookup_bounds_for_date_field(self, value):
+        """
+        Return a two-elements list with the lower and upper bound to be used
+        with a BETWEEN operator to query a DateField value using an iso_year
+        lookup.
+
+        `value` is an int, containing the looked-up iso year.
+        """
+        first = datetime.date(value, 1, 4)
+        # Find the Monday of the week containing January 4th
+        days_to_monday = first.weekday()
+        first = first - datetime.timedelta(days=days_to_monday)
+        # End is the day before the start of next ISO year
+        next_year = value + 1
+        jan4_next = datetime.date(next_year, 1, 4)
+        days_to_monday_next = jan4_next.weekday()
+        second = jan4_next - datetime.timedelta(days=days_to_monday_next) - datetime.timedelta(days=1)
+        return [first, second]
+
     def get_db_converters(self, expression):
         """
         Return a list of functions needed to convert field data.
diff --git a/django/db/models/lookups.py b/django/db/models/lookups.py
index 916478d075..d0fa2ef4a4 100644
--- a/django/db/models/lookups.py
+++ b/django/db/models/lookups.py
@@ -540,7 +540,13 @@ class IRegex(Regex):
 class YearLookup(Lookup):
     def year_lookup_bounds(self, connection, year):
         output_field = self.lhs.lhs.output_field
-        if isinstance(output_field, DateTimeField):
+        # Use ISO year bounds for iso_year lookup, regular year bounds otherwise
+        if self.lhs.lookup_name == 'iso_year':
+            if isinstance(output_field, DateTimeField):
+                bounds = connection.ops.iso_year_lookup_bounds_for_datetime_field(year)
+            else:
+                bounds = connection.ops.iso_year_lookup_bounds_for_date_field(year)
+        elif isinstance(output_field, DateTimeField):
             bounds = connection.ops.year_lookup_bounds_for_datetime_field(year)
         else:
             bounds = connection.ops.year_lookup_bounds_for_date_field(year)

</candidate_a_patch>

<candidate_b_patch>
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

</candidate_b_patch>

<generated_test_proposal>
{
  "test_name": "test_iso_year_lookup_bound_params",
  "specification_gap": "Optimized ISO-year lookup bounds must pass through the backend's DateField value adapter, just like ordinary year bounds. Otherwise generated SQL parameters may have types unsupported by a database driver.",
  "input_description": "Compile a SQLite queryset filtering an Item DateField with date__iso_year=2020 and inspect the parameters returned by Query.sql_with_params().",
  "expected_output": "The two bound parameters are the SQLite-adapted strings ('2019-12-30', '2021-01-03').",
  "expected_passing_candidate": "candidate_b",
  "why_interesting": "Candidate_b extends the existing year-bound operation, preserving its backend adaptation step. Candidate_a uses new ISO-specific operations that return raw datetime.date objects. The test checks an observable ORM compilation result at both ISO-year boundaries and avoids relying on SQLite's later implicit parameter conversion.",
  "test_patch": "diff --git a/tests/backends/sqlite/test_operations.py b/tests/backends/sqlite/test_operations.py\n--- a/tests/backends/sqlite/test_operations.py\n+++ b/tests/backends/sqlite/test_operations.py\n@@ -4,7 +4,7 @@ from django.core.management.color import no_style\n from django.db import connection\n from django.test import TestCase\n \n-from ..models import Person, Tag\n+from ..models import Item, Person, Tag\n \n \n @unittest.skipUnless(connection.vendor == 'sqlite', 'SQLite tests.')\n@@ -11,4 +11,10 @@ from ..models import Person, Tag\n class SQLiteOperationsTests(TestCase):\n+    def test_iso_year_lookup_bound_params(self):\n+        _, params = Item.objects.filter(\n+            date__iso_year=2020,\n+        ).query.sql_with_params()\n+        self.assertEqual(params, ('2019-12-30', '2021-01-03'))\n+\n     def test_sql_flush(self):\n         self.assertEqual(\n             connection.ops.sql_flush(\n",
  "test_command": "python tests/runtests.py backends.sqlite.test_operations.SQLiteOperationsTests.test_iso_year_lookup_bound_params"
}
</generated_test_proposal>

<validated_execution>
## candidate_a
returncode=1 passed=False test_ran=True
log=02_execution/attempt_02/candidate_a.log
```
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
Testing against Django installed in '/testbed/django' with up to 24 processes
Creating test database for alias 'default'...
System check identified no issues (0 silenced).
F
======================================================================
FAIL: test_iso_year_lookup_bound_params (backends.sqlite.test_operations.SQLiteOperationsTests)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/testbed/tests/backends/sqlite/test_operations.py", line 16, in test_iso_year_lookup_bound_params
    self.assertEqual(params, ('2019-12-30', '2021-01-03'))
AssertionError: Tuples differ: (datetime.date(2019, 12, 30), datetime.date(2021, 1, 3)) != ('2019-12-30', '2021-01-03')

First differing element 0:
datetime.date(2019, 12, 30)
'2019-12-30'

- (datetime.date(2019, 12, 30), datetime.date(2021, 1, 3))
+ ('2019-12-30', '2021-01-03')

----------------------------------------------------------------------
Ran 1 test in 0.001s

FAILED (failures=1)
Destroying test database for alias 'default'...
[pipeline] test_exit_code=1

```

## candidate_b
returncode=0 passed=True test_ran=True
log=02_execution/attempt_02/candidate_b.log
```
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
Testing against Django installed in '/testbed/django' with up to 24 processes
Creating test database for alias 'default'...
System check identified no issues (0 silenced).
.
----------------------------------------------------------------------
Ran 1 test in 0.001s

OK
Destroying test database for alias 'default'...
[pipeline] test_exit_code=0

```
</validated_execution>
