---
name: doctrine
description: "Audits and fixes Doctrine ORM/DBAL usage in a PHP/Symfony project. Analyzes entities, mappings, relations, repositories, QueryBuilder, transactions, UnitOfWork, cascades, indexes, performance, and consistency with migrations without being limited to migrations."
---

# Doctrine ORM/DBAL Audit

## Scope

Determine whether the user is asking for:

- An audit of a Doctrine entity, relation, repository, query, or service.
- A fix for a Doctrine error: mapping, query, transaction, flush, cascade, proxy.
- Design of a mapping or relation.
- Optimization of a query or loading strategy.
- Analysis of a diff that touches Doctrine.

If no scope is given:

- Read `git status`.
- Analyze modified Doctrine files: entities, repositories, migrations, services that use the EntityManager.
- Otherwise, ask for the scope.

Do not modify an already executed migration without explicit validation.

## Current State

Inspect according to the topic:

- `composer.json`: Doctrine ORM, DBAL, migrations, extensions, Symfony version
- `config/packages/doctrine.yaml`, mappings, custom types, naming strategy
- Affected entities, repositories, services, handlers, controllers, or normalizers
- Recent migrations if the schema changes
- Tests and fixtures that cover the behavior
- PHP/Symfony version for attributes, annotations, enums, or readonly

Never read `.env.local`.

## Severities

- **Blocking**: invalid mapping, data loss, inconsistent relation, missing transaction on critical writes, dangerous cascade, incorrect query, massive N+1.
- **Important**: missing index, excessive hydration, flush in a loop, repository carrying business logic, missing migration, inconsistent nullability.
- **Suggestion**: naming, PHPDoc/generics typing, query extraction, simplification, or convention alignment.

## Mapping and Entities

- Check consistency between PHP properties, Doctrine mapping, DB constraints, and Symfony validation.
- Respect the project convention: attributes, annotations, XML, or YAML.
- For PHP 7.x, do not introduce attributes, enums, or readonly.
- Initialize collections in the constructor.
- Prefer `DateTimeImmutable` if the project already uses it.
- Check `decimal`, `json`, `datetime`, `uuid`, enum, and value object types.
- Do not expose blind setters if the entity carries business invariants.
- Avoid application logic, network calls, or service access inside an entity.

## Relations

- Check owning side / inverse side.
- Maintain both sides of a bidirectional relation in `add/remove` methods.
- Define `nullable`, `onDelete`, cascade, and `orphanRemoval` explicitly according to the business rules.
- Do not use `cascade={"remove"}` or `orphanRemoval=true` without understanding the data impact.
- Add an index on FK columns used for search, join, or sorting.
- For ManyToMany, check join table, unique constraints, and deletion cases.
- For large relations, avoid loading the whole collection to count, filter, or paginate.

## Repositories and Queries

- Repositories contain queries, not business logic.
- Bind all DQL/SQL parameters.
- Avoid DQL/SQL concatenation with user data.
- Return a clear type: entity, typed list, scalar result, DTO projection, or paginator.
- Explicitly handle `null` after `find()` / `findOneBy()`.
- Avoid `findAll()` on large tables.
- Use pagination for lists.
- Check N+1 and lazy loads triggered by Twig, Serializer, or normalizers.
- For pure reads, consider scalar results, DTO projection, or a dedicated query.

## Transactions and Writes

- Use an explicit transaction for several consistent writes.
- Do not call `flush()` inside a loop unless the need is proven.
- For batches, use chunks and periodic `flush()` and `clear()`.
- Do not mix a long transaction with external HTTP/I/O calls.
- Check idempotency if the write is called from Messenger or a retry.
- Do not swallow a Doctrine exception that leaves the EntityManager closed.

## Migrations and Schema

- Every persisted entity modification must have a corresponding migration.
- Check nullability, default values, unique constraints, indexes, and foreign keys.
- Avoid destructive migrations without a data strategy.
- Do not assume the table is empty.
- For large tables, report lock risks and recommend a zero-downtime strategy if needed.

## Performance

- Identify N+1, excessive hydration, multiple collection joins, missing pagination.
- Check indexes for `WHERE`, `JOIN`, `ORDER BY`, unique constraints.
- Propose `EXPLAIN` / `EXPLAIN ANALYZE` if DB access is available.
- Do not optimize without proof or an explicit hypothesis.
- Do not hide a slow query behind a cache without an invalidation strategy.

## Tests and Validation

Plan according to risk:

- Unit test for an entity or value object invariant.
- Repository integration test with a real database.
- Functional test if Twig/API/Serializer triggers loading.
- Transaction or idempotency test for critical writes.
- Minimal fixtures consistent with constraints.

Useful commands:

- `bin/console doctrine:schema:validate`
- `bin/console doctrine:migrations:diff`
- `bin/console doctrine:migrations:migrate --dry-run`
- `vendor/bin/phpunit --filter TestName`
- PHPStan if repository/entity typing is present

Do not run a real migration, purge, or destructive command without explicit confirmation.

## Do Not

- Do not inject the EntityManager everywhere if a repository or application service is enough.
- Do not directly hydrate an entity from a sensitive external request.
- Do not ignore nullability issues between PHP, DB, and validation.
- Do not remove cascade, FK, or unique constraints without checking the data impact.
- Do not put heavy business logic in a repository or controller.
- Do not modify `.env.local`.

## Output Format

Present findings sorted by decreasing severity.

For each finding:

- File and line
- Severity: blocking, important, or suggestion
- Category: mapping, relation, repository, transaction, migration, performance, security
- Verified evidence
- Concrete impact
- Proposed fix
- Recommended validation test or command

If no issue is found, mention the limitations: DB not accessible, migrations not run, profiler absent, tests not executed.
