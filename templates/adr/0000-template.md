# ADR-<NNNN>: <Short title of the decision>

Date: <YYYY-MM-DD>

## Status

<Proposed | Accepted | Superseded by ADR-XXXX | Deprecated>

## Context

<What problem or constraint forced a decision? What is the situation before this
choice: existing stack, team habits, technical or organizational constraints.
Keep this factual — no rationale for the decision itself here.>

## Decision

<The decision itself, stated as a clear, actionable sentence. E.g. "We will use
a PHP-FPM container with a separate Nginx container for the local Docker
environment, instead of a single FrankenPHP container."

Describe the chosen structure concretely: images, services, key configuration
choices. Link to the reference template used, if any (e.g.
`templates/docker/` in claude-skills-php).>

## Alternatives Considered

- <Alternative 1> — <why it was not chosen>
- <Alternative 2> — <why it was not chosen>

## Consequences

- <Positive consequence: what becomes easier, more consistent, or safer>
- <Negative consequence or trade-off accepted: what becomes harder or is given up>
- <Follow-up work created by this decision, if any>

## Example: Standardizing the local Docker stack

> Status: Accepted
>
> Context: each project was free to choose its own Docker setup (FrankenPHP,
> PHP-FPM+Apache, PHP-FPM+Nginx), making it hard to onboard developers across
> projects and to reuse Makefile/CI conventions.
>
> Decision: new and re-dockerized projects use the reference stack described
> in `templates/docker/` of claude-skills-php: PHP-FPM (Debian bookworm) in its
> own container, a separate Nginx container, MariaDB/PostgreSQL depending on
> the project, Redis and Mailpit as needed, and a self-documenting Makefile
> (`make help`).
>
> Alternatives considered: FrankenPHP single-container (official Symfony
> Docker recommendation) — rejected as the default because it diverges from
> the stack already running in production for existing projects, making
> cross-project conventions (Makefile targets, healthchecks, log labels)
> harder to keep consistent.
>
> Consequences: every project dockerized this way shares the same
> `docker/php`, `docker/nginx`, `compose.yaml`, and `Makefile` shape, so a
> developer moving between projects finds the same commands and structure.
> Trade-off: slightly more container overhead than a single FrankenPHP
> container, considered acceptable for local dev.
