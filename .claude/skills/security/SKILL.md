---
name: security
description: "Fast security audit for PHP/Symfony projects. Analyzes modified files or a given scope. Detects SQL injections, XSS, CSRF, data exposure, and missing permissions. Based on the OWASP Top 10. Use before a commit or to audit a scope."
---

# PHP/Symfony Security Audit

## Scope

By default, analyze modified files (`git diff`). If the user specifies a file, directory, or scope, analyze that scope.

## Symfony Context to Check

Before the audit, inspect according to the scope:

- `config/packages/security.yaml`: firewalls, access control, password hashers, remember me
- `config/packages/framework.yaml`: CSRF, session, trusted proxies, trusted hosts
- `config/packages/nelmio_cors.yaml` if present: CORS configuration
- API controllers, voters, authenticators, application services, and serializers
- Twig templates used for emails
- `composer.json` and `composer.lock`
- `.env` and `.env.dist` if present

Never read or display the contents of `.env.local`.

## Analysis Criteria (OWASP Top 10)

### A01 - Broken Access Control (blocking)

- Sensitive route without visible access control through `#[IsGranted]`, `denyAccessUnlessGranted()`, `access_control`, voter, or application service
- Sensitive route accessible without authentication
- Access check based on the user ID passed as a parameter (IDOR)
- Missing voter for operations on owner-based entities
- `security.yaml`: firewall with `security: false` in production
- Direct access to an entity without checking that the current user is entitled to it

### A02 - Cryptographic Failures (blocking)

- Password stored in clear text or hashed with MD5/SHA1
- Token/secret hardcoded in source code
- `.env` file with committed secrets
- Weak or predictable encryption key
- HTTP communication instead of HTTPS for sensitive data

### A03 - Injection (blocking)

- DQL/SQL query built by concatenating variables
- `$connection->executeQuery("SELECT ... WHERE id = $id")` without binding
- Shell command built with user data without `escapeshellarg()`
- Regular expression built with user data
- LDAP query without escaping

### A04 - Insecure Design (important)

- Missing rate limiting on login, password reset, public API
- No server-side validation (client-side only)
- Authorization logic in the Twig template instead of the controller/voter
- Endpoint returning more data than necessary (overexposure)
- Missing input DTO on a sensitive endpoint that accepts user data
- Serializer configured with groups that are too broad or missing
- Deserialization that directly hydrates a Doctrine entity exposed through the API
- API response exposing sensitive fields through the serializer

### A05 - Security Misconfiguration (important)

- `APP_DEBUG=true` in production
- `kernel.debug` enabled in prod
- Symfony profiler accessible in production
- Overly permissive CORS (`allow_origin: '*'`)
- Missing security headers (CSP, X-Frame-Options, HSTS)

### A07 - XSS (blocking)

- `|raw` on user data in Twig
- `{{ variable|raw }}` without prior sanitization
- Injection into HTML attributes (`href="{{ user_input }}"`)
- User data inside inline JavaScript
- Email templates that inject user data without escaping or sanitization

### A08 - Insecure Deserialization (blocking)

- `unserialize()` on user data
- JSON deserialization without schema validation before processing
- `Serializer::deserialize()` without denormalization groups

### File Uploads (blocking)

- Upload without validating the real MIME type
- File extension used as the only proof of type
- User-provided filename kept as-is
- File uploaded into an executable public directory
- Missing server-side size limit

### Logs and Errors (important)

- Logs containing tokens, passwords, Authorization header, or sensitive personal data
- API error message exposing a stacktrace, SQL query, or internal information
- Business exception returned as-is to the client
- Sensitive data displayed in Messenger logs or HTTP client logs

### Dependencies (important)

- Vulnerabilities detected by `composer audit`
- Abandoned or unmaintained package used on a critical path
- Composer constraints too broad on a sensitive dependency
- `composer.lock` missing from the repository

### Sensitive Files

- `.env` or `.env.dist` with secrets in git
- `composer.lock` missing from the repository (unlocked versions)
- Configuration files with credentials

## Output Format

For each vulnerability:

- File and line
- OWASP category
- Severity (blocking / important / suggestion)
- Evidence verified in the code or configuration
- Realistic exploitation scenario
- Vulnerable code
- Fixed code
- Recommended regression test or verification
- Reference (Symfony Security documentation link if applicable)
