---
name: dto
description: "Generates and structures DTOs for PHP/Symfony projects. Produces typed, validated, readonly DTOs compatible with Serializer/API Platform/Messenger, with explicit mapping from entities and application to entities through a service, handler, or factory."
---

# DTO Generation

## Scope

If the user specifies an entity, endpoint, form, command, Messenger message, use case, or file, limit the work to that scope.

Otherwise:

- Identify existing DTOs and their conventions.
- Read the affected entity, controller, handler, form type, or serializer.
- Check the project architecture: classic MVC, API Platform, hexagonal, CQRS, Messenger.
- Ask for the use case if the DTO intent is unclear: input, output, update, query, command, or message.

Do not generate a generic DTO that exposes the whole entity without an identified need.

## Current State

Before generating or modifying DTOs, inspect according to the project:

- `composer.json`: PHP version, Symfony Validator, Serializer, API Platform, Messenger
- Existing DTOs: namespace, suffix, readonly, validation, mapping
- Related Doctrine entities: fields, nullability, relations, enums, value objects
- Controllers, handlers, processors, providers, forms, or commands that consume DTOs
- Existing validation rules and validation groups
- Serializer configuration if useful: groups, name converter, custom normalizers

Do not read `.env.local`.

## DTO Types

### Input DTO (creation/modification)

- `readonly` classes, constructor property promotion
- Validation constraints (`#[Assert\...]`)
- Contains only fields modifiable by the client
- Carries scalars, enums, simple value objects, or relation identifiers, not Doctrine entities
- Can be specialized by operation: `CreateUserDto`, `UpdateUserDto`, `ChangeUserEmailDto`
- For partial updates, distinguish absent field, field present as `null`, and field present with value if the API allows it

### Output DTO (read)

- No validation constraints
- Can contain computed or formatted fields
- Static `fromEntity()` method for mapping
- Exposes the expected public contract, not necessarily the whole entity
- Can aggregate several entities or computed values if the use case requires it
- Must not trigger uncontrolled hidden queries through lazy relations

### Command/Query DTO (CQRS)

- Command: modification intent
- Query: data request
- Immutable (readonly)
- Contains only the data needed by the use case
- Can be independent from HTTP, CLI, or Messenger transport

### Filter/Search DTO

- Represents search, sorting, pagination, and filter criteria.
- Validates bounds: positive page, maximum limit, allowed sort, status enum.
- Uses nullable values only if absence of criterion is a valid state.

### Message DTO (Messenger)

- Immutable and serializable message.
- Contains stable identifiers rather than Doctrine entities.
- Contains no service, resource, closure, non-serializable object, or unnecessary sensitive data.
- Remains compatible between two successive deployments if messages can remain queued.

### Form DTO

- Can decouple a Symfony form from the Doctrine entity.
- Carries validation constraints close to user input.
- Mapping to the entity must stay in the controller, a form handler, or an application service.

## Rules

- `readonly` classes (PHP 8.2+)
- No business logic in the DTO
- No dependency on Doctrine or Symfony (except Assert)
- Naming: `Create{Entity}Dto`, `Update{Entity}Dto`, `{Entity}Dto`
- Precisely typed properties: avoid `mixed`, `array`, or `object` if a precise shape exists
- Prefer native types, enums, and simple value objects over magic strings
- Use PHPDoc only for generics, array shapes, or types not expressible in native PHP
- Do not make nullable only to simplify mapping
- Do not expose a sensitive field by default: hashed password, token, secret, unnecessary personal data
- Do not reuse the same DTO for create, update, and output if contracts diverge

## Validation

- Put Symfony Validator constraints on input DTOs.
- Do not put user validation on output DTOs.
- Use `#[Assert\NotBlank]` for a required non-empty string, `#[Assert\NotNull]` for a required value that may be `0` or `false`.
- Add `#[Assert\Email]`, `#[Assert\Length]`, `#[Assert\Choice]`, `#[Assert\Regex]`, `#[Assert\Uuid]`, `#[Assert\Positive]` according to the real contract.
- For collections, use `#[Assert\All]`, `#[Assert\Count]`, and document the type: `@param list<string> $tags`.
- For nested objects, use `#[Assert\Valid]`.
- For dates, specify the expected type (`DateTimeImmutable`, ISO 8601 string, date only) and convert at the application boundary.
- For uploaded files, use `UploadedFile` only on the input/form side and validate size, real MIME type, and extension if needed.
- Use validation groups only if the project already uses them or if the create/update case clearly justifies them.

## Mapping

### Entity to Output DTO

A static `fromEntity()` method on the output DTO is acceptable for simple mapping:

```php
public static function fromEntity(User $user): self
{
    return new self(
        id: $user->getId(),
        email: $user->getEmail(),
        fullName: $user->getFullName(),
    );
}
```

Rules:

- The DTO may know the entity it exposes.
- Mapping must stay simple: field reads, light transformation, public format.
- Do not put queries, repository, EntityManager, or business rules in `fromEntity()`.
- For a list, plan an explicit factory or dedicated method if the project already does so.
- To avoid N+1, check that relations used by the DTO are loaded by the query.

### Input DTO to Entity

Avoid `toEntity()` or `applyToEntity()` in the input DTO.

Prefer:

- a factory to create an entity;
- an application handler to apply a command;
- a business method of the entity called by the service;
- an API Platform processor if the project uses it.

Example:

```php
final class UpdateUserHandler
{
    public function handle(User $user, UpdateUserDto $dto): void
    {
        $user->rename($dto->firstName, $dto->lastName);
        $user->changeEmail($dto->email);
    }
}
```

Rules:

- The DTO must not depend on Doctrine, a repository, or the container.
- Incoming relations must be represented by IDs or business codes.
- The handler resolves related entities, checks permissions, and applies business invariants.
- Do not directly hydrate a Doctrine entity from an external request.
- Do not bypass business methods of the entity with blind setters if the domain exposes explicit methods.

## Serializer and API

- Separate the public contract from the Doctrine model.
- Use serializer groups only if they are already the project convention.
- Do not rely only on serializer groups to protect sensitive fields: define an explicit output DTO if the API is sensitive.
- Keep field names stable for API clients.
- Plan an error DTO or coherent validation format if the endpoint exposes structured errors.
- For API Platform, respect project conventions: input/output DTO, provider, processor, resource class, or state options.

## Tests and Validation

Add or propose according to risk:

- Validation test for an input DTO: valid case, missing fields, invalid formats, limits.
- `fromEntity()` mapping test if the DTO contains computed fields, public formats, or relations.
- Handler/factory test for input DTO to entity mapping.
- Functional deserialization and validation test for an API endpoint.
- Regression test for sensitive fields not exposed.

Useful commands if present:

- `vendor/bin/phpunit --filter TestName`
- `bin/console lint:container`
- `bin/console debug:validator App\\Dto\\ExampleDto`
- PHPStan on the modified scope

## Output Format

For generation or modification, provide:

- Files created or modified
- DTO type and covered use case
- Validation constraints added
- Chosen mapping strategy
- Important typing choices
- Tests or validation commands run
- Remaining limitations or assumptions

Generate complete DTO classes with namespace, imports, types, validation, relevant mapping, and usage example in the handler, controller, or processor if needed.
