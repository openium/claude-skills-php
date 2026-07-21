# Claude Skills PHP & Symfony

Claude Code skills for PHP and Symfony developers. Each skill is a specialized prompt, versioned in the project, that the team can invoke with `/skill-name`.

## Installation

Copy the skills into the target project:

```bash
# One specific skill
cp -r .claude/skills/review /path/to/project/.claude/skills/

# All skills
cp -r .claude/skills /path/to/project/.claude/
```

Commit the `.claude/skills/` directory in the target project to make it available to the whole team.

## CLAUDE.md Template

A `CLAUDE.md` file template is available in [`templates/CLAUDE.md`](templates/CLAUDE.md).

It can be copied to the root of a PHP/Symfony project to document the project context, stack, useful commands, security rules, and expected workflow with Claude Code.

Recommended prompt to generate a `CLAUDE.md` tailored to a project:

```text
From the `templates/CLAUDE.md` template, generate a `CLAUDE.md` file at the root of this project.

Before writing the file:
- analyze the project structure;
- read the useful configuration files: `composer.json`, `composer.lock`, `symfony.lock`, `phpunit.xml*`, `phpstan*`, `.php-cs-fixer*`, `docker-compose*`, `Dockerfile*`, `Jenkinsfile`, `.github/workflows/*` if they exist;
- identify the PHP, Symfony, Doctrine versions and the tools actually used;
- identify the available commands to install, test, analyze, and run the project;
- infer the existing conventions without inventing rules that are not visible in the repository.

Then:
- replace the template placeholders with project information;
- keep the useful sections;
- remove examples or options that do not apply;
- keep the security, Git, and sensitive data rules;
- clearly state any information you could not determine.

Before modifying, list the planned changes and wait for my validation.
```

## Available Skills

### Code Quality

| Skill | Description |
|-------|-------------|
| [`/review`](.claude/skills/review/SKILL.md) | PHP/Symfony code review: bugs, security, architecture, tests, Doctrine, migrations. |
| [`/refactor`](.claude/skills/refactor/SKILL.md) | Safe refactoring without changing observable behavior. |
| [`/phpstan`](.claude/skills/phpstan/SKILL.md) | Fix PHPStan errors without ignores, baselines, or lowering the level. |
| [`/deprecations`](.claude/skills/deprecations/SKILL.md) | Detect and fix PHP/Symfony deprecations, including legacy projects. |
| [`/doctrine`](.claude/skills/doctrine/SKILL.md) | Doctrine ORM/DBAL audit: mappings, relations, repositories, transactions, performance. |

### Tests and Data

| Skill | Description |
|-------|-------------|
| [`/test`](.claude/skills/test/SKILL.md) | Generate unit, functional, or integration PHPUnit tests. |
| [`/fixtures`](.claude/skills/fixtures/SKILL.md) | Generate realistic and deterministic Doctrine/Alice/Foundry fixtures. |

### Security, Performance, and Observability

| Skill | Description |
|-------|-------------|
| [`/security`](.claude/skills/security/SKILL.md) | Security audit based on the OWASP Top 10 for PHP/Symfony. |
| [`/performance`](.claude/skills/performance/SKILL.md) | Performance audit: N+1, cache, Doctrine, Twig, HTTP, Messenger, batch. |
| [`/observability`](.claude/skills/observability/SKILL.md) | Logs, traces, metrics, alerting, Sentry/APM, and protection of sensitive data. |

### Code Generation

| Skill | Description |
|-------|-------------|
| [`/crud`](.claude/skills/crud/SKILL.md) | Symfony CRUD tailored to the project: Twig, JSON API, API Platform, DTOs, tests. |
| [`/dto`](.claude/skills/dto/SKILL.md) | Typed readonly DTOs, validation, entity/DTO mapping through handler or factory. |
| [`/command`](.claude/skills/command/SKILL.md) | Symfony Console commands with arguments, options, validation, progress bar, dry-run. |
| [`/api-doc`](.claude/skills/api-doc/SKILL.md) | OpenAPI/NelmioApiDocBundle documentation for Symfony endpoints. |

### Infrastructure and CI

| Skill | Description |
|-------|-------------|
| [`/docker`](.claude/skills/docker/SKILL.md) | PHP/Symfony Docker environment: PHP-FPM, web, DB, cache, queue, mail, assets. |

### Symfony

| Skill | Description |
|-------|-------------|
| [`/migration`](.claude/skills/migration/SKILL.md) | Doctrine migration audit: data loss, up/down, locks, zero-downtime. |
| [`/messenger`](.claude/skills/messenger/SKILL.md) | Symfony Messenger audit: transports, routing, retry, handlers, workers. |
| [`/twig`](.claude/skills/twig/SKILL.md) | Twig audit: security, accessibility, performance, forms, i18n, UX. |

### Planning and Onboarding

| Skill | Description |
|-------|-------------|
| [`/plan`](.claude/skills/plan/SKILL.md) | Implementation plan before code: steps, files, tests, risks. |
| [`/debug`](.claude/skills/debug/SKILL.md) | Diagnose an error, stacktrace, failing test, or unexpected behavior. |
| [`/upgrade`](.claude/skills/upgrade/SKILL.md) | PHP/Symfony/Composer dependency upgrade assistance. |
| [`/onboard`](.claude/skills/onboard/SKILL.md) | Technical onboarding guide for a PHP/Symfony project. |

## Invocation Examples

| Need | Skill | Example |
|------|-------|---------|
| Review a diff before commit | `/review` | `/review analyze the staged diff before commit` |
| Simplify a class without changing behavior | `/refactor` | `/refactor src/Service/InvoiceCalculator.php` |
| Fix PHPStan | `/phpstan` | `/phpstan fix this PHPStan error in UserRepository` |
| Prepare an upgrade | `/deprecations` | `/deprecations scan deprecations before Symfony 6.4` |
| Audit Doctrine outside migrations | `/doctrine` | `/doctrine check the mapping and relations of Order` |
| Generate tests | `/test` | `/test generate tests for src/Service/PriceCalculator.php` |
| Create fixtures | `/fixtures` | `/fixtures add a paid order scenario for functional tests` |
| Audit security | `/security` | `/security audit the modified controllers` |
| Diagnose slowness | `/performance` | `/performance analyze why this endpoint runs too many SQL queries` |
| Improve logs | `/observability` | `/observability add useful logs around the payment handler` |
| Generate a CRUD | `/crud` | `/crud create a Twig CRUD for Product with tests and migration` |
| Create DTOs | `/dto` | `/dto create input/output DTOs for the user creation endpoint` |
| Create a command | `/command` | `/command generate a CSV import command with dry-run` |
| Document an API | `/api-doc` | `/api-doc document the POST /api/orders endpoint` |
| Add Docker | `/docker` | `/docker generate a local environment with PostgreSQL, Redis, and Mailpit` |
| Audit a migration | `/migration` | `/migration check the latest Doctrine migration for zero-downtime` |
| Audit Messenger | `/messenger` | `/messenger check routing, retry, and failed transport` |
| Audit Twig | `/twig` | `/twig audit templates/order/show.html.twig` |
| Plan a feature | `/plan` | `/plan prepare the implementation of an invoice CSV export` |
| Debug an error | `/debug` | `/debug analyze this Symfony stacktrace` |
| Upgrade versions | `/upgrade` | `/upgrade prepare the move from Symfony 5.4 to 6.4` |
| Document a project | `/onboard` | `/onboard generate a guide for a new developer` |

## Structure Convention

Each skill is a directory in `.claude/skills/<name>/` with a required `SKILL.md` file.

```text
.claude/
└── skills/
    └── skill-name/
        └── SKILL.md
```

The directory name and the `name` field must be identical.

```yaml
---
name: skill-name
description: "Short and precise description of when to use this skill."
---
```

Recommended rules:

- Use a short lowercase name, with hyphens when needed.
- Write the skill in technical English, keeping native ecosystem terms such as `DTO`, `handler`, `dry-run`, `readonly`, `failure transport`.
- Start with a clear **Scope** section.
- Add a **Current State** section when the skill must inspect a project.
- Describe **Severities** for audit skills.
- List useful commands, but flag destructive commands to avoid without confirmation.
- Include a **Do Not** section for guardrails.
- End with an actionable **Output Format** section.
- Never ask to read `.env.local`.
- Never expose secrets, tokens, cookies, DSNs, passwords, or sensitive personal data.
- Respect legacy projects: do not assume PHP 8+, attributes, enums, readonly, or recent Symfony without proof.

## Contribution

To add or modify a skill:

1. Create or modify `.claude/skills/<name>/SKILL.md`.
2. Check that the front matter contains `name` and `description`.
3. Check that `name` matches the directory.
4. Add the skill to the README list.
5. Add at least one invocation example.
6. Reread the guardrails: secrets, `.env.local`, destructive commands, legacy compatibility.

A commit should ideally cover a single skill, except for a coordinated addition such as a new skill with the README.

## License

MIT

---

Maintained by [Efficience IT](https://www.itefficience.com) - PHP & Symfony Expertise
