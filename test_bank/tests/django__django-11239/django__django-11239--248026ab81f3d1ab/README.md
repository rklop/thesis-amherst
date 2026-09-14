# test_lazy_sslmode_updates_sslkey

- **Instance:** `django__django-11239`
- **Test ID:** `django__django-11239--248026ab81f3d1ab`
- **Status:** Rejected and not used in later generation
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 1

## What it checks

SSL connection options should be resolved coherently: evaluating a lazy SSL mode may refresh related certificate settings, and dbshell should use the resulting current client key rather than a value captured before that evaluation.

## Expected behavior

The psql subprocess environment contains PGSSLMODE=verify-ca and PGSSLKEY=client-current.key.

## Test command

`cd /testbed && python tests/runtests.py dbshell.test_postgresql.PostgreSqlDbshellCommandTestCase.test_lazy_sslmode_updates_sslkey`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
