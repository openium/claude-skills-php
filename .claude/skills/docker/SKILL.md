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

Base every recommendation on the official Symfony documentation: https://symfony.com/doc/current/setup/docker.html — but for a **new environment created from scratch**, the default target is the team's reference stack, not the official symfony-docker/FrankenPHP template: see [Reference Stack](#reference-stack-default-structure) below, based on the ready-to-copy files in `templates/docker/`.

- Default to a **PHP-FPM container + a separate Nginx container**, matching the structure already running in existing team projects (see Reference Stack). This keeps the same file layout, Makefile targets, and healthchecks across every dockerized project.
- Only propose the [symfony-docker](https://github.com/dunglas/symfony-docker) / **FrankenPHP** single-container template when the user explicitly asks for it, or the project already runs on it — do not switch a project that already made a stack choice.
- If the project already has PHP installed locally and only needs auxiliary services, propose the hybrid approach described in the official doc instead: run DB/Redis/RabbitMQ/Mailpit through Docker Compose and start the app with `symfony server:start`, which auto-detects the Docker services and exposes them as environment variables.
- Symfony Flex recipes automatically edit `compose.yaml` and `Dockerfile` when packages are installed (e.g. `composer require doctrine` adds a `database` service). This only works if the `###> recipes ###` / `###< recipes ###` markers are present and preserved in those files — never strip them.
- The current convention is `compose.yaml` (not `docker-compose.yml`); keep the existing file name if the project already uses one.
- Whether Flex generates Docker config on `symfony new`/package installs is controlled by `composer.json` → `extra.symfony.docker`; check it before assuming Docker support is present or absent.
- Do not overwrite an existing, working Docker setup to force it onto the reference stack just for consistency — audit and improve it in place instead (see Scope).

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

- **PHP application server**: default to **PHP-FPM** (Debian bookworm image, multi-stage `base`/`dev`) in its own container, paired with a separate **Nginx** container — see [Reference Stack](#reference-stack-default-structure). Version aligned to `composer.json`, configurable Xdebug, and OPcache.
- If the project already runs **FrankenPHP**, or a **Caddy**/**Apache** split, keep that setup instead of migrating it to PHP-FPM+Nginx.
- **MariaDB** by default (matches the reference stack); use **PostgreSQL** instead if `DATABASE_URL` indicates it. Persistent volume either way.
- **Redis**, **RabbitMQ**, **Mailpit** according to needs
- **Node** if package.json is present
- **Messenger Worker** optional, only if the project uses async transports
- **Search**: Elasticsearch/OpenSearch/Meilisearch only if necessary
- **Mercure** only if package/config is explicit

## Reference Stack (Default Structure)

For a new environment, copy `templates/docker/` from this skills repo into the project and adapt it, rather than inventing a new layout each time — this is what keeps every dockerized project consistent:

```text
docker/
├── php/
│   ├── Dockerfile              # multi-stage: composer -> base (php-fpm) -> dev (+ Xdebug)
│   ├── docker-entrypoint.sh    # composer install if vendor/ is missing, then exec "$@"
│   └── conf.d/
│       ├── symfony.ini         # memory_limit, opcache, date.timezone
│       └── xdebug.ini          # off by default, toggled by XDEBUG_MODE
└── nginx/
    └── default.conf            # front controller, fastcgi_pass php:9000
compose.yaml                    # x-app-env anchor, named volumes, healthchecks
Makefile                        # self-documenting `make help`, one target per common command
.dockerignore
```

`docker/sequelace/` is generated on demand by `make sequelace` (see [DB GUI Access](#db-gui-access-sequel-ace)) — not part of the copied template, must be git-ignored, never created upfront.

Key conventions to preserve when adapting the templates:

- **Multi-stage Dockerfile**: `composer` stage copies the Composer binary only; `base` installs system packages/PHP extensions and creates the `www-data` user with `UID`/`GID` build args (host permission parity); `dev` extends `base` with Xdebug. Only add a `prod` stage if explicitly requested (see Environments → Production).
- **Non-root by default**: the PHP container runs as `www-data`, never `root`, except transiently in the `dev` stage to install Xdebug.
- **`compose.yaml` `x-app-env` anchor**: centralize shared environment variables (DB, mailer, Messenger DSN) once and reuse them with `<<: *app-env` across `php` and `messenger-worker` instead of duplicating them.
- **Named volumes** for `vendor/` and DB data, so `vendor/` isn't shipped through the bind-mounted source and doesn't get wiped by `docker compose down` without `-v`.
- **Healthchecks** on `database` and `redis`, referenced via `depends_on: condition: service_healthy` so the PHP container never starts against a DB that isn't ready yet.
- **Self-documenting Makefile**: `make help` lists every target from its `## comment` via `grep`/`awk` (see Makefile and Commands) — extend it with the same pattern rather than a different one.

Adapt per project: `PHP_VERSION` build arg and installed extensions (from `composer.json`), `database` image (MariaDB vs PostgreSQL), whether `redis`/`messenger-worker`/`mailpit` are needed at all (see Need Detection), and the exposed ports if they collide with another local project.

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
- `.gitignore`: exclude generated local-only files (e.g. `/docker/sequelace/` behind its own marker block, see [DB GUI Access](#db-gui-access-sequel-ace))
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

## DB GUI Access (Sequel Ace)

In ~95% of team projects the DB engine is MariaDB and local dev happens on macOS with **Sequel Ace** as the DB browser. Because of that, treat `make sequelace` as part of the default reference stack — propose it by default whenever the project uses MariaDB/MySQL, without waiting to be asked — not as a rare opt-in like the extras below:

- Add the target from `templates/docker/Makefile`: it writes a local `.spf` connection file (host/port matching the `database` service's exposed port, user/password/database matching its dev defaults) to `docker/sequelace/<project>-dev.spf`, then opens it with `open`.
- Skip it only when the project doesn't use MariaDB/MySQL (e.g. PostgreSQL — no direct Sequel Ace equivalent, mention pgAdmin/TablePlus instead if asked), the target OS isn't macOS, or the user says they don't use it.
- **Git-ignore the generated file**: add a marked block to `.gitignore`, mirroring the Flex recipe marker convention so it can be detected and isn't duplicated:

  ```gitignore
  ###> sequelace ###
  /docker/sequelace/
  ###< sequelace ###
  ```

- Never commit the generated `.spf` file itself, and never put anything beyond the local dev-default credentials already in `compose.yaml` into it — it is a local convenience file, not a secrets store.

## Optional Extras

Unlike Sequel Ace above, these are rarer, infrastructure-specific add-ons seen on some team projects, not part of the default reference stack. Only include one when the project already uses it, or the user explicitly asks for it — never add it "for consistency":

- **Centralized log shipping (Filebeat/ELK)**: a `filebeat` service reading Docker container logs and forwarding them to a shared Logstash/ELK stack, gated by a `<project>_ship_logs=true` label on the services to collect and an `external: true` network shared with that stack. Only relevant if the team's shared ELK stack is actually reachable from the project's environment — see the commented block in `templates/docker/compose.yaml`.

Document in the Output Format which extras were added and why, same as any other service.

## Makefile and Commands

Propose only adapted commands, following the self-documenting pattern from `templates/docker/Makefile` (`make help` lists every target via its `## comment`, `.DEFAULT_GOAL := help`):

Core targets (always relevant):

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
- `make sequelace` — default when the DB is MariaDB/MySQL (see [DB GUI Access](#db-gui-access-sequel-ace)); drop it otherwise.

Conditional targets (only if the matching service/need exists):

- `make messenger-consume` / `make worker-logs` — only with an async transport and `messenger-worker` service.
- `make mail` — only with a `mailpit` service.
- Extra targets for [Optional Extras](#optional-extras) (e.g. `make elk-logs`) — only when that extra was actually added.

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

## Documenting the Decision (ADR)

When a Docker environment is generated **from scratch** (not a small fix to an existing setup), propose — do not silently create — an ADR in the target project capturing the stack choice, using `templates/adr/0000-template.md` as the base:

- Suggested path: `docs/adr/<NNNN>-docker-stack.md`, following the project's existing ADR numbering if `docs/adr/` already exists; otherwise start at `0001`.
- Fill in Context (why a Docker environment is needed now), Decision (the concrete stack: PHP-FPM+Nginx reference stack, or FrankenPHP/other if that's what was actually chosen and why), Alternatives Considered, and Consequences.
- Only write the file after the user agrees; if they decline or don't respond to the offer, proceed without it — it does not block generating the Docker environment itself.
- Skip this offer entirely for audits, targeted fixes, or when adding a single service to an already-decided stack — an ADR records a stack-level decision, not every incremental change.

## Do Not

- Do not read `.env.local`.
- Do not overwrite an existing Dockerfile or compose without preserving useful conventions.
- Do not unnecessarily expose DB, RabbitMQ, Redis, or internal services.
- Do not introduce an unnecessary service.
- Do not impose the PHP-FPM+Nginx reference stack, FrankenPHP, Caddy, Apache, Node, or a PHP version on a project that already made a different working choice.
- Do not use production credentials.
- Do not automatically run migrations, fixtures, or workers at startup without an explicit request.
- Do not generate a production configuration while pretending it is ready for every hosting provider.
- Do not strip the `###> recipes ###` / `###< recipes ###` markers from `Dockerfile` or `compose.yaml`; Symfony Flex needs them to keep injecting service config on package install.
- Do not add the [Optional Extras](#optional-extras) (centralized log shipping) by default; they are per-project opt-ins, unlike Sequel Ace which is the default for MariaDB/MySQL projects.
- Do not commit the generated Sequel Ace `.spf` file or omit its `.gitignore` marker block.
- Do not create an ADR file without the user's explicit go-ahead.

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
- For a from-scratch environment: whether an ADR was offered/created for the stack decision

Generate all required files (`compose.yaml` or `docker-compose.yml`, Dockerfile, web configs, `.dockerignore`, Makefile if useful) from `templates/docker/`, adapted to the project, ready to use with `docker compose up` in the requested context.
