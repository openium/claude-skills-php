---
name: performance
description: "Performance audit and optimization for PHP/Symfony projects. Analyzes endpoints, commands, workers, Doctrine, Twig, Serializer, cache, HTTP, and batch jobs. Measures before optimizing, identifies the root cause, and proposes verifiable fixes."
---

# PHP/Symfony Performance Audit

## Scope

If the user specifies an endpoint, command, Messenger worker, Doctrine query, Twig page, API, diff, or file, analyze only that scope.

Otherwise:

- Read `git status` and the diff if the audit concerns recent changes.
- Identify sensitive application paths: controllers, repositories, handlers, commands, templates, normalizers.
- Ask for the exact symptom if no target is given: slowness, timeout, memory, N+1, DB load, network latency.

Do not modify code during an audit unless explicitly requested.

## Current State

Inspect according to the project:

- `composer.json`: Symfony, Doctrine, Messenger, HttpClient, Cache, Serializer, DebugBundle, Blackfire
- Doctrine configuration: DBAL, ORM, mappings, second-level cache, SQL logging
- Cache configuration: pools, adapters, tags, TTL, HTTP cache
- Messenger configuration: transports, retry, workers, possible batch size
- Controllers, repositories, services, handlers, templates, or normalizers in scope
- Tests, fixtures, and commands available to reproduce the scenario
- Existing Symfony logs, profiler, or traces if available

Never read `.env.local`. Do not display secrets or personal data present in logs.

## Measure Before Optimizing

Before proposing a change, look for evidence:

- Response time, command duration, or message duration
- Number of SQL queries
- Slow SQL query and important parameters
- Number of rows read, returned, or hydrated
- Memory consumed and peak memory
- Number of HTTP or I/O calls
- API payload size or generated HTML size
- Input data volume

If direct measurement is not possible, state the hypothesis and propose the command or instrumentation to use.

## Severities

- **Blocking**: timeout, OOM, massive N+1, critical flush in a loop, full table load on a frequent path, probable DB lock, external call without timeout on a critical web path.
- **Important**: plausible slow query, missing index, missing pagination, missing cache on costly computation, excessive payload, uncontrolled batch.
- **Suggestion**: minor optimization, simplification, micro-optimization without major measured impact.

## Analysis Criteria

### Doctrine - Queries (blocking)

- **N+1**: loop over a collection with access to a lazy-loaded relation
- **Flush in a loop**: `$em->flush()` called on each iteration
- **SELECT ***: query loading all columns when only a few are needed
- **Query without index**: WHERE or ORDER BY on a non-indexed column
- Missing pagination on a potentially large list
- `findAll()` or `toArray()` on an unbounded table or collection
- Query inside a loop or inside a normalizer called on a collection
- Uncontrolled joins that multiply rows and explode hydration
- Costly `COUNT` needlessly recalculated on each query
- Non-sargable filter or sort: function on column, leading wildcard, cast, non-indexable expression

### Doctrine - Hydration (important)

- Full entity hydration when only a few fields are read
- Loading whole collections into memory
- Entities unnecessarily tracked by the UnitOfWork
- Serializer triggering lazy loads
- Fetch join of several collections that produces a Cartesian product
- Missing `clear()` in batch processing
- Managed entities used for pure reads or massive exports
- `PersistentCollection` filtered in PHP instead of SQL

### Database (important)

- Missing index on `WHERE`, `JOIN`, `ORDER BY`, or unique constraint columns
- Index unusable because of a function, conversion, or leading wildcard
- Deep offset pagination without an adapted strategy
- `LIKE '%term%'` on large volume without a specialized index
- JSON column filtered without an adapted index
- Long lock or overly broad transaction
- Massive delete or update query without batch or limit
- Risky index migration without online/concurrent strategy according to the DBMS

For PostgreSQL, propose `EXPLAIN (ANALYZE, BUFFERS)` if DB access is available.
For MySQL/MariaDB, propose `EXPLAIN` and check indexes, cardinality, filesort, and temporary tables.

### Cache (important)

