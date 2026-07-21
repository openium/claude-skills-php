---
name: fixtures
description: "Generates and structures Doctrine fixtures for PHP/Symfony projects. Produces realistic, deterministic data adapted to tests, with Alice, DoctrineFixturesBundle, Foundry, or the existing format, while respecting relations, constraints, security, and performance."
---

# Fixture Generation

## Scope

If the user specifies an entity, test, business scenario, API, command, or directory, limit fixtures to that need.

Otherwise:

- Identify existing fixtures and their format.
- Read the affected Doctrine entities.
- Read the tests or endpoints that will consume this data if context is given.
- Ask for the expected scenario if the need is unclear: minimal dataset, functional scenario, validation, demo, local seed, or volume.

Do not generate fixtures for the whole model without an explicit need.

## Current State

Before generating or modifying fixtures, inspect according to the project:

- `composer.json`: DoctrineFixturesBundle, Alice, HautelookAliceBundle, Foundry, Faker
- Existing directories: `src/DataFixtures/`, `fixtures/`, `tests/DataFixtures/`, `tests/fixtures/`
- Reference, group, factory, story, or processor conventions
- Doctrine entities: fields, types, nullability, relations, cascades, unique constraints, enums
- Symfony Validator constraints (`#[Assert\...]`)
- Functional or integration tests that load fixtures

Never read `.env.local`.

## Process

1. Read the target entity and its Doctrine annotations/attributes.
2. Identify validation constraints (`#[Assert\...]`).
3. Detect relations (ManyToOne, OneToMany, ManyToMany).
4. Determine the format used in the project.
5. Define the covered scenario and required references.
6. Generate fixtures in the existing format.
7. Propose or run targeted validation if possible.

## Format Detection

Inspect the project to identify the fixture system in place:

- `fixtures/*.yaml` or `*.yml`: Alice (hautelook/alice-bundle or fidry/alice-data-fixtures)
- `src/DataFixtures/*.php`: DoctrineFixturesBundle
- `tests/fixtures/` or `tests/DataFixtures/`: dedicated test fixtures
- `src/Factory/`, `tests/Factory/`, `Story`: Zenstruck Foundry

Priority:

1. Use the format already present in the project.
2. If several formats coexist, follow the format of the fixtures closest to the test or target entity.
3. If no format is in place, propose DoctrineFixturesBundle for simple PHP fixtures, or Foundry if the project already uses factories elsewhere.
4. Do not install a bundle without an explicit request.

## Dataset Types

### Minimal Dataset

- Data strictly necessary for a test or scenario.
- Few objects, explicit and stable references.
- Prefer for fast functional tests.

### Business Scenario

- Coherent data that tells a use case: active user, paid order, overdue invoice, etc.
- Reference naming based on the scenario.
- Complete and readable relations.

### Validation Fixtures

- Invalid data only in a separate file, group, or provider.
- Do not mix invalid fixtures with the default loaded dataset.
- Document the expected error if the fixture is used to test a constraint.

### Volume Fixtures

- Generate only on explicit request.
- Separate from default fixtures.
- Plan batch, periodic flush, and possible SQL logger disabling if the project does so.
- Do not load 10k objects in standard functional tests.

## Rules

### Realistic Data

- French first and last names (no "John Doe").
- Valid emails consistent with names.
- Dates in realistic ranges (no dates in year 3000).
- Phone numbers in French format.
- French addresses.
- SIRET, intra-community VAT if relevant.
- Plausible business data: statuses, amounts, currencies, periods, slugs, internal references.
- No real personal data, secrets, tokens, real passwords, or API keys.

### Determinism

- Tests must remain reproducible.
- Prefer explicit values for critical scenarios.
- If Faker is used, set a seed when the framework allows it.
- Avoid uncontrolled relative dates (`now`, `+3 days`) in sensitive assertions.
- Use a controlled clock or fixed dates when behavior depends on time.
- Generate emails, slugs, and references uniquely and predictably.

### Coverage

