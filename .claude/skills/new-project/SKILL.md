---
name: new-project
description: "Creates and initializes a new Symfony project from scratch. Asks the user for project type, PHP/Symfony version, database, templating, security, and tooling, then uses the Symfony CLI and Composer, following the official Symfony documentation, to scaffold, configure, and validate the project."
---

# New Symfony Project

## Scope

Use this skill when the user wants to:

- Create a brand-new Symfony project (web application, API, or minimal skeleton).
- Initialize an empty directory with Symfony, Composer, Git, and base configuration.
- Bootstrap a project before any business code exists.

Do not use this skill to add a feature, bundle, or CRUD to an existing project — see `/crud`, `/docker`, `/dto`, or other skills for that.

## Required Information

Before running any command, ask the user for the missing elements. Do not assume answers that materially change the generated project.

Ask, or infer with a stated assumption if strongly implied by context:

1. **Project name / directory**: name of the project and target path (current directory or a new subdirectory).
2. **Project type**:
   - Full-stack web application (`symfony new --webapp`, includes Twig, forms, security, ORM).
   - API-only project (`symfony/skeleton` + API Platform, or skeleton + plain JSON controllers).
   - Minimal skeleton (`symfony/skeleton`, add bundles later as needed).
3. **PHP version** available locally (`php -v`) and minimum PHP version to target.
4. **Symfony version**: latest stable, a specific LTS (e.g. 6.4, 7.x), or a version constraint. Confirm compatibility with the PHP version.
5. **Database and ORM**: PostgreSQL, MySQL/MariaDB, SQLite, none. Doctrine ORM or no persistence layer.
6. **Templating/front-end**: Twig, API-only (no templating), Webpack Encore / AssetMapper / no frontend tooling.
7. **Authentication/security**: none for now, `symfony/security-bundle` with a login form, JWT (LexikJWTAuthenticationBundle), API token.
8. **Testing**: PHPUnit (default), Panther, Behat if explicitly requested.
9. **Package manager for assets**: npm, yarn, pnpm, or none if AssetMapper/no frontend.
10. **Containerization**: Docker environment now (delegate to `/docker`) or not yet.
11. **Version control**: initialize Git, default branch name, initial `.gitignore`.
12. **License and PHP version constraint** in `composer.json` if the user cares about it.

If the user already answered some points in their initial request, do not ask again — only ask about what is still missing or ambiguous.

## Prerequisites Check

Before scaffolding, verify the local environment:

```bash
php -v
composer --version
symfony version
symfony check:requirements
```

- If the Symfony CLI (`symfony`) is not installed, fall back to `composer create-project` and mention the CLI as optional but recommended (local TLS server, `symfony check:requirements`, Docker/services helpers).
- If PHP or Composer is missing or below the required version, stop and report it instead of forcing an incompatible scaffold.
- Confirm the target directory is empty or does not already contain a Symfony project before writing into it.

## Reference

Base commands and options on the official Symfony documentation rather than on memory or habit:

- Setup: https://symfony.com/doc/current/setup.html
- Symfony CLI: https://symfony.com/doc/current/setup/symfony_server.html
- Full-stack vs microservice skeleton: https://symfony.com/doc/current/setup.html#choosing-symfony-version
- Security: https://symfony.com/doc/current/security.html
- Doctrine: https://symfony.com/doc/current/doctrine.html

If uncertain about a flag, a package name, or a current best practice, check the documentation or `composer show -a <package>` rather than guessing, especially since Symfony CLI options and recommended bundles change between versions.

## Scaffolding

Use the Symfony CLI when available:

```bash
# Full-stack web application
symfony new <project-name> --webapp --version="<constraint>"

# API-only / minimal skeleton
symfony new <project-name> --version="<constraint>"

# In the current empty directory instead of a new one
symfony new . --webapp
```

Composer fallback:

```bash
composer create-project symfony/skeleton <project-name> "<constraint>"
# or
composer create-project symfony/website-skeleton <project-name> "<constraint>"
```

After scaffolding:

```bash
cd <project-name>
symfony check:requirements
```

## Adding Bundles Based on Answers

Add only what the user asked for or what the chosen project type requires, one `composer require` per concern:

- Doctrine ORM + migrations: `composer require symfony/orm-pack doctrine/doctrine-migrations-bundle`
- API Platform: `composer require api`
- Security bundle (already included in `--webapp`, otherwise): `composer require symfony/security-bundle`
- JWT authentication: `composer require lexik/jwt-authentication-bundle`
- Twig (already included in `--webapp`): `composer require symfony/twig-bundle`
- Forms/Validator: `composer require symfony/form symfony/validator`
- Mailer: `composer require symfony/mailer`
- Messenger: `composer require symfony/messenger`
- Testing: `composer require --dev symfony/test-pack`
- Webpack Encore: `composer require symfony/webpack-encore-bundle` then `npm install`
- AssetMapper: `composer require symfony/asset-mapper symfony/asset symfony/twig-pack`

Do not install a bundle "for later" without the user asking for it.

## Database Configuration

If a database was requested:

- Set `DATABASE_URL` in `.env` matching the chosen engine (never write real credentials into a committed `.env`; use `.env.local` for that, and never read or display it).
- `bin/console doctrine:database:create`
- Generate a first entity only if the user explicitly asks for one at this stage; otherwise leave the schema empty.

## Git Initialization

If Git was requested and no `.git` directory exists:

```bash
git init
git add .
git commit -m "Initial Symfony project setup"
```

- Keep the default `.gitignore` generated by Symfony (`/.env.local`, `/var/`, `/vendor/`, `/node_modules/`, `/public/build/`).
- Never commit `.env.local` or secrets.
- If the directory already has Git history or a configured remote, check `git status` first and confirm with the user before committing.

## Docker

If the user wants a containerized environment, delegate to the `/docker` skill rather than duplicating that logic here; only mention that it is available.

## Validation

After scaffolding and configuration, run:

```bash
symfony check:requirements
composer validate --strict
bin/console about
bin/console debug:router
bin/console lint:container
bin/console lint:yaml config/
```

If a database was configured:

```bash
bin/console doctrine:database:create
bin/console doctrine:schema:validate
```

Start the local server only if the user asks to see the project running:

```bash
symfony server:start -d
# or
symfony server:start --no-tls
```

## Do Not

- Do not scaffold into a non-empty directory without confirmation.
- Do not assume Docker, a specific database, or API Platform without asking.
- Do not commit `.env.local` or any secret.
- Do not force a PHP or Symfony version incompatible with what `php -v` / `symfony check:requirements` reports.
- Do not run `git push`, create a remote repository, or force-push without explicit instruction.
- Do not install bundles that were not requested and are not required by the chosen project type.
- Do not silently downgrade or upgrade the PHP version constraint in `composer.json`.

## Output Format

Provide:

- Summary of the answers collected (project type, PHP/Symfony version, database, templating, security, testing, Docker, Git).
- Commands run, in order.
- Files and directories created or modified.
- Bundles installed and why.
- Environment variables to fill in (`.env` / `.env.local`), without secrets.
- Validation commands run and their result.
- Next steps (e.g. create the first entity, add Docker, add CI) left for the user to request explicitly.
