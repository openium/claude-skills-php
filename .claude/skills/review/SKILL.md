---
name: review
description: "PHP/Symfony code review for a git diff, modified files, branch, or given scope. Use for a review, reread, PR audit, or pre-commit check. Prioritizes bugs, security, architecture, regressions, missing tests, Doctrine, and migrations."
---

# PHP/Symfony Code Review

## Scope

If the user specifies a file, directory, commit, branch, or scope, analyze only that scope.

Otherwise:

- Analyze the staged diff (`git diff --cached`) if it exists.
- Otherwise, analyze the unstaged diff (`git diff`).
- If no diff exists, ask for the scope to review.

## Process

1. Read `git status` and a diff summary (`git diff --stat` or equivalent).
2. Read the detailed diff for the affected files.
3. Inspect neighboring files only when needed to verify a risk.
4. Identify project conventions before reporting a violation.
5. Report only issues proven by the code, configuration, or diff.

Do not modify code during a review unless explicitly requested.

## Severities

- **Blocking**: confirmed bug, security flaw, data loss, regression, uncontrolled backward compatibility break, certain architecture violation.
- **Important**: real and plausible risk, missing test for critical behavior, technical debt that blocks a near-term change.
- **Suggestion**: readability, convention, simplification, or improvement without immediate risk.

## Analysis Points

### Bugs and Regressions

- Behavior change that is uncovered or unannounced
- Broken edge case: null, empty string, empty collection, extreme value
- Exception swallowed or replaced by a silent error
- Change to a signature, return type, API payload, or public exception
- Dangerous default value or one incompatible with existing behavior

### Architecture

- Business logic in a controller, console command, or listener
- Dependency from the Domain to the Infrastructure or to the Framework
- Symfony import in a Domain class (Entity, ValueObject, business Service)
- Repository that contains business logic instead of queries
- Application service that directly accesses `$_GET`, `$_POST`, `$_SESSION`, or `Request`
- Abstraction added without a real need or incompatible with project conventions

### Security

- DQL/SQL queries built by concatenation instead of bound parameters
- User data displayed in Twig without escaping (`|raw` on untrusted data)
- Sensitive route without access control (`#[IsGranted]`, `denyAccessUnlessGranted`, voter, `access_control`)
- Tokens, passwords, or API keys hardcoded in code
- User data deserialized without validation
- File upload without checking the real MIME type
- Shell command built with unescaped user data
- Log containing a token, password, Authorization header, or sensitive personal data

### Tests

- Business behavior added without a corresponding test
- Possible regression without a regression test
- Functional test that mocks the database instead of testing it for real
- Missing data provider when several similar cases are tested
- Test that does not actually verify the modified behavior

### Doctrine and Migrations

- Entity modified without a corresponding migration
- Destructive migration without a safeguard or safe default value
- Relation without an appropriate cascade
- N+1 query detectable in the diff
- Missing index on an added column used in a WHERE, JOIN, or ORDER BY
- Flush inside a loop instead of a single flush
- Multiple consistent writes without an explicit transaction

### API, DTO, and Validation

- Endpoint that accepts user data without server-side validation
- Direct hydration of a Doctrine entity from an external request
- Serializer with groups that are too broad or missing on a sensitive response
- Missing input DTO for a complex or sensitive operation
- HTTP status, error format, or response contract incompatible with existing behavior

### Messenger and Async

- Handler that is not idempotent while the message may be replayed
- Dangerous retry for a non-repeatable operation
- Message containing too much data instead of stable identifiers
- Missing log or failure handling for critical processing

### Performance

- Query inside a loop, missing pagination, or excessive loading
- Lazy loading triggered inside a loop
- Network call or blocking I/O inside a loop without batching
- Serializer that exposes or loads more data than necessary

### Conventions

- PSR-12 violations (naming, spacing, structure)
- Naming inconsistent with the rest of the project
- Method longer than 30 lines without extraction
- Class longer than 300 lines
- Line longer than 100 characters
- 2 or more consecutive blank lines
- Missing constructor injection (use of `new` for a service)

## Do Not Report

- Missing tests if the diff does not change behavior.
- A convention if the project clearly follows a different convention.
- A stylistic improvement with no impact when more important issues exist.
- A speculative risk without a credible execution path.
- The same issue repeated line by line: group the remark.

## Output Format

Present findings first, sorted by decreasing severity.

For each finding:

- File and line
- Severity: blocking, important, or suggestion
- Evidence verified in the code or diff
- Concrete impact
- Proposed fix
- Recommended test or verification

If no issue is found, say so clearly and mention any tests not run or review limitations.
