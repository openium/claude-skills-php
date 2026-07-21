---
name: upgrade
description: "Assists PHP/Symfony and Composer dependency upgrades. Analyzes current versions, deprecations, breaking changes, and conflicts, then produces or applies a step-by-step migration plan. Never upgrades PHP without explicit validation."
---

# PHP/Symfony Version Upgrade

## Scope

Determine whether the user is asking for:

- An upgrade diagnosis or plan: do not modify code.
- Applying a migration: proceed through verifiable steps.
- A targeted scope: PHP, Symfony, Composer dependency, bundle, config, or application code.

If the target version is not given, analyze the current state and ask for the target before modifying.

## Current State

Inspect according to the project:

- `composer.json`, `composer.lock`, `symfony.lock`
- Required PHP version, `config.platform.php`, PHP extensions
- `symfony/*` packages, third-party bundles, Doctrine, PHPUnit, PHPStan, Rector
- `config/bundles.php`, `config/packages/`, `config/routes/`
- `phpunit.xml*`, `phpstan.neon*`, `rector.php`
- Dockerfile, `docker-compose*.yml`, GitHub/GitLab CI, Composer scripts

Never read or modify `.env.local`.

## Strategy

- Prefer an incremental migration: latest stable minor before a major change.
- Fix deprecations before a major Symfony change.
- Keep `symfony/*` packages coherent on the same target version.
- Handle Composer conflicts and third-party bundles first, then application code.
- Separate upgrade, refactor, and functional change.
- Do not modify `composer.lock` by hand.

## PHP Version

Never upgrade PHP without explicit validation.

If Symfony or a dependency requires a higher PHP version:

- Identify the minimum required PHP version.
- Check where PHP is pinned: `composer.json`, `config.platform.php`, Docker, CI, hosting.
- Explain the impact on the runtime environment.
- Ask for confirmation before any PHP-related modification.

Do not introduce enums, readonly, `#[\Override]`, or other recent syntax only because the target version supports them.

## Symfony

Check:

- Annotations replaced by PHP 8+ attributes
- Classes, methods, options, and services marked `@deprecated`
- Deprecated or moved YAML configuration
- Flex recipe and generated file changes
- Signature, event, security voter, authenticator, serializer, form, validation changes
- Third-party bundle compatibility with the target version

Compare Flex recipes carefully and keep project adaptations.

## Composer and Dependencies

- Use `composer why-not` to explain a version blocker.
- Use `composer update --dry-run` before a risky update.
- Do not force an incompatible constraint without understanding the blocking package.
- Do not remove a bundle without checking its usages in code, config, and templates.
- Check `composer audit` if available.

## Rector

If Rector is already present (`rector.php`, `rector/rector` dependency, or dedicated Composer script), it can be used for mechanical migrations.

Rules:

- Do not install Rector without an explicit request.
- Run Rector in dry-run first if possible.
- Limit the scope if the user requested a targeted migration.
- Reread the generated diff before considering the migration valid.
- Reject a Rector change that modifies business behavior.
- Run tests after applying changes.

## Useful Commands

Run only commands adapted to the project:

- `composer outdated`
- `composer why-not vendor/package version`
- `composer update --dry-run`
- `composer audit`
- `bin/console debug:container --deprecations`
- `vendor/bin/rector process --dry-run` if Rector is present
- Targeted tests, full suite, PHPStan if present

Report commands run, their result, and blockers.

## Do Not

- Do not make an unrequested major jump.
- Do not hide or ignore deprecations.
- Do not mix major upgrade and broad refactor.
- Do not change business behavior to satisfy an upgrade.
- Do not remove a package or bundle without proof that it is no longer used.
- Do not modify local environment files or secrets.

## Output Format

For a diagnosis:

- Current state: PHP, Symfony, key dependencies
- Requested or recommended target
- Composer blockers
- Identified deprecations and breaking changes
- Impacted files or packages
- Ordered execution plan

After applying:

- Modified files
- Updated dependencies
- Applied fixes
- Commands run
- Tests or checks
- Remaining risks or steps
