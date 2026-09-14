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

</issue_statement>
<candidate_a_patch>
diff --git a/django/db/backends/base/operations.py b/django/db/backends/base/operations.py
index 0fcc607bcf..7e1622c072 100644
--- a/django/db/backends/base/operations.py
+++ b/django/db/backends/base/operations.py
@@ -558,6 +558,39 @@ class BaseDatabaseOperations:
         second = self.adapt_datetimefield_value(second)
         return [first, second]
 
+
+    def iso_year_lookup_bounds_for_date_field(self, value):
+        """
+        Return a two-elements list with the lower and upper bound to be used
+        with a BETWEEN operator to query a DateField value using an iso_year
+        lookup.
+
+        `value` is an int, containing the looked-up iso_year.
+        """
+        first = datetime.date(value, 1, 4)
+        first = first - datetime.timedelta(days=first.weekday())
+        second = datetime.date(value, 12, 28)
+        second = second + datetime.timedelta(days=6 - second.weekday())
+        return [first, second]
+
+    def iso_year_lookup_bounds_for_datetime_field(self, value):
+        """
+        Return a two-elements list with the lower and upper bound to be used
+        with a BETWEEN operator to query a DateTimeField value using an
+        iso_year lookup.
+
+        `value` is an int, containing the looked-up iso_year.
+        """
+        first = datetime.datetime(value, 1, 4)
+        first = first - datetime.timedelta(days=first.weekday())
+        second = datetime.datetime(value, 12, 28, 23, 59, 59, 999999)
+        second = second + datetime.timedelta(days=6 - second.weekday())
+        if settings.USE_TZ:
+            tz = timezone.get_current_timezone()
+            first = timezone.make_aware(first, tz)
+            second = timezone.make_aware(second, tz)
+        return [first, second]
+
     def get_db_converters(self, expression):
         """
         Return a list of functions needed to convert field data.
diff --git a/django/db/models/functions/datetime.py b/django/db/models/functions/datetime.py
index 90e6f41be0..92e688696c 100644
--- a/django/db/models/functions/datetime.py
+++ b/django/db/models/functions/datetime.py
@@ -6,6 +6,7 @@ from django.db.models.fields import (
     DateField, DateTimeField, DurationField, Field, IntegerField, TimeField,
 )
 from django.db.models.lookups import (
+    IsoYearExact, IsoYearGt, IsoYearGte, IsoYearLt, IsoYearLte,
     Transform, YearExact, YearGt, YearGte, YearLt, YearLte,
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
index 916478d075..d053c74724 100644
--- a/django/db/models/lookups.py
+++ b/django/db/models/lookups.py
@@ -597,6 +597,44 @@ class YearLte(YearLookup, LessThanOrEqual):
         return (finish,)
 
 
+class IsoYearLookup(YearLookup):
+    def year_lookup_bounds(self, connection, year):
+        output_field = self.lhs.lhs.output_field
+        if isinstance(output_field, DateTimeField):
+            bounds = connection.ops.iso_year_lookup_bounds_for_datetime_field(year)
+        else:
+            bounds = connection.ops.iso_year_lookup_bounds_for_date_field(year)
+        return bounds
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

</candidate_a_patch>
<candidate_b_patch>
diff --git a/django/db/backends/base/operations.py b/django/db/backends/base/operations.py
index 0fcc607bcf..1c6c5c9386 100644
--- a/django/db/backends/base/operations.py
+++ b/django/db/backends/base/operations.py
@@ -558,6 +558,49 @@ class BaseDatabaseOperations:
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
+        # Find the Monday of the first week (January 4 is always in ISO week 1)
+        first = first - datetime.timedelta(days=first.weekday())
+        # The last day is the day before the first day of the next ISO year
+        first_next_year = datetime.date(value + 1, 1, 4)
+        first_next_year = first_next_year - datetime.timedelta(days=first_next_year.weekday())
+        second = first_next_year - datetime.timedelta(days=1)
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
+        # Find the Monday of the first week (January 4 is always in ISO week 1)
+        first = first - datetime.timedelta(days=first.weekday())
+        # The last day is the day before the first day of the next ISO year
+        first_next_year = datetime.datetime(value + 1, 1, 4)
+        first_next_year = first_next_year - datetime.timedelta(days=first_next_year.weekday())
+        second = first_next_year - datetime.timedelta(days=1)
+        if settings.USE_TZ:
+            tz = timezone.get_current_timezone()
+            first = timezone.make_aware(first, tz)
+            second = timezone.make_aware(second, tz)
+        first = self.adapt_datetimefield_value(first)
+        second = self.adapt_datetimefield_value(second)
+        return [first, second]
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
index 916478d075..1868dd99b2 100644
--- a/django/db/models/lookups.py
+++ b/django/db/models/lookups.py
@@ -597,6 +597,66 @@ class YearLte(YearLookup, LessThanOrEqual):
         return (finish,)
 
 
+class IsoYearLookup(Lookup):
+    def year_lookup_bounds(self, connection, year):
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
+            start, finish = self.year_lookup_bounds(connection, self.rhs)
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
  "test_name": "test_iso_year_lookup_bounds_for_date_field",
  "specification_gap": "ISO-year lookup bounds must cross calendar-year boundaries when necessary and, like ordinary year lookup bounds, must be returned in the backend-adapted representation rather than as raw Python date objects.",
  "input_description": "Call BaseDatabaseOperations.iso_year_lookup_bounds_for_date_field(2015). ISO year 2015 begins on Monday, 2014-12-29, and ends on Sunday, 2016-01-03.",
  "expected_output": "The method returns the two backend-adapted date values ['2014-12-29', '2016-01-03']. candidate_b applies adapt_datefield_value() to both bounds; candidate_a returns datetime.date objects instead.",
  "expected_passing_candidate": "candidate_b",
  "why_interesting": "This checks a backend compatibility invariant of the optimized public lookup path while also exercising an ISO year whose bounds lie in three calendar years. Raw temporal parameters may not be accepted or interpreted identically by every database driver.",
  "test_patch": "diff --git a/tests/backends/base/test_operations.py b/tests/backends/base/test_operations.py\n--- a/tests/backends/base/test_operations.py\n+++ b/tests/backends/base/test_operations.py\n@@ -65,6 +65,12 @@ class SimpleDatabaseOperationTests(SimpleTestCase):\n         value = timezone.now().date()\n         self.assertEqual(self.ops.adapt_unknown_value(value), self.ops.adapt_datefield_value(value))\n \n+    def test_iso_year_lookup_bounds_for_date_field(self):\n+        self.assertEqual(\n+            self.ops.iso_year_lookup_bounds_for_date_field(2015),\n+            ['2014-12-29', '2016-01-03'],\n+        )\n+\n     def test_adapt_unknown_value_time(self):\n         value = timezone.now().time()\n         self.assertEqual(self.ops.adapt_unknown_value(value), self.ops.adapt_timefield_value(value))\n",
  "test_command": "python tests/runtests.py backends.base.test_operations.SimpleDatabaseOperationTests.test_iso_year_lookup_bounds_for_date_field"
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
      "duration_seconds": 1.612,
      "log_path": "02_execution/attempt_01/candidate_a.log"
    },
    {
      "label": "candidate_b",
      "returncode": 0,
      "passed": true,
      "test_ran": true,
      "duration_seconds": 1.542,
      "log_path": "02_execution/attempt_01/candidate_b.log"
    }
  ]
}
</candidate_execution_summary>
<candidate_a_execution_log>
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
Testing against Django installed in '/testbed/django' with up to 24 processes
System check identified no issues (0 silenced).
F
======================================================================
FAIL: test_iso_year_lookup_bounds_for_date_field (backends.base.test_operations.SimpleDatabaseOperationTests)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/testbed/tests/backends/base/test_operations.py", line 69, in test_iso_year_lookup_bounds_for_date_field
    self.assertEqual(
AssertionError: Lists differ: [datetime.date(2014, 12, 29), datetime.date(2016, 1, 3)] != ['2014-12-29', '2016-01-03']

First differing element 0:
datetime.date(2014, 12, 29)
'2014-12-29'

- [datetime.date(2014, 12, 29), datetime.date(2016, 1, 3)]
+ ['2014-12-29', '2016-01-03']

----------------------------------------------------------------------
Ran 1 test in 0.000s

FAILED (failures=1)
[pipeline] test_exit_code=1

</candidate_a_execution_log>
<candidate_b_execution_log>
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
Testing against Django installed in '/testbed/django' with up to 24 processes
System check identified no issues (0 silenced).
.
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
  "duration_seconds": 1.424,
  "log_path": "gold_execution/gold.log"
}
</gold_execution_summary>
<gold_failure_contract>
{
  "kind": "behavioral_test_failure",
  "required_substrings": [
    "test_iso_year_lookup_bounds_for_date_field"
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
  "reference_log_sha256": "0a4b8c50941e128c55354ecd860990b37208ecad8fc1b0de06ccf2aa6478f2fd"
}
</gold_failure_contract>
<gold_execution_log>
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
E
======================================================================
ERROR: test_iso_year_lookup_bounds_for_date_field (backends.base.test_operations.SimpleDatabaseOperationTests)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/testbed/tests/backends/base/test_operations.py", line 70, in test_iso_year_lookup_bounds_for_date_field
    self.ops.iso_year_lookup_bounds_for_date_field(2015),
AttributeError: 'BaseDatabaseOperations' object has no attribute 'iso_year_lookup_bounds_for_date_field'

----------------------------------------------------------------------
Ran 1 test in 0.000s

FAILED (errors=1)
Testing against Django installed in '/testbed/django' with up to 24 processes
System check identified no issues (0 silenced).
[pipeline] test_exit_code=1

</gold_execution_log>
