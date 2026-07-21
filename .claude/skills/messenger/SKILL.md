---
name: messenger
description: "Audits and fixes Symfony Messenger. Analyzes transports, routing, buses, handlers, retry/failure, serializers, workers, idempotency, transactions, and performance, considering Symfony/PHP versions and project conventions."
---

# Symfony Messenger Check

## Scope

Determine whether the user is asking for:

- A Messenger configuration audit.
- Debugging a message that is not consumed, failed, or processed several times.
- Adding a message, handler, transport, or routing.
- Fixing retry, failure transport, serializer, or worker.
- A worker performance or reliability audit.
- A targeted scope: message, handler, transport, bus, environment, or file.

If no scope is given, analyze the project's Messenger configuration and messages/handlers.

Do not run a long-running worker or massive retry without explicit confirmation.

## Current State

Inspect according to the project:

- `composer.json`: Symfony version, Messenger, installed transports, Serializer, Doctrine
- Messenger configuration: `config/packages/messenger.yaml`, `.php`, `.xml`, and `when@test`, `when@prod` variants
- Transports, failure transports, routing, buses, middleware, serializers
- Messages and handlers in `src/`, service tags, attributes, or interfaces
- `config/services.yaml`, `config/services_test.yaml`, autowire/autoconfigure
- Messenger tests, in-memory/test transports, fixtures
- Symfony logs and failed messages if the issue is runtime
- Worker/supervision configuration if present: Docker, systemd, Supervisor, Kubernetes, CI

Never read `.env.local`. Check environment variable names without displaying sensitive values.

## Process

1. Read Messenger configuration (`config/packages/messenger.yaml` or `.php`).
2. Scan all messages and handlers.
3. Cross-check information and detect inconsistencies.
4. Identify Symfony/PHP version and handler declaration conventions.
5. Check routing, transports, retry, failure transport, serializer, and bus.
6. Propose targeted fixes and validations.

## Symfony Compatibility

- Old Symfony / PHP 7.x: handlers are often declared through service tags or autoconfigure, not PHP attributes.
- Recent Symfony / PHP 8+: `#[AsMessageHandler]` is possible if already a project convention.
- Do not convert tags to attributes if the project must remain PHP 7.x compatible.
- Respect existing configuration: YAML, PHP, XML, environment-specific config.
- Check command and option differences according to the installed Symfony version.

## Severities

- **Blocking**: message without handler, routed transport not defined, unintended sync fallback for long processing, infinite retry, serializer incompatible with shared transport, non-idempotent handler for a critical replayable message.
- **Important**: missing failure transport, fragile retry policy, wrong bus, handler too heavy, external call without timeout, message contains a Doctrine entity, poorly controlled transaction.
- **Suggestion**: naming, logs, handler split, routing simplification, observability, or additional tests.

## Analysis Criteria

### Routing (blocking)

- Message without registered handler
- Handler registered for a message that does not exist
- Message routed to an undefined transport
- Synchronous message that should be asynchronous (long processing, external call)
- Asynchronous message without explicit transport (silent fallback to sync)
- Message routed to the wrong bus
- Routing too broad and capturing unintended messages
- Command and event processed the same way while their guarantees differ
- Critical message not routed in `prod` but routed in `dev` or `test`
- Test transport different from prod behavior without justification

### Transports (important)

- Doctrine transport used for high volume without checking polling, indexes, and cleanup
- AMQP/RabbitMQ without coherent exchange/queue/routing key
- Redis/SQS without adapted visibility, delay, or retry options
- Sync transport used unintentionally
- In-memory transport used only in tests, never assumed in prod
- Missing DSN or environment variable not documented by name
- Queue names inconsistent with environment or domain
- Missing separation between critical messages and slow processing

### Retry and Failure (important)

- Async transport without defined retry policy
- Infinite retry (no `max_retries`)
- Missing failure transport for async transports
- Same retry for permanent and transient errors
- Too aggressive or missing backoff on external dependency
- No jitter while several messages may retry at the same time
- Failure transport not monitored or documented
- Retry/remove commands used without prior inspection of the exception

