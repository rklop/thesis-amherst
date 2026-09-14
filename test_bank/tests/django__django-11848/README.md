# django__django-11848 tests

- [`test_parsing_rfc850_year_exactly_50_years_in_future`](django__django-11848--6e0adfcc2a66e2f5/) — accepted; must fail in the recorded way; Round 3
- [`test_parsing_rfc850_fifty_year_century_boundary`](django__django-11848--894f624df59c2116/) — rejected; must fail in the recorded way; Round 2
- [`test_parsing_rfc850_year_at_century_boundary`](django__django-11848--a67ba19a3d344aa5/) — accepted; must pass; Round 2
- [`test_parsing_rfc850_year_in_past`](django__django-11848--b13200b8ee25f2cb/) — accepted; must pass; Round 3
- [`test_parsing_rfc850_exactly_50_years_ahead`](django__django-11848--be8c43ec158290aa/) — rejected; must fail in the recorded way; Round 1
