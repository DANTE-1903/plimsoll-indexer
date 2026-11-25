# Deploying Plimsoll Indexer

This document details zero-downtime deployment and schema upgrade procedures for the Plimsoll Indexer.

## Zero-Downtime Schema Migrations

The indexer provides a `migrate` subcommand that wraps versioned database schema migrations:

```bash
plimsoll-indexer migrate
```

### Behavior
- Connects using `DATABASE_URL` from the environment or `.env` configuration.
- Checks `schema_migrations` table under an advisory lock (`pg_advisory_xact_lock`).
- Executes pending migrations inside a transaction.
- **Idempotency**: Running `migrate` multiple times applies nothing on subsequent invocations if the database is already up-to-date.
- **Exit codes**:
  - `0`: Migrations succeeded or no pending migrations to apply.
  - `1`: Failure during migration application (transaction rolls back cleanly).

### Deployment Procedure
1. Run `plimsoll-indexer migrate` before restarting or rolling out new service containers.
2. If `migrate` exits `0`, proceed with rolling deployment of application containers.
3. If `migrate` exits `1`, halt deployment and inspect database logs.
