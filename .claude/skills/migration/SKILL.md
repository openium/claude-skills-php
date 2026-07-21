---
name: migration
description: "Analyzes Doctrine migrations in a PHP/Symfony project. Detects data loss, up/down inconsistencies, long locks, non-zero-downtime operations, missing indexes, and migrations incompatible with PostgreSQL, MySQL, or MariaDB."
---

# Doctrine Migration Check

## Scope

If the user specifies a migration, directory, diff, branch, or PR, analyze only that scope.

Otherwise:

- Read `git status`.
- Analyze modified migrations in `migrations/` or `src/Migrations/`.
- Otherwise, analyze the most recent migration.
- If no migration exists or the scope is ambiguous, ask for the file to check.

Do not modify a migration already executed in production without explicit validation. In that case, recommend a new corrective migration.

## Current State

Before concluding, inspect according to the project:

- Doctrine Migrations version and configuration (`doctrine_migrations.yaml`, `migrations_paths`)
- Target DBMS: PostgreSQL, MySQL, MariaDB, or other
- Doctrine entities related to the change
- Repositories, queries, controllers, DTOs, serializers, or templates that use the affected columns
- Existing constraints: foreign keys, unique constraints, indexes, `NOT NULL`, default values
- Potentially large data or critical tables

Never read `.env.local`. If the DBMS cannot be identified, state the hypothesis used.

## Process

1. Identify `up()` and `down()` methods, SQL queries, and Doctrine DBAL calls.
2. Classify each operation: creation, deletion, rename, type modification, constraint, index, backfill, massive update.
3. Check data loss, reversibility, locks, probable duration, and application compatibility.
4. Check consistency with entities and code that may still run during deployment.
5. Propose a fix or multi-migration strategy if needed.

## Severities

- **Blocking**: possible data loss, non-reversible migration when it should be reversible, probable application break, SQL invalid for the target DBMS, already executed migration modified.
- **Important**: probable long lock, non-zero-downtime operation, missing index on large table, unbounded massive backfill, incomplete `down()` acceptable only with justification.
- **Suggestion**: readability, grouping operations, useful comment, naming or robustness improvement without immediate risk.

## Analysis Criteria

### Data Loss (blocking)

- `DROP TABLE` without checking the table is empty or data was migrated
- `DROP COLUMN` without prior data migration
- `TRUNCATE` in a migration
- Column type modification that truncates data (VARCHAR(255) to VARCHAR(50))
- Removing a UNIQUE constraint that could create duplicates
- Removing a foreign key without checking concurrent application writes
- Table or column rename without compatibility period
- `UPDATE` that overwrites an existing value without a safety condition
- `DELETE` without `WHERE` clause or without backup/data migration

### Doctrine and Application Consistency (blocking)

- Missing migration while an entity modifies the schema
- Migration present but entity inconsistent with the final schema
- Removed column still referenced in code, a DQL/SQL query, serializer, form, or template
- `NOT NULL` constraint added while code can still write `NULL`
- New Doctrine relation without coherent foreign key, index, or cascade strategy
- Table, column, or index rename incompatible with the project's Doctrine conventions

### Up/Down Consistency (blocking)

- Missing `down()` method
- `down()` does not exactly reverse `up()`
- `down()` tries to recreate a deleted column without restoring data
- `down()` deletes data created before the migration
- Deletion order inconsistent with foreign keys
- Types, default values, nullability, constraints, or indexes not restored

If the operation is intentionally non-reversible, `down()` must contain an explicit exception or clear comment, and the risk must be reported.

### Performance (important)

- `ALTER TABLE` on a large table (> 1M rows) without duration estimate
- Adding an index on a large table without `CONCURRENTLY` (PostgreSQL) or equivalent
- Several `ALTER TABLE` statements on the same table instead of one
- Massive `UPDATE` without limiting `WHERE` clause
- Backfill without batch, limit, resumability, or idempotent condition
- Adding a `NOT NULL DEFAULT` column that may rewrite the whole table depending on DBMS/version
- Creating a foreign key or unique constraint without a suitable prior index
- Constraint validation on a large table without `NOT VALID` then `VALIDATE CONSTRAINT` phase when PostgreSQL allows it
- Long transaction combining DDL, backfill, and constraints on a critical table

### Zero-downtime (important)

- Column rename (breaks old code still in prod)
- Removing a column still referenced by code
- Adding a NOT NULL column without default value
- Column type modification
- Table rename
- Enum, check constraint, or allowed value change before deploying compatible code
- Adding a unique constraint without prior duplicate cleanup
- Migration assuming all application traffic is stopped
- Dependency on exact code + migration deployment at the same instant

## DBMS-specific Points

### PostgreSQL

- Use `CREATE INDEX CONCURRENTLY` and `DROP INDEX CONCURRENTLY` for large indexes when possible.
- Disable Doctrine transaction for concurrent indexes with `public function isTransactional(): bool { return false; }`.
- Prefer `ADD CONSTRAINT ... NOT VALID` then `VALIDATE CONSTRAINT` to reduce locks.
- Check locks for `ALTER TYPE`, `ALTER TABLE ... SET NOT NULL`, `DROP COLUMN`, `RENAME COLUMN`.
- Avoid destructive `USING` casts without data checks.

### MySQL and MariaDB

- Check `ALTER TABLE` impact according to engine, version, `ALGORITHM=INPLACE/INSTANT`, and `LOCK=NONE`.
- Anticipate table copy on type, index, or nullability modifications.
- Check index lengths with `utf8mb4`.
- Preserve exact column options in `down()`: type, collation, charset, default, nullable, unsigned.
- For large tables, recommend an online schema change strategy if the project has one.

## Recommended Zero-downtime Pattern

For destructive operations or operations incompatible with old code, recommend a multi-step deployment:

1. Migration A: add the new column/table/permissive constraint, without removing the old schema.
2. Deployment A: compatible code that reads old and new, or writes to both.
3. Backfill: migrate data by batch, idempotently and resumably.
4. Deployment B: code that no longer depends on the old schema.
5. Migration B: remove the old schema or make the constraint strict.

## Queries and Data

- Backfill queries must be idempotent.
- Migrations that modify business data must be explicitly justified.
- `UPDATE` and `DELETE` statements must have a precise `WHERE` clause or justification.
- Default values must be compatible with business rules and Symfony validation.
- Migrations must not depend on unstable application services, the Symfony container, or external data.

## Useful Commands

Adapt commands to the project, without installing a dependency unless explicitly requested:

- `bin/console doctrine:migrations:status`
- `bin/console doctrine:migrations:diff`
- `bin/console doctrine:migrations:migrate --dry-run`
- `bin/console doctrine:schema:validate`
- `bin/console doctrine:schema:update --dump-sql`
- Targeted tests for affected repositories, services, or endpoints

Report commands run, their result, and possible limitations.

## Do Not

- Do not consider a migration safe only because Doctrine generated it.
- Do not ignore inconsistent `down()` methods.
- Do not propose deleting data without a backup or prior migration strategy.
- Do not mix broad application refactoring and migration fix unless necessary.
- Do not modify a historized or already executed migration without confirmation.
- Do not hide a zero-downtime risk behind a simple comment.

## Output Format

Present findings first, sorted by decreasing severity.

For each finding:

- File and line
- Severity: blocking, important, or suggestion
- Affected SQL operation
- Evidence verified in the migration or code
- Concrete impact: data, availability, rollback, or application compatibility
- Proposed fix
- Zero-downtime strategy if applicable
- Recommended validation test or command

If no issue is found, say so clearly and mention the assumed DBMS, commands not run, and analysis limitations.
