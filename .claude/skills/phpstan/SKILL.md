---
name: phpstan
description: "Fixes PHPStan errors in a PHP/Symfony project. Analyzes a report, file, or project; interprets typing, generics, Doctrine, and Symfony errors; and applies fixes without @phpstan-ignore, without lowering the level, and without worsening the baseline."
---

# Fixing PHPStan Errors

## Scope

If the user provides a PHPStan error, fix that error first.

If the user specifies a file or directory, limit the analysis to that scope.

Otherwise, use the project's PHPStan command if it exists. Do not install PHPStan without an explicit request.

## Current State

Before fixing, inspect according to the project:

- `phpstan.neon`, `phpstan.neon.dist`, and included files
- `composer.json`: scripts, PHP version, PHPStan dependencies
- Current PHPStan level, analyzed paths, Symfony/Doctrine/PHPUnit extensions
- Any baseline and custom rules

Do not modify the level, `ignoreErrors`, the baseline, or analyzed paths to make an error disappear.

## Process

1. Reproduce the error or read the exact PHPStan message.
2. Identify the file, line, expected type, and actually possible type.
3. Read called code and public usages before modifying a signature.
4. Fix the root cause: type, control flow, validation, initialization, or contract.
5. Rerun PHPStan on the targeted scope if possible.
6. Run targeted tests if runtime behavior may be impacted.

## Absolute Rules

- NEVER add `@phpstan-ignore-*` to hide an error.
- NEVER lower the PHPStan level in configuration.
- NEVER add an entry to `ignoreErrors` in `phpstan.neon`.
- Each fix must solve the real problem, not the symptom.
- Do not add `mixed`, `array`, or `object` if a precise type is available.
- Do not make something nullable only to satisfy PHPStan.
- Do not add a lying `@var`.
- Do not change business behavior to satisfy typing.
- Do not delete dead code without checking indirect Symfony, Doctrine, Serializer, or reflection usages.

## Common Fixes

### Method Call on a Nullable Type

- Do not use `?->` as the default solution.
- If null is impossible, fix the contract or add an explicit guard.
- If null is legitimate, handle the case with an exception, early return, or clear business branch.

### Incorrect Return Type

- Add the correct return type.
- If several types are real, use a union or extract a result object.
- Do not widen to `mixed` to avoid understanding the flow.

### Uninitialized Property

- Initialize in the constructor when the value is required.
- Add a default value only if it has business meaning.
- Mark nullable only if absence of value is a valid state.

### Dead Code

Delete dead code if confirmed. Check that there is no magic (reflection, container) before deleting.

### Generics and Templates

- Add `@template`, `@extends`, `@implements`, or `@use` when the class is truly generic.
- Type Doctrine collections: `Collection<int, Entity>`.
- Type repositories: `ServiceEntityRepository<Entity>` or equivalent.
- Type iterables: `iterable<int, Foo>` or `Generator<int, Foo, mixed, void>`.

### Arrays and Shapes

- Prefer a DTO when the structure is business-related or crosses several layers.
- Use an array shape for a local and stable structure.
- Check `isset`, `array_key_exists`, or validation before accessing an optional key.
- Use `non-empty-string`, `positive-int`, `class-string<T>` only if the code really guarantees that constraint.

### Callables and Closures

- Describe signatures with `callable(Input): Output` or a dedicated interface.
- Type parameters and returns of closures passed to `array_map`, `filter`, collections, or promises.

## Symfony and Doctrine

- Values from `Request`, `InputInterface`, `ParameterBag`, env vars, or config must be validated or converted.
- After `Repository::find()`, handle the `null` case before using the entity.
- Do not use `/** @var Entity $entity */` after a nullable `find()` without a real guard.
- Type collections, repositories, query builders, and query results.
- For services, prefer the existing interface when it represents the real contract.
- Do not fetch a service from the container to work around a type error.

## PHPDoc and Annotations

PHPDoc is acceptable for:

- Generics, templates, array shapes, callable signatures
- Doctrine types that cannot be expressed in native PHP
- Local precision after real runtime validation

PHPDoc must not lie about a value possible at runtime. Prefer native types when they are enough.

## Validation

- Rerun PHPStan on the modified file or scope if possible.
- If the full command is too costly, run the smallest useful scope.
- If PHPStan cannot be run, explain why and state the command to execute.
- Run targeted tests when a fix touches behavior, not only types.

## Output Format

Summarize:

- Initial PHPStan errors fixed
- Modified files
- Root cause
- Applied fix
- PHPStan commands run and result
- Tests run if relevant
- Remaining risks or errors
