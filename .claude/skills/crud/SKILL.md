---
name: crud
description: "Generates or completes a Symfony CRUD adapted to the project. Produces Doctrine entity, repository, migration, Twig/API controller, form, DTOs, handlers, templates, tests, and fixtures according to the existing architecture, while respecting security, validation, PHP/Symfony versions, and local conventions."
---

# Symfony CRUD Generation

## Scope

Determine whether the user is asking for:

- A complete CRUD for a new resource.
- A CRUD for an existing Doctrine entity.
- A Twig web, JSON API, API Platform, back-office, or EasyAdmin CRUD.
- A targeted action: create, read, update, delete, listing, search, export.
- A fix or extension of an existing CRUD.

If the need is ambiguous, ask for the expected interface type and editable fields.

Do not generate a complete CRUD if the user asks only for one action or endpoint.

## Current State

Before generating, inspect according to the project:

- `composer.json`: PHP, Symfony, Doctrine, Twig, Form, Validator, Security, API Platform, EasyAdmin versions
- Architecture: classic MVC, API-only, API Platform, hexagonal, CQRS, back office
- Existing entities, repositories, DTOs, handlers, forms, controllers, and templates
- Routing: PHP attributes, annotations, YAML, or XML
- Security: `security.yaml`, voters, `#[IsGranted]`, access control, existing roles
- Doctrine: migrations, mapping conventions, custom types, relations, indexes
- Tests: `tests/` structure, fixtures, base classes, naming conventions
- Templates: layout, components, partials, pagination, existing design system

Never read or modify `.env.local`.

## Process

1. Understand the requested entity (fields, types, relations, constraints).
2. Detect the project architecture and PHP/Symfony version.
3. Identify local conventions before creating new files.
4. Define the CRUD type: Twig, JSON API, API Platform, back office, CQRS.
5. Generate the required files in the right order.
6. Add or propose migration, fixtures, and targeted tests.
7. Run adapted validations if possible.

## Context Detection

- Twig templates present? -> classic web CRUD
- API Platform installed? -> API Platform Resource
- JSON controllers only? -> API CRUD without Twig
- Hexagonal architecture? -> Separate Domain/Application/Infrastructure
- EasyAdmin installed? -> Back-office CRUD through EasyAdmin dashboard/controller
- Existing DTOs/handlers? -> Do not hydrate the entity directly from the request
- Legacy PHP 7.x / old Symfony project? -> Follow annotations/YAML/XML and avoid PHP 8 attributes
- PHP 8+ / recent Symfony project? -> Attributes, readonly, enums, or modern types only if already compatible

## Generated Files

Adapt the list to the real context. Do not create useless files for the requested CRUD type.

### 1. Doctrine Entity

- Mapping according to project convention: PHP 8+ attributes, annotations, YAML, or XML
- Validation constraints (`#[Assert\NotBlank]`, etc.)
- Relations if requested
- Business methods (no blind setters in hexagonal architecture)
- Coherent Doctrine types: string length, text, decimal, datetime immutable, enum if supported
- Sensitive fields excluded from forms and public outputs by default
- Association methods that maintain both sides of bidirectional relations

### 2. Repository

- Extends `ServiceEntityRepository`
- `save()` and `remove()` methods with optional flush
- Specific search methods if identifiable
- Paginated or filtered queries if listing, search, or sorting is requested
- No business logic in the repository
- PHPDoc/generics types if PHPStan or project conventions use them

### 3. Migration

- Generate or report the required migration after entity modification
- Check `NOT NULL`, default values, unique constraints, indexes, and foreign keys
- Add an index on columns used in `WHERE`, `JOIN`, `ORDER BY`
- Avoid unrequested destructive operations
- For an existing entity, do not assume the table is empty

### 4. Twig Controller

- Routes according to project convention: attributes, annotations, YAML, or XML
- Actions: `index`, `show`, `new`, `edit`, `delete`
- Access control on each action: `#[IsGranted]`, `denyAccessUnlessGranted`, voter, or `access_control`
- Flash messages in French
- Redirect after creation/modification/deletion
- Mandatory CSRF on deletion and mutating actions
- Explicit `404` handling through nullable repository or param converter according to convention
- No heavy business logic in the controller

### 5. JSON API Controller

- Input/output DTO if the project uses them or if the endpoint is externally exposed
- Server-side validation before modification
- Coherent HTTP statuses: `201`, `200`, `204`, `400`, `403`, `404`, `422`
- Access control by action and ownership check if needed
- Do not directly hydrate a Doctrine entity from an external request
- Map input DTO to entity through handler, service, factory, or processor

### 6. API Platform

