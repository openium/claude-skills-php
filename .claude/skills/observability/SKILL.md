---
name: observability
description: "Audits and improves observability in a PHP/Symfony project. Analyzes Monolog logs, errors, metrics, traces, correlation IDs, Sentry, Messenger, HTTP clients, sensitive data, and alerting to diagnose production without exposing secrets."
---

# PHP/Symfony Observability

## Scope

Determine whether the user is asking for:

- An audit of logs, errors, traces, or metrics.
- Adding logs around a business flow, endpoint, command, or Messenger handler.
- Integrating or fixing Sentry, Monolog, OpenTelemetry, Prometheus, or an equivalent tool.
- Diagnosing missing information in production.
- Reducing noise or sensitive data leakage in logs.

If no scope is given:

- Read `git status` and the diff if the audit concerns recent changes.
- Inspect Monolog/config and modified critical paths.
- Ask for the flow or incident that needs to become observable if necessary.

Do not read or display `.env.local`, DSNs, tokens, or sensitive personal data.

## Current State

Inspect according to the project:

- `composer.json`: MonologBundle, Sentry, OpenTelemetry, Prometheus, Messenger, HttpClient
- `config/packages/monolog.yaml`, `sentry.yaml`, messenger, framework/http_client
- Affected services, controllers, commands, handlers, listeners, and external clients
- Existing logs, Monolog processors, channels, handlers, JSON or text format
- CI/prod config if observability is deployed by environment
- Tests that verify logs or error behavior if present

Never expose a secret or sensitive payload in the response.

## Goals

Good observability must make it possible to answer:

- What failed?
- For which user, tenant, organization, or resource, without exposing sensitive data?
- Which business input or stable identifier can retrieve the context?
- Which external dependency or component is involved?
- Is it transient, permanent, business-related, or technical?
- How can logs, traces, metrics, and Messenger messages be correlated?

## Severities

- **Blocking**: secret/PII leak, silent critical error, swallowed exception, critical payment/email/write flow not traceable, logs containing tokens.
- **Important**: missing correlation ID, insufficient logs on a critical flow, massive noise, missing metric or alert, failed messages not observable.
- **Suggestion**: context enrichment, naming, dedicated channel, JSON structure, useful dashboard or alert.

## Logs

- Use structured logs with context instead of concatenating strings.
- Include stable identifiers: request id, user id, tenant id, entity id, message id, external id.
- Do not log passwords, tokens, cookies, Authorization header, DSN, complete sensitive payload, or unnecessary personal data.
- Choose the correct level: debug, info, notice, warning, error, critical.
- Avoid logs inside loops or hot paths without a guard.
- Use dedicated Monolog channels for critical flows if the project already does so.
- Do not swallow an exception after logging if retry/failure must be triggered.

## Errors and Exceptions

- Distinguish expected business errors from technical incidents.
- Do not hide an exception behind a silent return.
- Add business context to exceptions or logs without leaking sensitive data.
- Check that critical errors reach the alerting tool.
- For APIs, do not expose stacktraces or internal details to the client.
- For commands, return a coherent exit code.

## Correlation IDs and Traces

- Check propagation of a request id / correlation id.
- Propagate context to outgoing HTTP calls if this is a project convention.
- Propagate a correlation identifier in Messenger messages if useful.
- Avoid generating a new id at every layer if one already exists.
- For OpenTelemetry/APM, instrument boundaries: HTTP, DB, queue, external clients.

## Metrics and Alerting

- Identify useful metrics: error rate, latency, job duration, queue backlog, failed messages, retries, external timeouts.
- Add business metrics only if they support an alert or dashboard.
- Plan labels with controlled cardinality: no email, massive user UUID, or dynamic payload as labels.
- Define thresholds and symptoms, not only raw counters.
- Avoid noisy alerts that trigger no action.

## Messenger and Async

- Log start/end/failure of critical handlers with message id and business identifiers.
- Observe retry count, failed transport, processing duration, and memory if relevant.
- Do not log the complete message if the payload is sensitive.
- Differentiate transient and permanent errors.
- Preserve the exception so Messenger can retry/fail if necessary.
- Check that `messenger:failed:*` or an equivalent dashboard is usable.

## HTTP Clients and External Dependencies

- Always have an explicit timeout.
- Log target service, operation, status, duration, retry count, and correlation id.
- Do not log sensitive headers or complete payloads by default.
- Classify network errors, business 4xx, provider 5xx, and timeouts.
- Add metrics or traces on critical dependencies.

## Security and Compliance

- Redact secrets, tokens, cookies, Authorization, reset-password tokens, magic links.
- Limit personal data: full email, phone, address, full name only if necessary and authorized.
- Check logs for emails, webhooks, payments, uploaded files.
- Avoid sending sensitive data to third-party observability services without filtering.
- Do not read `.env.local` to check a DSN.

## Tests and Validation

Plan according to the change:

- Test that the exception is not swallowed.
- Test that a critical event logs the expected context.
- Test that sensitive data is not in the logged context if easily testable.
- Functional test for API errors without stacktrace leakage.
- Messenger handler test on transient/permanent error.

Useful commands:

- `bin/console debug:config monolog`
- `bin/console debug:container monolog`
- `bin/console debug:event-dispatcher`
- `bin/console messenger:failed:show`
- Targeted tests

Do not trigger real external calls or production alerts without explicit confirmation.

## Do Not

- Do not log secrets or complete payloads to "debug quickly".
- Do not replace an exception with a simple log.
- Do not add verbose logs on hot paths without an appropriate level or sampling.
- Do not create metrics with uncontrolled cardinality.
- Do not send personal data to a third-party tool without filtering.
- Do not modify `.env.local`.
- Do not add an observability dependency without an explicit request.

## Output Format

Present findings sorted by decreasing severity.

For each finding:

- File and line
- Severity: blocking, important, or suggestion
- Category: logs, errors, traces, metrics, alerting, Messenger, HTTP client, security
- Verified evidence
- Operational impact
- Proposed fix
- Recommended test or verification

After modification, summarize:

- Modified files
- Context added or removed
- Sensitive data protected
- Commands/tests run
- Remaining limitations
