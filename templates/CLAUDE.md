# CLAUDE.md

Project instructions for Claude Code.

This file must stay short, concrete, and maintained with the project. It complements the specialized skills available in `.claude/skills/`.

## Project Context

- Project name: `<project-name>`
- Business domain: `<short description>`
- Application type: `<Symfony monolith, API, back office, worker, bundle, library>`
- Main environment: `<Docker, local PHP, VM, CI only>`
- Criticality level: `<low, medium, high>`

## Technical Stack

- PHP: `<7.0, 7.4, 8.1, 8.2, 8.3, ...>`
- Symfony: `<3.4, 4.4, 5.4, 6.4, 7.x, ...>`
- Database: `<MySQL, MariaDB, PostgreSQL, SQLite>`
- ORM: `<Doctrine ORM, Doctrine DBAL, other>`
- Frontend: `<Twig, Webpack Encore, Vite, React, Vue, none>`
- Queue / async: `<Symfony Messenger, RabbitMQ, Redis, SQS, none>`
- Observability: `<Monolog, Sentry, Blackfire, Datadog, New Relic, Grafana, none>`

## General Rules

- Answer in concise, actionable technical English.
- Prefer the project's existing conventions over generic choices.
- Read the code before proposing a modification.
- Do not assume a recent PHP or Symfony version without proof in the project.
- Respect legacy compatibility when the project targets PHP 7.x or an old Symfony LTS version.
- Before a significant modification, list the planned files and changes, then wait for validation if the request says so.
- Do not do opportunistic refactoring unrelated to the request.

## Security and Sensitive Data

- Never read, display, or commit `.env.local`.
- Never expose secrets, tokens, passwords, cookies, API keys, DSNs, or sensitive personal data.
- Never add realistic test credentials in code, fixtures, or documentation.
- Ask for confirmation before any destructive command: data deletion, irreversible migration, Git reset, database drop, shared cache purge.

## Modification Workflow

Before modifying:

1. Identify the exact need.
2. Read the affected files.
3. Check local conventions.
4. Propose a short plan if the change touches several files.

During the modification:

- Keep the diff minimal.
- Add or adapt tests when behavior changes.
- Preserve existing names, patterns, and abstractions.
- Avoid unrequested style changes.

After the modification:

- Summarize the modified files.
- State the commands run.
- Clearly report tests not run or impossible to run.
- Mention residual risks if needed.

## Project Commands

Adapt this section to the project.

```bash
# Install dependencies
composer install

# Run tests
vendor/bin/phpunit

# Static analysis
vendor/bin/phpstan analyse

# Code quality
vendor/bin/php-cs-fixer fix --dry-run --diff

# Doctrine migrations
bin/console doctrine:migrations:status
bin/console doctrine:migrations:migrate --dry-run
```

## Available Skills

Use the appropriate skill when the request matches:

- `/review` for a code review.
- `/debug` to diagnose an error or stacktrace.
- `/plan` to prepare an implementation before code.
- `/test` to create or fix tests.
- `/phpstan` to fix static analysis.
- `/security` for a security audit.
- `/performance` for slowness or load issues.
- `/doctrine` for entities, repositories, relations, transactions, and queries.
- `/migration` for Doctrine migrations.
- `/fixtures` for test datasets.
- `/dto` for DTOs and mappings.
- `/crud` to generate or modify a CRUD.
- `/messenger` for Symfony Messenger.
- `/twig` for Twig templates.
- `/docker` for the local environment.
- `/observability` for logs, traces, metrics, and alerting.

## PHP / Symfony Conventions

- Strict types: `<yes/no/by existing file>`
- Typing style: `<PHPDoc, native types, mixed>`
- Dependency injection: prefer constructor injection unless the project convention differs.
- Controllers: keep them thin, delegate business logic to services.
- Doctrine: avoid implicit queries inside loops and document transactional choices.
- DTO: do not expose entities directly if the project already uses DTOs.
- Exceptions: use existing business exceptions when they exist.

## Tests

- Framework: `<PHPUnit, Pest, Behat, Panther, other>`
- Location: `<tests/, tests/Unit, tests/Functional, ...>`
- Test fixtures: `<DoctrineFixturesBundle, Alice, Foundry, custom factories>`
- Test database: `<SQLite, PostgreSQL, MySQL, containers>`

When a bug is fixed, add a regression test if the project allows it.

## Git

- Do not modify files unrelated to the request.
- Do not revert user changes without an explicit request.
- Do not include IDE files, caches, logs, or local artifacts.
- Commits must be scoped and describe the real change.

### Branches

- `main` represents production code.
- `develop` represents code deployed to the development environment.
- Do not push directly to `main` or `develop`.
- Use a pull request to integrate a branch into `develop`.
- Use a pull request from `develop` to `main` to prepare a production release.
- Create working branches from `develop`, except `hotfix` branches, which start from `main`.

Recommended format:

```text
<type>/<request_id>_<short-name-in-kebab-case>
```

Common types:

- `feature` or `feat`: new feature.
- `bugfix` or `fix`: bug fix for a bug present on `develop`.
- `hotfix`: urgent bug fix for a bug present on `main`.
- `chore`: cleanup, maintenance, technical tasks without functional change.
- `update`: dependency, configuration, or existing code update.

Examples:

```text
feature/US-123_invoice-export
fix/TICKET-456_order-total
hotfix/TICKET-789_payment-error
chore/cleanup-obsolete-services
```

### Commits

- Use the Conventional Commits convention.
- Use the ticket or User Story reference as the scope.
- If no ticket exists, use a short scope describing the technical area.
- Minimal format:

```text
<type>(<ticket-or-scope>): <short description>
```

Examples:

```text
feat(US-123): add invoice export
fix(TICKET-456): handle refused transaction
chore(deps): update symfony packages
test(TICKET-789): add regression test
docs(readme): document local setup
```

Frequent types:

- `feat`: feature.
- `fix`: bug fix.
- `docs`: documentation.
- `style`: formatting without logical change.
- `refactor`: refactoring without functional change.
- `perf`: performance improvement.
- `test`: add or fix tests.
- `build`: build, dependencies, packaging.
- `ci`: continuous integration.
- `chore`: maintenance.
- `revert`: revert a commit.

### Pull Requests

- A pull request must have a clear and limited scope.
- The person who creates the pull request remains responsible for the merge.
- In a team, request a review from at least one other person.
- Do not merge a pull request marked WIP or depending on an unmerged sub-branch.
- Mention in the PR the verification commands run and known risks.

### Tags and Deployments

- Deployment tags are created automatically by CI/CD.
- Do not create or push a tag manually without an explicit request.
- A merge into `develop` may trigger deployment to development.
- A merge into `main` may trigger deployment to production.

### Guardrails for Claude

- Before a commit, check the staged and unstaged diff.
- Never commit `.env.local`, IDE files, caches, logs, dumps, or generated artifacts.
- Do not use `git reset --hard`, `git checkout --`, `git clean`, or a destructive command without explicit confirmation.
- Do not rewrite shared history without explicit confirmation.
- If unrelated changes are present, leave them out of the commit.

## Project Notes

Add important local decisions here:

- `<decision 1>`
- `<decision 2>`
- `<decision 3>`