- Respect project conventions: resource on entity or dedicated class
- Use provider/processor if logic exceeds trivial CRUD
- Input/output DTO to protect the public contract if needed
- Security expressions, voters, or processors for permissions
- Normalization/denormalization groups only if already a project convention

### 7. Form Type

- Fields typed with the right FormType: TextType, DateType, EnumType, EntityType, FileType, MoneyType, etc.
- Labels in French
- Validation constraints coherent with the entity
- Options (placeholder, required, help)
- Exclude non-editable fields: id, owner, createdAt, updatedAt, sensitive roles, system status
- Handle upload, enums, relations, and optional fields explicitly
- Coherent `data_class`: entity or DTO according to architecture

### 8. Twig Templates (if web)

- `index.html.twig`: paginated list with table
- `show.html.twig`: entity details
- `new.html.twig`: creation form
- `edit.html.twig`: modification form
- `_form.html.twig`: shared form partial between new and edit
- `_delete_form.html.twig`: deletion form with confirmation
- Extend the existing layout
- Use the project's components, CSS classes, and Twig conventions
- Escape data by default, avoid `|raw` unless content is already sanitized
- Display actions according to permissions if the project convention does so
- Plan empty state, pagination, sorting, or search if requested

### 9. DTOs, Handlers, and Services

- Input DTO for create/update if API or application architecture
- Output DTO for public contract if API or sensitive serializer
- Application handler/service to apply modifications
- Factory if entity creation has invariants
- Voter if permissions depend on object or owner

### 10. Tests

- Unit test for the entity: validation, invariants, business methods
- Functional Twig controller test: routes, HTTP codes, redirects, forms, CSRF
- API test: valid payload, invalid payload, `404`, `403`, `422`, deletion
- Security test: anonymous access, insufficient role, IDOR/owner
- Repository test if custom query, pagination, or complex filter
- Minimal fixtures adapted to tested scenarios

### 11. Fixtures

- Create only data needed for tests or scenarios
- Respect relations, unique constraints, and validation
- Use the format already present: DoctrineFixturesBundle, Alice, Foundry, or dedicated fixtures

## Conventions

- Route naming: `app_entity_action` (ex: `app_user_index`, `app_user_new`)
- Template naming: `entity/action.html.twig`
- Flash messages: `success`, `error`
- Pagination: KnpPaginatorBundle if installed, otherwise manual pagination
- Namespaces, suffixes, and directories according to existing conventions
- Do not introduce a new style if the project already has one
- For PHP 7.x, do not generate attributes, enums, readonly, or PHP 8 syntax
- For old Symfony, respect annotations/YAML/XML if that is the convention

## Security

- Every sensitive action must have access control.
- For owner-based resources, check access to the object, not only the role.
- Protect Twig deletions with CSRF.
- Do not expose sensitive fields in APIs, templates, or forms.
- Do not trust IDs sent by the user for owner, organization, or tenant.
- Log or handle critical actions properly if the project already does so.

## Validation

- Business rules must live in the entity, domain service, or handler, not only in the form.
- `Assert` constraints must match DB constraints.
- Handle optional fields, nullable fields, and default values explicitly.
- For relations, validate that the related entity exists and that the user may use it.
- For uploads, validate size, real MIME type, extension, and storage.

## Do Not

- Do not generate a complete CRUD without an explicit need.
- Do not bypass business methods with blind setters in domain architecture.
- Do not directly hydrate a Doctrine entity from a sensitive external request.
- Do not add admin access or a mutating route without access control.
- Do not delete or modify existing already executed migrations without confirmation.
- Do not modify `.env.local`.
- Do not introduce a dependency, bundle, or new architecture without an explicit request.
- Do not create a public API that exposes the whole entity by default.

## Useful Commands

Adapt to the project:

- `bin/console make:entity` or `make:crud` only if the project uses it and the user accepts scaffolding
- `bin/console doctrine:migrations:diff`
- `bin/console doctrine:migrations:migrate --dry-run`
- `bin/console doctrine:schema:validate`
- `bin/console lint:twig templates/`
- `bin/console lint:container`
- `vendor/bin/phpunit --filter TestName`
- PHPStan on the modified scope if present

Do not run a real migration, DB purge, or destructive command without explicit confirmation.

## Output Format

For generation or modification, provide:

- Chosen architecture: Twig, JSON API, API Platform, back office, CQRS
- Files created or modified
- Required entity, relations, constraints, and migration
- Security applied by action
- Chosen validation and mapping
- Tests and fixtures added or proposed
- Commands run and result
- Remaining assumptions or limitations

Generate each file with its full path, ready to use, and state the creation order: entity, migration, repository, DTO/handler, controller, form/templates, fixtures, tests.