- Minimum 5 entries per entity.
- Vary cases: normal values, boundary values, special cases.
- Include at least one case with optional fields set to null.
- Include at least one case with long values (near VARCHAR limits).
- Do not overproduce data if the test needs only one or two records.
- Cover important statuses or domain transitions if the scenario uses them.

### Relations

- Create referenced entities first.
- Use explicitly named references.
- Cover cases: relation present, null relation (if nullable), multiple relations.
- For DoctrineFixturesBundle, use `DependentFixtureInterface` if a fixture depends on references created elsewhere.
- For Alice, use `@reference_name` references and readable names.
- For a required ManyToOne, create the target before the source.
- For bidirectional OneToMany, check that the add method maintains both sides of the relation.
- For ManyToMany, create at least one empty case if allowed and one case with several relations.
- Do not rely on a missing cascade persist: explicitly persist required entities.

### Validation Constraints

- Each fixture must respect the entity's `#[Assert\...]` constraints.
- Respect DB constraints: `NOT NULL`, unique, length, enum, foreign key.
- Create invalid fixtures only in a separate group, file, or test.
- For unique constraints, generate explicitly distinct values.
- For passwords, use the project hasher or a known test hash, never a real password.
- For files, use dedicated test files and not local absolute paths.

### Security and Environments

- Never target or document loading fixtures in production.
- Do not read `.env.local`.
- Do not include secrets, tokens, cookies, credentials, or real personal data.
- Do not load destructive fixtures without explicit confirmation.
- For local or test loading, specify the expected environment (`--env=test` if relevant).

### Performance

- Avoid a `flush()` after each entity unless explicitly needed.
- Use a final flush for small datasets.
- For significant volume, flush and clear by batch.
- Avoid network calls, external I/O, or slow services in fixtures.
- Keep default fixtures fast to load.

## Alice Format (YAML)

```yaml
App\Entity\User:
    user_admin:
        email: 'admin@example.com'
        firstName: 'Marie'
        lastName: 'Dupont'
        roles: ['ROLE_ADMIN']
    user_{1..5}:
        email: '<email()>'
        firstName: '<firstNameFemale()>'
        lastName: '<lastName()>'
        roles: ['ROLE_USER']
```

Alice rules:

- Use explicit references for important entities.
- Keep Faker expressions readable and compatible with the version used.
- Avoid unbounded randomness for validated or unique fields.
- Split files by domain or scenario if the project already does so.

## DoctrineFixturesBundle Format (PHP)

```php
public function load(ObjectManager $manager): void
{
    $user = new User();
    $user->setEmail('admin@example.com');
    $user->setFirstName('Marie');
    $user->setLastName('Dupont');
    $manager->persist($user);

    $this->addReference('user_admin', $user);
    $manager->flush();
}
```

DoctrineFixturesBundle rules:

- Implement `DependentFixtureInterface` for dependencies between fixtures.
- Use `addReference()` and `getReference()` with stable names.
- Use the password hasher if authenticatable users are created.
- Avoid calling complex business services from fixtures unless the project convention is clear.
- For grouped fixtures, respect `FixtureGroupInterface` if the project uses it.

## Foundry Format

If Foundry is already present:

- Use existing factories.
- Create or modify a `Story` for reusable scenarios.
- Keep states readable: `UserFactory::new()->admin()`, `OrderFactory::new()->paid()`.
- Prefer explicit overrides for important assertions.
- Do not introduce Foundry into a project that does not use it without an explicit request.

## Useful Commands

Adapt commands to the project:

- `bin/console doctrine:fixtures:load --env=test`
- `bin/console doctrine:fixtures:load --group=name --env=test`
- `bin/console doctrine:schema:validate`
- `vendor/bin/phpunit --filter TestName`
- Hautelook Alice or Foundry commands if they exist in the project

Do not run a destructive command without explicit confirmation, especially fixture loading that purges the database.

## Output Format

For generation or modification, provide:

- Files created or modified
- Chosen format and reason
- Covered scenarios or entities
- Created references and important relations
- Validation or DB constraints considered
- Validation commands run or not run
- Remaining limitations or assumptions

Generate the complete fixture file, ready to use, with imports, references, dependencies, and groups needed for relations.
