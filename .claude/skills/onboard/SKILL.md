---
name: onboard
description: "Generates a technical onboarding guide for a PHP/Symfony project. Analyzes the repo to document stack, local installation, architecture, workflows, commands, conventions, CI/CD, and verifiable points of attention for a new developer."
---

# Technical Onboarding

## Scope

By default, produce a guide for the whole project.

If the user specifies a module, directory, or objective, limit onboarding to that scope.

Do not modify the repository unless explicitly requested.

## Sources to Inspect

Read when present:

- `README*`, `docs/`
- `composer.json`, `composer.lock`, `symfony.lock`
- `Makefile`, `Taskfile*`, Composer scripts
- `Dockerfile`, `docker-compose*.yml`
- `.env`, `.env.dist`, `.env.example`
- `config/bundles.php`, `config/packages/`, `config/routes/`
- `src/`, `tests/`, `migrations/`, `templates/`
- `assets/`, `package.json`, `yarn.lock`, `pnpm-lock.yaml`, `package-lock.json`
- `.github/workflows/`, `.gitlab-ci.yml`

Never read or display `.env.local`.

## Rules

- Document only what is verifiable in the repo.
- Distinguish verified facts from points to confirm.
- Cite consulted source files when it helps understanding.
- Do not invent commands: prefer Makefile, Composer scripts, Docker Compose, or npm scripts actually present.
- If README, Docker, Composer, or CI contradict each other, report the discrepancy.
- Never display a secret. For environment variables, document only the name and role if verifiable.
- If a secret appears committed, report it without copying it.

## Guide Sections

### Overview

Project objective, application type, main components, and identifiable external services.

### Technical Stack

PHP version, Symfony, database, tools (Redis, RabbitMQ, etc.), main bundles.

### Local Installation

Prerequisites, dependency installation, environment configuration, application startup, Docker if present.

### Configuration

Documented environment variables, important configuration files, enabled bundles, main routes.

### Database

Database type, migrations, fixtures, creation/reset commands if verifiable.

### Architecture

`src/` directory structure, architectural pattern (MVC, hexagonal, DDD, CQRS), layers and responsibilities, naming conventions.

### Development Workflows

Dev commands, cache, assets, workers, Messenger, cron or scheduled tasks if present.

### Tests and Quality

Tests, linter, PHPStan, CS-Fixer, migrations, cache, assets, Makefile.

### Code Conventions

Coding standard (PSR-12, custom), test organization, git workflow, CI/CD.

### CI/CD

Pipelines, jobs, PHP versions, test, build, deployment steps if verifiable.

### Points of Attention

Technical debt areas proven by TODO/FIXME, contradictory documentation, sensitive files, project specifics, obsolete dependencies if verifiable.

## Output Format

Produce a factual and concise Markdown document.

Include:

- Useful commands in `sh` blocks
- References to consulted files when relevant
- "To Confirm" section for non-verifiable information
- "Documentation Gaps or Discrepancies" section if needed

Do not include secrets or `.env.local` content.
