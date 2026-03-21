Design schemas, generate migrations, review queries, or audit database conventions.

The target description or file path is: $ARGUMENTS

**Default database:** Use **PostgreSQL** for all database setups unless the project or `$ARGUMENTS` explicitly specifies a different engine. Do not substitute SQLite, MySQL, or another database without a stated reason.

## Modes

### Default — Schema design
When `$ARGUMENTS` describes entities or a data model:
1. Propose a normalized schema — output as `CREATE TABLE` SQL or ORM model definitions
2. Include: explicit foreign keys, indexes (all FKs + all query columns), constraints, and `created_at` / `updated_at` timestamps
3. Add `deleted_at` for any user-owned entity (soft delete)
4. Flag any deliberate denormalization with a comment explaining the trade-off
5. End with a **Design Decisions** section explaining key choices

### `--migrate` — Generate a migration
Given a description of a schema change in `$ARGUMENTS`:
1. Detect the existing migration tool (Alembic, Flyway, Django, Knex, etc.) by reading config files
2. Detect the naming convention from existing migration files
3. Generate a migration file with both `up` and `down` operations
4. Flag any destructive operation (column drop, type change) with a warning comment
5. Include a note on how to test the rollback

### `--review` — Review a query
Given a query in `$ARGUMENTS` or from the current file context:
1. Identify: N+1 risk, missing indexes, `SELECT *` usage, unbounded result sets
2. Suggest the `EXPLAIN ANALYZE` command to run in the container
3. Propose an optimized version with rationale

### `--audit` — Audit the database layer
Scan migration directories and model files for:
- Nullable columns without explanatory comments
- Foreign key columns missing indexes
- `SELECT *` in production query paths
- User-owned tables without a `deleted_at` column
- Migrations that lack a `down` operation
- PII columns without comments describing their encryption/hashing strategy

Produce a prioritized list grouped by severity: **Critical**, **Warning**, **Suggestion**.
