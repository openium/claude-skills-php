---
name: refactor
description: "Safe refactoring for PHP/Symfony projects. Use to simplify a class or method, extract business logic, reduce duplication, move code between Domain/Application/Infrastructure, or improve design without changing observable behavior."
---

# PHP/Symfony Refactoring

## Scope

If the user specifies a file, class, directory, or diff, limit the refactor to that scope.

If the scope is ambiguous or too broad, ask for a target before modifying code.

## Process

1. Read the target code and its existing tests.
2. Search for incoming calls and public usages of the class or method.
3. Identify the project's real architecture: classic MVC, hexagonal, partial DDD, or another model.
4. Choose the smallest refactor that improves the code without changing behavior.
5. Apply changes in coherent steps, keeping the code compilable.
6. Run targeted tests if possible, or state precisely which tests to run.

## Principles

- Preserve observable behavior: same inputs, outputs, exceptions, side effects, and public contracts.
- Respect the project's existing conventions before applying a general rule.
- Keep controllers, commands, handlers, and listeners thin: parsing, delegation, output.
- Move business logic to an application service, use case, or the Domain according to the existing architecture.
- Prefer names that describe business intent rather than technical mechanics.
- Reduce coupling without needlessly scattering the code.

## Architecture

- Do not impose a full hexagonal architecture on a simple MVC project.
- If the project is hexagonal, the Domain does not depend on Symfony, Doctrine, Infrastructure, or the Framework.
- If use cases exist, place them in Application/ and let them handle orchestration.
- Repositories execute queries; business decisions remain in Domain/Application.
- Infrastructure adapters implement the ports or interfaces expected by Domain/Application.
- Do not move a class only to follow a theory if the project does not have that organization.

### SOLID Principles

- **S**: a class has one identifiable responsibility.
- **O**: prefer extension only when several variants exist or are explicitly planned.
- **L**: a subtype must remain substitutable without surprises.
- **I**: prefer small, focused interfaces.
- **D**: depend on abstractions when it genuinely protects the Domain or an extension point.

## Common Refactors

- Method > 20 lines: extract into private methods named by intent
- Class > 250 lines: identify responsibilities before extracting
- Controller with business logic: extract to an application service or use case
- Heavy console command: extract logic to a service, keep the command for I/O
- Heavy listener, subscriber, or handler: extract to a dedicated service
- Repository with business logic: extract to a Domain service
- Nested conditions: prefer early returns, named private methods, or a strategy if several real variants exist
- Duplicated validation: extract to DTO, constraint, validator, or service according to project conventions
- Complex object construction: extract a factory only if construction is reused or carries a business rule

## Abstractions

- Do not create an abstraction for a single use case.
- An interface is acceptable for an outgoing Domain dependency or a real extension point.
- A strategy is acceptable if at least two behaviors exist or are explicitly requested.
- A factory is acceptable if construction is complex, reused, or business-related.
- A dedicated service is acceptable if the extracted behavior has a clear name and reduces one responsibility.
- Avoid "just in case" abstractions.

## Tests and Validation

- Run targeted tests when they exist.
- Run PHPStan if the project uses it and the refactor touches types, signatures, or generics.
- Add or adapt a test only if needed to secure the refactor.
- If no test covers the refactored behavior, report the risk in the response.

## Do Not

- Do not extract an interface if only one class implements it (except for the Domain).
- Do not rename without a reason.
- Do not change a public signature unless necessary.
- Do not modify API payloads, HTTP statuses, business exceptions, or validation rules.
- Do not modify an existing migration that may already have been applied.
- Do not change route, command, service, or handler names without an explicit request.
- Do not mix refactoring and functional change in the same modification.

## Output Format

Summarize:

- Modified files
- Refactors applied and short reason
- Preserved behavior
- Tests or checks run
- Remaining risks or limitations
