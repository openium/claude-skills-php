---
name: deprecations
description: "Scans and fixes PHP and Symfony deprecations in a PHP/Symfony project, including legacy PHP 7.0+. Identifies obsolete code, removal version, project or vendor origin, and proposes replacements compatible with the target version without imposing an upgrade."
---

# Deprecation Detection

## Scope

Determine whether the user is asking for:

- Diagnosis only: do not modify code.
- Applying fixes: proceed through verifiable batches.
- A targeted scope: PHP, Symfony, Doctrine, bundle, configuration, tests, file, or directory.
- A target version: PHP 7.x, PHP 8.x, Symfony 2.8/3.4/4.4/5.4/6.4/7.x, or a precise dependency.

If no target version is given, analyze the current project version and report relevant deprecations without assuming an upgrade.

Never upgrade PHP, Symfony, or a dependency without explicit validation.

## Current State

Inspect according to the project:

- `composer.json`, `composer.lock`, `symfony.lock`
- Required PHP version, `config.platform.php`, Composer constraints, and PHP extensions
- Symfony version, third-party bundles, Doctrine, PHPUnit, PHPStan, Rector
- `phpunit.xml*`, `phpstan.neon*`, `rector.php`
- `config/bundles.php`, `config/packages/`, `config/routes/`
- Composer scripts for tests, static analysis, or deprecation helper
- CI, Dockerfile, or runtime configuration if the real PHP version is ambiguous

Never read or modify `.env.local`.

## Process

1. Identify current versions and possible target.
2. Reproduce or collect deprecations: logs, tests, `debug:container --deprecations`, PHPUnit, PHPStan, Rector dry-run if present.
3. Classify each deprecation: PHP, Symfony, Doctrine, PHPUnit, third-party bundle, project code, configuration.
4. Check the version where the API is removed or becomes incompatible.
5. Fix project code and configuration first, then propose a strategy for third-party dependencies.
6. Rerun the command that exposed the deprecation.
7. Run targeted tests if runtime behavior may be impacted.

Do not mix deprecation cleanup, major upgrade, and broad refactor unless explicitly requested.

## Priorities

- **Blocking**: API removed in the target version, deprecation that breaks tests with `SYMFONY_DEPRECATIONS_HELPER`, certain runtime incompatibility.
- **Important**: deprecation on a critical path, massive noise hiding other errors, fix needed before a near-term upgrade.
- **Suggestion**: useful modern replacement but not required for the current target version.

## PHP Deprecations

Always check the current PHP version and target version before fixing. For a legacy project, propose a replacement compatible with the minimum supported version.

### PHP 7.0

- PHP 4-style constructors (`function ClassName()`): use `__construct()`.
- `preg_replace()` with `/e` modifier removed: replace with `preg_replace_callback()`.
- Removed or obsolete extensions according to runtime: `mysql_*`, `ereg_*`, `mcrypt`, to handle carefully according to the real version.
- Methods or usages incompatible with scalar typing if the project starts typing gradually.

### PHP 7.1

- Function calls with too few arguments: fix the signature or call.
- `mcrypt` deprecated: migrate to `openssl` or `sodium` according to need.
- `each()` deprecated: replace with `foreach`.
- `create_function()` deprecated: replace with closure.

### PHP 7.2

- `__autoload()` deprecated: use `spl_autoload_register()`.
- `count()` on non-countable type: check type or use a guard.
- `assert()` with string: replace with boolean expression.
- `parse_str()` without second argument: provide an output array.

### PHP 7.3

- `FILTER_FLAG_SCHEME_REQUIRED` and `FILTER_FLAG_HOST_REQUIRED` deprecated.
- `case-insensitive constants` deprecated: use case-sensitive constants.
- Some old heredoc/nowdoc syntaxes may require normalization before upgrade.

### PHP 7.4

- Array/string access with braces: `$foo{0}` -> `$foo[0]`.
- `get_magic_quotes_gpc()` and magic quotes: remove dead branches.
- Old `implode($pieces, $glue)` order: use `implode($glue, $pieces)`.
- Typed properties only to introduce if compatible with the project's minimum version.

### PHP 8.0

- Method signatures incompatible with internal interfaces: align signatures.
- Required parameters after optional parameters: reorder or make the signature explicit.
- `match`, nullsafe, attributes available but do not introduce them if PHP 7.x compatibility must remain.
- Warnings transformed into `TypeError` or `ValueError`: fix inputs rather than hiding the error.

### PHP 8.1

- `FILTER_SANITIZE_STRING`, `strftime()`, `utf8_encode()`/`utf8_decode()`
- Implicit `null` return for internal methods incompatible: align types.
- Passing `null` to non-nullable internal parameters: add conversion or explicit guard.
- Serializable deprecated without `__serialize()` / `__unserialize()`.
- `DateTime` and `Intl`: check locale-dependent formats.

### PHP 8.2

- Dynamic properties, `${var}` string interpolation, partially supported callables
- `utf8_encode()` and `utf8_decode()` deprecated: use `mb_convert_encoding()` or an explicit charset strategy.
- `FILTER_SANITIZE_STRING` removed: replace with validation/normalization adapted to the context.
- `DateTimeInterface::ISO8601` to avoid depending on context: prefer `DateTimeInterface::ATOM` if the contract allows it.