- Costly query result not cached
- Configuration fetched from the DB on every request
- Static data recalculated on every call
- Cache without invalidation strategy
- Missing or too-long TTL for changing data
- Cache stampede risk on costly computation
- Per-user cache that explodes cardinality
- Missing HTTP cache for stable public content
- Missing warmup for costly data used at startup

### HTTP and Network (important)

- Synchronous HTTP calls inside a web request
- Missing timeout on external calls
- Sequential HTTP calls that could be parallelized
- External calls inside a loop
- Retry without limit or backoff
- No circuit breaker while the project already uses one elsewhere
- External payload too large or not paginated
- Disk I/O or remote storage on a critical web path

### Twig Templates (suggestion)

- Doctrine queries in templates (lazy loading)
- Loops with costly filtering on large collections
- Costly method calls inside a Twig loop
- Component or partial rendering where each render triggers queries
- Missing pagination or empty state that is costly to compute
- Use of `|length` on a non-initialized lazy collection

### Serializer and API (important)

- Serializer groups too broad and loading too many relations
- Normalizer that calls a repository or external service per item
- Circular references worked around by exposing too much data
- Unpaginated or overly large payload
- Costly computed fields on each element of a collection
- Missing output DTO while the public contract could limit loaded data

### Messenger and Batch (important)

- Message or command loading too many entities into memory
- Missing batch size, periodic `flush()`/`clear()`, or resumability
- Worker leaking memory on long processing
- Retry on a non-idempotent or very costly operation
- Long processing executed in a web request instead of asynchronously
- Ack/retry/prefetch not adapted to message cost

### PHP Runtime and Algorithms (important)

- Quadratic algorithm on large collections
- `array_map`, `array_filter`, `usort`, or `in_array` repeated on large volumes without prior indexing
- Costly regex or regex executed in a loop
- Whole large files read into memory instead of streaming
- Export generation without pagination or streaming
- Autoload or reflection repeated in a loop
- High-volume logs on a hot path

## Expected Fixes

- Replace N+1 with an adapted query, controlled fetch join, targeted eager loading, or DTO projection.
- Add pagination, limit, or streaming for lists and exports.
- Use scalar results, partial objects, or DTOs when the full entity is unnecessary.
- Add indexes or adapt the query to use existing indexes.
- Move long processing to Messenger if the web response must not wait.
- Add cache with key, TTL, and explicit invalidation.
- Add timeouts, bounded retry, or parallelization for HTTP calls.
- For Doctrine batches, use chunks and periodic `flush()` and `clear()`.

## Useful Commands

Adapt to the project:

- Symfony Profiler for query count, SQL time, memory, timeline
- `bin/console about`
- `bin/console debug:container`
- `bin/console doctrine:schema:validate`
- `bin/console doctrine:migrations:status`
- Targeted tests or reproduction scenario
- PHPStan if the optimization touches types or contracts
- Blackfire if present in the project
- `EXPLAIN` / `EXPLAIN ANALYZE` to propose if DB access is available

Do not run heavy load, destructive benchmark, global cache purge, or migration without explicit confirmation.

## Do Not

- Do not optimize without proof or a clearly stated hypothesis.
- Do not add cache without invalidation or TTL.
- Do not hide a slow query behind cache if the query remains critical elsewhere.
- Do not load a whole table or collection into memory.
- Do not replace a simple query with a complex abstraction without clear benefit.
- Do not sacrifice business invariants to optimize.
- Do not make PHP micro-optimizations when the main cost is SQL, network, or I/O.
- Do not delete tests, useful logs, or validations to artificially gain time.

## Output Format

Present findings sorted by decreasing severity.

For each issue:

- File and line
- Severity: blocking, important, or suggestion
- Type: Doctrine, DB, cache, HTTP, Twig, Serializer, Messenger, PHP runtime
- Available evidence or measurement
- Concrete impact: time, memory, SQL queries, volume, timeout risk
- Root cause
- Proposed fix
- Risk or tradeoff of the fix
- Recommended validation: profiler, test, EXPLAIN, targeted benchmark
- Expected gain if estimable

If no measurement was possible, say so clearly and separate hypotheses from proven issues.
