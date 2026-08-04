---
name: docker
description: "Generates or audits a Docker environment for PHP/Symfony projects. Produces Dockerfile, docker-compose, web configs, DB/cache/queue/mail/assets services, Makefile, and validations, while respecting PHP/Symfony versions, security, performance, dev/test/CI, and existing conventions."
---

# Docker Environment for PHP/Symfony

## Scope

Determine whether the user is asking for:

- Creation of a complete local Docker environment.
- Audit or fix of an existing Dockerfile / docker-compose.
- Adding a service: DB, Redis, RabbitMQ, Mailpit, Node, worker, search.
- Dev, test, CI, or production environment.
- Targeted fix: permissions, PHP extensions, Xdebug, performance, healthcheck, volumes.

If the need is ambiguous, ask for the target environment: local dev, test, CI, or prod.

Do not overwrite an existing Docker configuration without analyzing its conventions.

## Reference

Base every recommendation on the official Symfony documentation: https://symfony.com/doc/current/setup/docker.html

- For a complete environment created from scratch, follow the setup the official doc recommends: the [symfony-docker](https://github.com/dunglas/symfony-docker) template, built on **FrankenPHP** (a single application server that embeds PHP and the web server, no separate PHP-FPM + Nginx/Caddy split). Use this as the default target unless the project already has a different working setup.
- If the project already has PHP installed locally and only needs auxiliary services, propose the hybrid approach described in the doc instead: run DB/Redis/RabbitMQ/Mailpit through Docker Compose and start the app with `symfony server:start`, which auto-detects the Docker services and exposes them as environment variables.
- Symfony Flex recipes automatically edit `compose.yaml` and `Dockerfile` when packages are installed (e.g. `composer require doctrine` adds a `database` service). This only works if the `###> recipes ###` / `###< recipes ###` markers are present and preserved in those files — never strip them.
- The current convention is `compose.yaml` (not `docker-compose.yml`); keep the existing file name if the project already uses one.
- Whether Flex generates Docker config on `symfony new`/package installs is controlled by `composer.json` → `extra.symfony.docker`; check it before assuming Docker support is present or absent.

## Current State

Inspect according to the project:

- `composer.json`: PHP version, required extensions, Composer scripts, Symfony
- `composer.lock` if present to confirm extensions and packages
- `.env` and `.env.dist` if present, without reading `.env.local`
- Dockerfile, `docker-compose.yml`, `compose.yaml`, overrides, and existing scripts
- `package.json`, lockfiles, Vite, Webpack Encore, AssetMapper, npm/yarn/pnpm
- Symfony config: Doctrine, Messenger, Mailer, Redis/cache, Mercure, search
- GitHub/GitLab CI if Docker must integrate with tests
- Legacy constraints: PHP 7.x, old extensions, existing Apache, old Symfony

Never read or display `.env.local` or secrets.

## Need Detection

From `composer.json`, `.env`, and configuration:

- PHP version, required extensions
- Database: `DATABASE_URL`
- Redis: `REDIS_URL`
- RabbitMQ: `MESSENGER_TRANSPORT_DSN` with `amqp://`
- Mailer: `MAILER_DSN`
- Node.js: `package.json` present
- Search: Elasticsearch, OpenSearch, Meilisearch according to packages/config
- Mercure: Mercure config or package
- Storage/files: uploads, S3/MinIO if package or explicit config
- Tests: dedicated database or `when@test` override

## PHP/Symfony Compatibility

- Align the PHP image with the project constraint, not the latest available version.
- For legacy PHP 7.x projects, use compatible images and extensions available for that version.
- Do not introduce syntax, tools, or images that force an unvalidated PHP upgrade.
- Respect the existing web server if the project already has Apache, Nginx, or Caddy.
- Check extensions required by Symfony and bundles: `intl`, `pdo_*`, `zip`, `opcache`, `amqp`, `redis`, `gd`, `imagick`, `sodium`, `mbstring`, `xml`, `curl`.

## Services

- **PHP application server**: prefer **FrankenPHP** (single image bundling PHP and the web server, per the official symfony-docker setup) for a new environment, with version aligned to composer.json, configurable Xdebug, and OPcache.
- If the project already runs **PHP-FPM** with a separate **Caddy**, **Nginx**, or **Apache**, keep that split instead of migrating it to FrankenPHP.
- **PostgreSQL/MySQL** according to DATABASE_URL, persistent volume
- **Redis**, **RabbitMQ**, **Mailpit** according to needs
- **Node** if package.json is present
- **Messenger Worker** optional, only if the project uses async transports
- **Search**: Elasticsearch/OpenSearch/Meilisearch only if necessary
- **Mercure** only if package/config is explicit

## Environments

### Local Dev

- Bind mounts for source code.
- Xdebug disableable by variable.
- OPcache configured for dev or disabled according to convention.
- Mailpit/Mailhog for emails.
- Ports exposed only for services useful locally.

### Test

- Isolated database or distinct database name.
- Test or in-memory Messenger transport if the project expects it.
- External services replaced by fakes/stubs if possible.
- Reproducible test commands through Makefile or Composer.

### CI

- Minimal and deterministic configuration.
- No dependency on specific local ports if avoidable.
- Composer/Node cache should be planned on the CI side rather than in local compose.
- `docker compose config`, build, and tests as separate steps.

### Production

- Do not generate a complete production configuration by default.
- If explicitly requested: immutable images, secrets managed outside git, non-root user, healthchecks, stdout/stderr logs, OPcache enabled, no Xdebug, no source volumes.

## Best Practices

- `.dockerignore`: exclude vendor/, node_modules/, var/, .git/
- Healthcheck on each service
- Environment variables in .env
- Makefile with common commands
- Official images or images already used by the project
- Multi-stage build if application or prod image is requested
- Optimized Composer and Node cache
- Install only required extensions
- `COPY composer.*` before `composer install` to benefit from Docker cache
- Do not automatically run migrations when the container starts unless this is an explicit convention
- Logs to stdout/stderr
- Readable service and network names

## Security

- Never commit secrets, prod credentials, tokens, or `.env.local`.
- Expose only necessary local ports.
- Avoid root at runtime if possible.
- Separate secrets and non-sensitive variables.
- Do not use `latest` for critical images if reproducibility matters.
- Volumes with controlled permissions for `var/`, cache, logs, uploads.
- Disable Xdebug outside dev.
- Do not include private keys or production dumps in the image.

## Performance

- Complete `.dockerignore` to reduce build context.
- Docker layers ordered to maximize cache.
- Composer install with flags adapted to dev or prod.
- OPcache enabled for prod or prod-like environment.
- Xdebug optional, never enabled by default in perf/prod.
- Volumes adapted to the OS if the project has bind mount slowness.
- Node dependencies managed with lockfile and cache if relevant.

## Doctrine and Messenger

- DB with healthcheck.
- Local persistent volume for DB, but isolated test database.
- Migrations run manually or through an explicit command.
- RabbitMQ/Redis only if async transports are detected.
- Separate Messenger worker if needed, with explicit command and limits if possible.
- Do not run an infinite worker without supervision or a clear command.

## Node and Assets

- Detect the package manager: npm, yarn, pnpm.
- Respect Vite, Webpack Encore, AssetMapper, or absence of Node build.
- For dev, propose a watcher if the project uses one.
- For prod/build image, compile assets in a dedicated stage if requested.
- Do not add Node if `package.json` is absent and assets are managed by AssetMapper without build.

## Makefile and Commands

Propose only adapted commands:

- `make up`
- `make down`
- `make build`
- `make shell`
- `make composer`
- `make console`
- `make test`
- `make logs`
- `make phpstan`
- `make fixtures`
- `make db-migrate`

Do not hide the real Docker commands: the Makefile must remain a readable shortcut.

## Validation

Propose or run according to context:

- `docker compose config`
- `docker compose build`
- `docker compose up -d`
- `docker compose ps`
- `docker compose exec php composer install`
- `docker compose exec php bin/console about`
- `docker compose exec php bin/console doctrine:schema:validate`
- Targeted tests if the project is ready

Do not run a real migration, DB purge, volume deletion, or massive download without explicit confirmation.

## Do Not

- Do not read `.env.local`.
- Do not overwrite an existing Dockerfile or compose without preserving useful conventions.
- Do not unnecessarily expose DB, RabbitMQ, Redis, or internal services.
- Do not introduce an unnecessary service.
- Do not impose FrankenPHP, Caddy, Nginx, Apache, Node, or a PHP version if the project indicates something else.
- Do not use production credentials.
- Do not automatically run migrations, fixtures, or workers at startup without an explicit request.
- Do not generate a production configuration while pretending it is ready for every hosting provider.
- Do not strip the `###> recipes ###` / `###< recipes ###` markers from `Dockerfile` or `compose.yaml`; Symfony Flex needs them to keep injecting service config on package install.

## Output Format

For generation or modification, provide:

- Files created or modified
- Generated services and reason
- Chosen PHP, DB, Node, and image versions
- Exposed ports
- Volumes and persistent data
- Environment variables to define, without secrets
- Useful Makefile or Docker commands
- Validation commands run or to run
- Assumptions, limitations, and security points

Generate all required files (`compose.yaml` or `docker-compose.yml`, Dockerfile, web configs, `.dockerignore`, Makefile if useful), ready to use with `docker compose up` in the requested context.