### PHP 8.3

- `NumberFormatter::TYPE_CURRENCY`
- Check deprecations from `intl`, `mbstring`, `random`, `DateTime` extensions according to usages.
- Fix only if the target version or tests expose them.

### PHP 8.4

- Class constants without explicit visibility
- Implicitly nullable parameters through `Type $param = null` deprecated: use `?Type $param = null`.
- New internal extension deprecations: check project documentation and logs.

## Symfony Deprecations

Always check the current Symfony version, target version, and minimum PHP version before proposing a replacement. Legacy projects may require a fix compatible with annotations, YAML/XML, PHP 7.x, or old components.

### Symfony 6.4 LTS

- Fix deprecations before Symfony 7: removed configuration options, typed signatures, removed services or aliases.
- Check third-party bundles compatible with Symfony 7.
- Prefer attributes, explicit injection, and modern types if the project's minimum PHP version allows it.
- Check Security, Messenger, Serializer, Notifier, HttpClient, and Doctrine Bridge.

### Symfony 7.x

- Check removals from Symfony 6.4 deprecations.
- Replacements must remain compatible with the targeted minor version.
- Do not use APIs introduced in Symfony 7.1+ if the project targets Symfony 7.0.

- Annotations instead of PHP 8 attributes
- `ContainerAwareTrait` / `ContainerAwareCommand`
- `AbstractController::getDoctrine()`
- `@Route` annotation instead of `#[Route]`
- `EventSubscriberInterface` replaced by `#[AsEventListener]` (Symfony 6.2+)
- Services fetched directly from the container instead of explicit injection
- Renamed service aliases, parameters, or configuration options
- YAML config moved or renamed between Symfony versions
- Security: replaced guard authenticator, voters or access control to adapt according to version
- Forms: deprecated options, renamed types or extensions
- Validator: constraints, options, or annotations replaced by attributes according to target
- Serializer: normalizers, context options, and groups to check
- Routing: Doctrine annotations vs PHP attributes according to minimum PHP version
- Event Dispatcher: string event names replaced by classes or attributes according to component
- Doctrine Bridge: registry, annotations, mapping, and migrations to check

Do not automatically replace annotations with attributes if the project must remain PHP 7.x compatible.

## Doctrine, PHPUnit, and Tool Deprecations

- Doctrine annotations to attributes: only if PHP 8+ is validated.
- Doctrine DBAL: types, platforms, `executeUpdate()` to `executeStatement()`, replaced fetch methods.
- Doctrine ORM: annotations, proxy, mapping driver, repository signatures.
- PHPUnit: annotations to attributes only if the target PHP version allows it.
- PHPUnit assertions or hook signatures (`setUp`, `tearDown`) to align with installed version.
- PHPStan/Psalm: never add ignores to hide a deprecation.

## Dependencies and Vendor

- Never modify `vendor/`.
- If the deprecation comes from a third-party package, identify the package and version.
- Check whether a newer version fixes the deprecation.
- Use `composer why`, `composer why-not`, or `composer outdated` to explain the blocker.
- Propose an update, package replacement, or upstream issue if the project cannot fix directly.
- Do not remove a bundle without checking its usages in code, config, templates, and tests.

## Rector

If Rector is already present (`rector.php`, `rector/rector` dependency, or dedicated Composer script), it can help with mechanical migrations.

Rules:

- Do not install Rector without an explicit request.
- Run Rector in dry-run first.
- Limit the scope if the user requested a file, directory, or deprecation type.
- Reread the diff before considering the fix valid.
- Reject changes that modify business behavior.
- Do not introduce PHP syntax newer than the minimum supported version.

## Useful Commands

Adapt commands to the project:

- `bin/console debug:container --deprecations`
- `bin/console about`
- `composer outdated`
- `composer why vendor/package`
- `composer why-not vendor/package version`
- `vendor/bin/phpunit`
- `SYMFONY_DEPRECATIONS_HELPER=weak vendor/bin/phpunit`
- `SYMFONY_DEPRECATIONS_HELPER=max[self]=0 vendor/bin/phpunit`
- `vendor/bin/phpstan analyse`
- `vendor/bin/rector process --dry-run` if Rector is present

Report commands run and their result. Do not run costly or destructive commands without a clear reason.

## Do Not

- Do not hide or ignore deprecations.
- Do not change PHP, Symfony, or Composer version without explicit validation.
- Do not modify `.env.local` or secrets.
- Do not introduce attributes, enums, readonly, nullsafe, or other PHP 8 syntax in a project that supports PHP 7.x.
- Do not replace one API with another without checking the minimum version that introduces it.
- Do not perform broad refactoring to fix a local deprecation.
- Do not modify `composer.lock` by hand.
- Do not remove a package or bundle without proof it is no longer used.

## Output Format

For a diagnosis:

- Detected versions: PHP, Symfony, key dependencies
- Analyzed target or hypothesis used
- Deprecation list with file, line, origin, message, removal version
- Recommended replacement compatible with the project's minimum version
- Priority: blocking, important, suggestion
- Commands run and result
- Ordered correction plan

After correction:

- Modified files
- Deprecations fixed
- Replacements applied
- Commands rerun and result
- Tests executed or not executed
- Risks, remaining deprecations, or blocking third-party dependencies
