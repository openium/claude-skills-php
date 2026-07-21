---
name: debug
description: "Debugging help for PHP/Symfony projects. Analyzes an error, stacktrace, logs, failing test, or unexpected behavior. Reproduces the issue when possible, isolates the root cause, and proposes a verifiable fix without hiding the error."
---

# PHP/Symfony Debugging

## Scope

If the user provides an error, stacktrace, log, failing test, URL, command, or file, start from that signal.

Otherwise:

- Read `git status` to identify recent changes.
- Look for available Symfony logs in `var/log/`.
- Identify useful scripts in `composer.json`.
- Ask for the exact symptom if no usable signal is present.

Never read or display `.env.local`. Do not display any secret, token, cookie, password, or sensitive personal data present in logs.

## Objective

Find the minimal root cause and propose a targeted fix.

Do not settle for:

- hiding the exception with an overly broad `try/catch`;
- ignoring a test error;
- clearing the cache as a permanent solution;
- modifying global configuration without proof;
- changing business behavior to make the symptom disappear.

## Process

1. Capture the exact symptom: message, HTTP code, command, user input, environment, stacktrace, test, or log.
2. Reproduce the issue with the smallest possible command or action.
3. Read the first application frame in the stacktrace, then the necessary neighboring calls.
4. Formulate a verifiable hypothesis and look for proof in code, configuration, database, or logs.
5. Fix the root cause with the smallest change coherent with the project.
6. Rerun the command, test, or scenario that reproduced the bug.
7. Add or propose a regression test when business behavior is touched.

If reproduction is impossible, explain what is missing and provide the commands or data to collect.

## Analysis by Error Type

### Symfony Exceptions

- Walk up the stacktrace to the first application frame.
- Distinguish expected business exception, bad configuration, missing service, serialization error, and runtime bug.
- Check affected configuration files in `config/packages/`, `config/routes/`, `config/services.yaml`, and `config/bundles.php`.
- For a container error, compare expected argument, injected service, autowiring, and aliases.
- For a routing error, check attributes, YAML/XML, prefixes, HTTP methods, host, locale, and route order.

### PHP Errors

- `TypeError`: check the caller, nullable values, DTOs, and conversions from `Request`, env vars, or config.
- `ArgumentCountError`: check signature, dependency injection, factory, callable, and listener.
- `Undefined array key`: validate input, distinguish optional key from incomplete contract.
- `Call to a member function ... on null`: identify why `null` is possible at runtime instead of adding `?->` by default.
- `Cannot access uninitialized property`: initialize in constructor or make the state explicitly nullable if business-related.
- `Memory exhausted`: look for massive loading, missing pagination, Doctrine lazy collection loaded in a loop, export, or overly broad serializer.

### Doctrine Errors

- `MappingException`: check entity annotations/attributes.
- `QueryException`: check DQL syntax and field names.
- `UniqueConstraintViolationException`: identify the constraint and duplicate data.
- `TableNotFoundException`: missing or unexecuted migration.
- `NotNullConstraintViolationException`: check validation, default value, form, DTO, and migration.
- `ForeignKeyConstraintViolationException`: check persistence/deletion order, cascade, orphanRemoval, and existing data.
- `EntityManager is closed`: look for the previous exception that closed the EntityManager.
- `A new entity was found through the relationship`: check cascade persist or explicit persistence of the related entity.
- Broken hydration or proxy: check constructor, readonly properties, PHP types, and nullable columns.
- Schema/entity drift: run or propose `doctrine:schema:validate` and inspect recent migrations.

### HTTP Errors

- 404: check routing (`debug:router`)
- 403: check voters, firewalls, `security.yaml`
- 405: check HTTP methods, forms, API routes, and CORS preflight.
- 400: check validation, deserialization, JSON payload, query parameters, and DTO constraints.
- 401: check authenticator, token, session, cookies, remember me, and firewalls.
- 422: check server-side validation, validation groups, and error mapping.
- 500: read logs (`var/log/dev.log`, `var/log/test.log`, or `var/log/prod.log`) and trace the application cause.
- Unexpected redirect: check access control, login path, trailing slash, locale, and HTTPS.

### Cache Errors

Symptoms: code modification has no effect, service not found after modification, stale template, config not considered.

Check:

- environment used (`APP_ENV`, `--env=test`, `--env=prod`) without reading `.env.local`;
- Symfony cache in `var/cache/`;
- OPcache in prod or in the PHP container;
- Doctrine metadata/query/result cache;
- HTTP cache, reverse proxy, or CDN if the symptom is on the HTTP response side.

Clearing cache is a diagnostic or validation step, not a durable fix if the problem returns.

### Messenger Errors

- Message not consumed: check routing and transport.
- Handler not called: check `#[AsMessageHandler]` and autoconfigure.
- Message in failed transport: inspect the original exception with `messenger:failed:show`.
- Retry loop: check idempotency, retry strategy, transient or permanent exception.
- Duplicate handler: check tags, autoconfiguration, bus, and multiple handlers.
- Impossible deserialization: check message class, readonly properties, post-deployment compatibility, and transport.
- Transaction and async: check dispatch before/after commit, possible outbox, and non-replayable side effects.

### Test Errors

- Read the first real failure before cascading failures.
- Distinguish application bug, missing fixture, uncontrolled clock, test order, external dependency, and overly fragile assertion.
- For PHPUnit, rerun the targeted test with a filter if possible.
- For functional tests, check kernel reboot, test database, transactions, fixtures, and authenticated client.
- Do not modify the assertion to align with broken behavior without proof that the new behavior is expected.

### Configuration and Environment Errors

- Check `composer.json`, `composer.lock`, PHP version, PHP extensions, enabled bundles, and scripts.
- Compare CLI, HTTP, Messenger worker, and Docker container environments if the bug appears only in one context.
- Check required environment variables by name only, without displaying sensitive values.
- Check file permissions for cache, logs, uploads, and sessions.
- For Docker, check service, network, ports, volumes, and injected variables.

### Performance or Timeout Errors

- Identify the saturated resource: CPU, memory, database, network, disk, lock.
- Look for queries in loops, Doctrine N+1, missing pagination, massive serialization, blocking HTTP call.
- Check SQL logs or profiler if available.
- Propose a targeted fix: index, pagination, batch, controlled eager loading, cache, explicit timeout, or async processing.

## Useful Commands

Run only commands adapted to the project:

- `composer test`, `composer phpunit`, or equivalent script if present
- `vendor/bin/phpunit --filter TestName`
- `bin/console about`
- `bin/console debug:router`
- `bin/console debug:router route_name`
- `bin/console debug:container Service\\Id`
- `bin/console debug:event-dispatcher`
- `bin/console debug:autowiring`
- `bin/console lint:container`
- `bin/console lint:yaml config/`
- `bin/console messenger:failed:show`
- `bin/console messenger:failed:retry`
- `bin/console doctrine:schema:validate`
- `bin/console doctrine:migrations:status`
- `composer diagnose`
- `composer show vendor/package`

Do not run destructive commands (`migrate`, massive retry, purge, truncate, DB reset, prod cache pool clear) without explicit confirmation.

## Expected Fixes

- Fix the input contract if invalid data comes from a request, command, message, or config.
- Add an explicit guard when the error case is business-related.
- Fix dependency injection or configuration if the container is the cause.
- Fix the query, mapping, or migration if Doctrine is the cause.
- Fix the route, voter, firewall, or authenticator if HTTP/security is the cause.
- Add a targeted regression test when the fix touches observable behavior.

## Do Not

- Do not hide an exception without treating the cause.
- Do not replace an error with a silent `return null`.
- Do not add `@phpstan-ignore`, `@psalm-suppress`, or a lying assertion to work around the issue.
- Do not delete a failing test without proving it is obsolete.
- Do not clear or modify local data without agreement.
- Do not expose secrets, tokens, cookies, Authorization headers, or personal data in the response.
- Do not assume the error comes from cache without proof.

## Output Format

Respond with:

1. **Diagnostic**: root cause in one sentence, with confidence level if needed.
2. **Evidence**: files, lines, stacktrace, command, or log that justify the diagnostic.
3. **Fix**: applied change or proposed patch, limited to the cause.
4. **Validation**: rerun command, result, or reason it could not be run.
5. **Prevention**: regression test, guard, monitoring, or useful project rule.

If the cause is not yet proven, present hypotheses sorted by probability and the next diagnostic action for each.