### Message Design (important)

- Message containing a Doctrine entity instead of a stable identifier
- Non-serializable message: closure, resource, service, Doctrine proxy, complex mutable object
- Message containing too much data or unnecessary sensitive data
- Mutable message while it should be immutable
- Message incompatible between two successive deployments
- Message name too technical or not aligned with business intent

### Idempotency and Transactions (blocking)

- Handler not idempotent while the message may be replayed
- Email, payment, external call, or critical write repeated without guard
- Message dispatched before DB commit with risk of reading non-persisted state
- Doctrine transaction too broad around external HTTP/I/O calls
- Missing lock or unique constraint to avoid double processing
- No outbox while the project uses one to guarantee dispatch after commit

### Handlers (important)

- Handler containing heavy business logic (> 50 lines)
- Handler making synchronous HTTP calls without timeout
- Handler flushing Doctrine inside a loop
- Handler without error handling for recoverable failures
- Handler fetching services from the container instead of explicit injection
- Handler mixing orchestration, business logic, and external I/O
- Handler without useful logs for a critical message
- Handler swallowing an exception and preventing retry/failure
- Handler not distinguishing transient exception from permanent business error
- Batch handler without periodic `flush()`/`clear()` or memory limit

### Serialization (important)

- External transport (RabbitMQ, SQS) without explicit serializer
- Native PHP serializer on a transport shared between applications
- Message class or namespace change without compatibility strategy
- Readonly properties or strict types incompatible with the serializer used
- Sensitive data serialized while an identifier would be enough
- Unversioned format for messages exchanged between applications

### Workers and Supervision (important)

- Worker without restart strategy on deployment
- No `messenger:stop-workers` or equivalent after release
- No memory, time, or message count limit if the project may leak
- Prefetch/concurrency not adapted to message cost
- No visible supervision or healthcheck for critical workers
- Insufficient logs to correlate message, handler, and error

### Performance (important)

- Message too large
- Handler loading too many entities into memory
- Doctrine N+1 in a handler
- Sequential HTTP calls in a handler
- Long processing that should be split into several messages
- Doctrine transport not cleaned up or `messenger_messages` table too large

## Tests and Validation

- Unit test for handler with mocked dependencies if orchestration logic exists.
- Functional dispatch and routing test if the project allows it.
- Idempotency test for replayable messages.
- Transient vs permanent error test if retry/failure is critical.
- In-memory or test transport to verify that a message is dispatched.
- Do not use real external transports in tests except for dedicated integration tests.

## Useful Commands

Adapt to the project and Symfony version:

- `bin/console debug:messenger`
- `bin/console debug:container --tag=messenger.message_handler`
- `bin/console messenger:failed:show`
- `bin/console messenger:failed:show --stats`
- `bin/console messenger:failed:retry`
- `bin/console messenger:failed:remove`
- `bin/console messenger:consume async -vv`
- `bin/console messenger:stop-workers`
- `bin/console lint:container`
- Targeted handler or dispatch tests

Do not run `messenger:consume` without a limit, massive retry, or failed-message removal without explicit confirmation.

## Do Not

- Do not put a Doctrine entity in a message.
- Do not assume an async message will be processed exactly once.
- Do not retry a non-idempotent operation without a guard.
- Do not use native PHP serializer for a transport shared between applications.
- Do not swallow exceptions in a critical handler.
- Do not route long processing to sync by default.
- Do not add a transport without adapted failure transport and retry policy.
- Do not display DSNs, credentials, tokens, or sensitive payloads in the response.

## Output Format

Present:

1. **Summary**: number of messages, handlers, transports, buses, and failure transports.
2. **Routing Matrix**: message, handler, bus, transport, retry, failure transport.
3. **Findings** sorted by severity.

For each finding:

- File and line
- Severity: blocking, important, or suggestion
- Evidence: config, handler, message, command, or log
- Concrete impact: lost message, sync processing, duplicate, infinite retry, operational debt
- Proposed fix with config or code example if useful
- Recommended validation: command, test, or scenario

If the analysis is limited by missing config, logs, or Symfony version, report it explicitly.
