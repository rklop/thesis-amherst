# test_ssl_options_accept_mapping_get_requiring_default

- **Instance:** `django__django-11239`
- **Test ID:** `django__django-11239--d9ac6136171b1611`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 1

## What it checks

PostgreSQL runshell_db() should preserve its existing connection-parameter mapping contract when adding TLS options: each parameter is retrieved with an explicit empty fallback, allowing mapping implementations whose get() requires a default argument.

## Expected behavior

The psql subprocess environment contains PGSSLMODE=verify-ca, PGSSLROOTCERT=ca.crt, PGSSLCERT=client.crt, and PGSSLKEY=client.key. candidate_b instead calls get() with one argument for the TLS keys and raises TypeError before launching psql.

## Test command

`cd /testbed && ./tests/runtests.py dbshell.test_postgresql.PostgreSqlDbshellCommandTestCase.test_ssl_options_accept_mapping_get_requiring_default`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
