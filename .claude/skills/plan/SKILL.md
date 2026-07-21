---
name: plan
description: "Plans the implementation of a feature, bugfix, refactor, migration, or ticket in a PHP/Symfony project. Analyzes context, clarifies unknowns, identifies files, steps, tests, risks, validation commands, and execution order without modifying code before validation."
---

# Implementation Planning

## Scope

Determine whether the user is asking for a plan for:

- Business feature
- Bugfix
- Refactor
- Doctrine or data migration
- API endpoint or DTO contract
- Twig/form/back-office interface
- Symfony command
- Messenger message or workflow
- PHP/Symfony/dependency upgrade
- Security/performance audit or targeted fix

If the user asks only for a plan, do not modify code.

If the scope is ambiguous, ask only blocking questions. For non-blocking points, state explicit assumptions.

## Objective

Produce an executable, ordered plan adapted to the project.

The plan must:

- Respect existing conventions.
- Keep the project compilable at each step when possible.
- Separate code, config, migration, fixtures, and tests.
- Identify risks before implementation.
- Avoid over-engineering.
- Provide useful validation commands.

Do not turn the plan into an unrequested broad refactor.

## Current State

Inspect according to the topic:

- `composer.json`: PHP version, Symfony, key dependencies, scripts
- Architecture: MVC, hexagonal, CQRS, API Platform, Twig, Messenger, legacy PHP/Symfony
- Files close to the scope: entities, repositories, services, handlers, controllers, forms, templates, DTOs
- Configuration: routes, services, security, messenger, doctrine, serializer, validator
- Existing tests and `tests/` conventions
- Doctrine migrations and schema if data is impacted
- Fixtures if functional or integration tests are needed
- CI, PHPStan, Rector, linters if relevant

Never read or modify `.env.local`.

## Open Questions

List only:

- Business decisions required to avoid a wrong implementation.
- Technical choices that strongly change architecture or cost.
- Information without which the plan would be misleading.

For the rest, state assumptions:

- Assumed PHP/Symfony version
- Chosen architecture
- Assumed API or UI format
- Expected compatibility or BC level
- Assumed data volume

## Plan Structure

### 1. Summary

One sentence that restates the need for validation.

### 2. Existing Code Analysis

Impacted files and classes, already used patterns, identified constraints.

### 3. Implementation Plan

For each step: file to create or modify, what changes and why, dependencies with other steps. Order steps so the code compiles at each intermediate step.

### 4. Files to Create

Full path, responsibility, namespace, and architectural layer.

### 5. Files to Modify

Full path, nature of the modification, impact risk.

### 6. Tests to Write

Unit tests, functional tests, edge cases to cover.

### 7. Risks and Points of Attention

Impact on existing behavior, required migrations, configuration changes, backward compatibility.

### 8. Complexity Estimate

Number of impacted files, relative complexity (simple / moderate / complex), possible prerequisites.

## Architecture Analysis

Adapt the plan to the project's style:

- Classic Symfony MVC: controller, form, Twig, repository, service if needed.
- JSON API: input/output DTO, validation, handler/service, structured response and errors.
- API Platform: resource, provider, processor, input/output DTO if project convention.
- Hexagonal/CQRS: separate Domain, Application, Infrastructure.
- Messenger: immutable message, idempotent handler, transport, retry/failure.
- Legacy Symfony/PHP 7.x: annotations/YAML/XML, no PHP 8 attributes, no readonly/enums.
- PHP 8+ / recent Symfony: attributes and modern types only if compatible with the project.

Do not introduce a new architectural style if the project already has a clear convention.

## Step Breakdown

Order steps to reduce risks:

1. Adapt or create the minimal data model.
2. Add migration and data strategy if needed.
3. Add services/handlers/use cases.
4. Add HTTP, CLI, Messenger, or UI interface.
5. Add validation, security, and error handling.
6. Add fixtures if useful.
7. Add targeted tests.
8. Run validations.

For a data migration or incompatible change, propose a multi-step deployment if needed.

## Risks to Analyze

- **BC break**: public signature, API contract, payload, route, event, message.
- **Data**: destructive migration, backfill, constraints, rollback.
- **Security**: permissions, IDOR, CSRF, sensitive field exposure, server-side validation.
- **Performance**: N+1, pagination, batch, index, external calls.
- **Concurrency**: transaction, lock, double processing, idempotency.
- **Messenger**: retry, failure transport, dispatch before commit, incompatible messages.
- **UX**: form errors, empty states, redirects, flash messages.
- **Tests**: critical behavior not covered, missing fixtures, overly fragile test.

## Test Strategy

Plan according to the change:

- Unit: value objects, services, business rules, simple handlers.
- Functional: controller, form, Twig, routes, security, redirects.
- Integration: repository, Doctrine, Messenger, real services.
- API: valid/invalid payload, HTTP codes, errors, serialization.
- Migration: schema validate, dry-run, rollback if available.
- Performance: large cases, pagination, batch.
- Regression: bugfix with a test that fails before the fix.

State required fixtures and avoid mocking the database for important Doctrine behavior.

## Technical Validation

Propose only commands adapted to the project:

- `vendor/bin/phpunit --filter TestName`
- Composer test script if present
- PHPStan on the modified scope
- `bin/console lint:container`
- `bin/console lint:twig templates/`
- `bin/console lint:yaml config/`
- `bin/console doctrine:schema:validate`
- `bin/console doctrine:migrations:migrate --dry-run`
- `bin/console debug:router`
- `bin/console debug:messenger`

Do not run destructive commands, real migration, purge, or massive retry in a plan.

## Rules

- Do not propose over-engineering.
- Respect existing project patterns.
- Report refactoring opportunities without including them unless explicitly requested.
- Do not add a new bundle, abstraction, or architecture without clear benefit.
- Do not mix feature, broad refactor, and upgrade unless explicitly requested.
- Do not hide important uncertainty: list it as a decision to validate.
- Do not assume a recent PHP/Symfony version without proof.
- Do not propose a solution that bypasses security, validation, or tests.

## Output Format

Present the plan with:

1. **Summary**: need restated in one sentence.
2. **Assumptions**: assumed choices if not confirmed.
3. **Existing Context**: observed files, conventions, and constraints.
4. **Step-by-step Plan**: order, file, change, reason, dependencies.
5. **Files to Create**: path, responsibility, layer.
6. **Files to Modify**: path, nature of change, risk.
7. **Tests**: tests to write or adapt, required fixtures.
8. **Validation**: commands to run.
9. **Risks**: impacts and mitigations.
10. **Complexity**: simple, moderate, or complex, with short justification.
11. **Decisions to Validate**: only if needed before implementation.
