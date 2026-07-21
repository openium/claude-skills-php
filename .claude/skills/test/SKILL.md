---
name: test
description: "Generates PHPUnit tests for a PHP/Symfony file or class. Covers the happy path, edge cases, and error cases. Uses the Unit/Functional/Integration structure by default, adapts to existing projects by mirroring src/ into tests/, and centralizes mocks/stubs in tests/Mock."
---

# PHPUnit Test Generation

## Process

1. Read the target file to understand the logic.
2. Identify the adapted test type (unit or functional).
3. Analyze project conventions (tests/ structure, naming, base classes).
4. Generate the complete test.

## Conventions to Detect

Before generating, inspect the `tests/` directory to identify:

- Current structure: `tests/Unit/`, `tests/Functional/`, `tests/Integration/`, mirror of `src/`, or another existing convention
- Base class used: `TestCase`, `KernelTestCase`, `WebTestCase`, custom
- Method naming: `testCamelCase()`, or `#[Test]` attribute
- Data provider usage: `#[DataProvider('providerName')]`
- Project setup/teardown patterns
- Presence of `tests/Mock/` and test service configuration in `config/services.yaml` or `config/services_test.yaml`

## Test Structure

### Default Convention

If the project does not yet have a clear convention, use an explicit structure by test type:

Examples:

- `src/Service/AcmeService.php` -> `tests/Unit/Service/AcmeServiceTest.php`
- `src/Controller/UserController.php` -> `tests/Functional/Controller/UserControllerTest.php`
- `src/Repository/UserRepository.php` -> `tests/Integration/Repository/UserRepositoryTest.php`

The namespace follows the tests structure, for example `App\Tests\Unit\Service` for `tests/Unit/Service/AcmeServiceTest.php`.

### Adapting to Existing Code

- If the project already mirrors the `src/` structure in `tests/`, respect that architecture.
- Example: `src/Service/AcmeService.php` -> `tests/Service/AcmeServiceTest.php`.
- If several conventions coexist, follow the one used by tests closest to the target class.
- Do not move or rename existing tests without an explicit request.
- For a new project or a project without tests, apply the `tests/Unit/`, `tests/Functional/`, `tests/Integration/` convention.

## Rules

### Unit Tests (Domain classes, ValueObjects, Services without external dependency)

- Place in `tests/Unit/`, mirroring the `src/` structure, unless an existing convention differs.
- No dependency on the Symfony kernel.
- Mock interfaces, not concrete classes.
- Cover:
  - The happy path
  - At least one edge case (null, empty, extreme value)
  - At least one error case (expected exception)

### Functional Tests (controllers, commands, integration)

- Place in `tests/Functional/` for controllers and commands, or `tests/Integration/` for tests with kernel, database, or real services, unless an existing convention differs.
- Never mock the database.
- Use project fixtures if they exist.
- Test HTTP response codes, content, redirects.

### Mocks and Stubs

- Place reusable mocks, stubs, and fakes in `tests/Mock/`.
- Reproduce the same structure in `tests/Mock/` as in `src/`.
- Example: mock for `src/Client/AcmeClient.php` -> `tests/Mock/Client/AcmeClientMock.php`.
- Prefer a dedicated mock in `tests/Mock/` when it is shared by several tests.
- Keep mocks local to the test when they are simple and used only once.
- Symfony service mocks must be registered only in the test environment.

Expected configuration in `config/services.yaml` if the project does not already have equivalent configuration:

```yaml
when@test:
    services:
        App\Tests\Mock\:
            resource: '../tests/Mock/'
            autowire: true
            autoconfigure: true
```

Adapt the namespace if the project configuration uses a different tests namespace.

### Data Providers

Use a data provider when:

- More than 2 cases test the same method with different inputs.
- Edge cases are numerous (validation, parsing, conversion).

Format:

```php
#[DataProvider('exampleProvider')]
public function test_example(string $input, string $expected): void
{
    // ...
}

public static function exampleProvider(): iterable
{
    yield 'happy path' => ['input', 'expected'];
    yield 'edge case' => ['', ''];
}
```

### Assertions

- Prefer specific assertions (`assertSame` > `assertEquals` > `assertTrue`).
- One test = one behavior. No 15 assertions in a single test.
- Name test methods descriptively in camelCase: `testCreateUserWithInvalidEmailThrowsException`.
- Always declare `void` as the return type of test methods.

## Output Format

Generate the complete test file, ready to run, with imports, class, and test methods. If a shared mock is needed, also generate the file in `tests/Mock/` and the `when@test` configuration adjustment if absent.
