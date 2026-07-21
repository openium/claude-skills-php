---
name: twig
description: "Audits and improves Symfony Twig templates. Checks security, accessibility, performance, forms, architecture, i18n, UX, emails, and project conventions while considering Symfony/Twig versions and existing patterns."
---

# Twig Template Audit

## Scope

If the user specifies a template, directory, diff, form, layout, component, Twig email, back-office page, or HTML rendering, analyze only that scope.

Otherwise:

- Analyze modified templates (`git diff`) if any.
- Otherwise, ask for the template or directory to audit.

Do not modify templates during an audit unless explicitly requested.

## Current State

Inspect according to the scope:

- Target template, parent layouts, partials, includes, macros, or components used
- Controller, form type, DTO, or view model that provides variables
- Routes used by `path()` / `url()`
- Translations and i18n domains if present
- Assets, importmap, Webpack Encore, AssetMapper, or existing pipeline
- Design system, CSS classes, UI components, or project conventions
- Functional tests covering the page if present

Never read `.env.local`.

## Symfony/Twig Compatibility

- Do not assume Twig Components, UX, Stimulus, AssetMapper, or importmap if the project does not use them.
- On legacy projects, respect existing macros, partials, includes, and conventions.
- On recent projects, use Twig Components only if already installed and used.
- Follow the project's routing, forms, translations, and assets style.
- Do not introduce a new template organization without clear benefit.

## Severities

- **Blocking**: XSS, missing CSRF on mutating action, sensitive data leak, dangerous URL, untrusted dynamic include.
- **Important**: broken accessibility, probable N+1, incomplete form, heavy business logic, duplication that makes the page fragile.
- **Suggestion**: convention, readability, partial extraction, i18n, non-critical UX improvement.

## Analysis Criteria

### Security (blocking)

- `|raw` on user data without sanitization
- Variables in HTML attributes without contextual escaping
- URLs built from user data (`href="{{ user_url }}"`)
- Forms without CSRF token (`{{ csrf_token('intent') }}` or `form_rest()`)
- Dynamic template inclusion (`include(variable)`) with untrusted data
- User data injected into inline JavaScript without adapted JSON encoding
- `href`, `src`, `action`, or `formaction` fed by an unvalidated external value
- Sensitive data displayed: token, private email, internal role, technical identifier, unnecessary personal information
- Email template exposing sensitive data or non-expiring links
- Authorization logic only in Twig without controller/voter-side control

### Performance (important)

- Doctrine queries triggered in the template (lazy loading through relation getters)
- Loops over unpaginated collections
- Costly filters in loops (`|sort`, `|filter`, `|map` on large collections)
- Unversioned assets (missing cache busting)
- Repetitive blocks that could be cached with `{% cache %}`
- `|length` on a non-initialized lazy Doctrine collection
- Includes, embeds, components, or macros called in a large loop
- Repeated computation or formatting that should be prepared in controller/view model
- Images too heavy or without dimensions if visible in the template
- Excessive HTML payload for an unpaginated list

### Architecture (important)

- Business logic in the template (calculations, complex conditions, advanced formatting). Extract to a Twig Extension or service
- Template longer than 200 lines without block or partial extraction
- HTML duplicated between templates. Extract to a `_partial.html.twig` or Twig Component
- Template depending on too many implicit variables
- Role or business-state conditions duplicated in several templates
- Partial whose behavior changes heavily based on too many options
- Macro or component introduced while the project already uses a different convention
- Layout or blocks inconsistent with the rest of the application

### Accessibility (important)

- Images without `alt` attribute
- Forms without labels associated with inputs
- Links without descriptive text
- Broken heading hierarchy (h1 then h3 without h2)
- Interactive elements not keyboard-accessible
- Tables without `<thead>`, `<th>`, or `scope`
- Button rendered as link or link rendered as button without correct semantics
- Error message not associated with the affected field
- Required field indicated only by color or placeholder
- Modal, menu, or dropdown without focus/keyboard management if visible in the template
- Icon alone without accessible text (`aria-label`, hidden text, or title according to convention)
- Contrast or target size clearly problematic if visible in classes/styles

### Symfony Forms (important)

- Use `form_start()` and `form_end()` unless the project convention differs
- Keep `form_rest(form)` or `form_end(form)` for hidden fields and CSRF
- Display global errors and field errors
- Associate labels, help, and errors with fields
- Do not display or make sensitive/system fields editable
- Deletion protected by CSRF and confirmation if destructive
- Collection prototypes properly escaped and documented on the JS side if used
- Upload: correct enctype through `form_start`, user help, and visible errors

### Twig and Symfony Best Practices

- Use `{{ path('route_name') }}` instead of hardcoded URLs
- Use `{{ asset('path') }}` for static files
- Use `|trans` for strings (i18n-ready)
- Prefer Twig Components over macros for reusable elements
- `form_rest(form)` at the end of each form
- Block naming: explicit and coherent
- Use `url()` when an absolute URL is needed, especially email
- Keep conditions simple and readable
- Prefer a controller-prepared variable over a complex Twig computation
- Respect existing translation domains
- Use date, number, and currency filters according to locale/project convention
- Do not force Twig Components if the project uses macros/partials

### Template Structure

- Inheritance: one base template (`base.html.twig`), intermediate layouts if needed
- Naming: `entity/action.html.twig` coherent with routes
- Partials prefixed with `_`: `_navbar.html.twig`, `_sidebar.html.twig`
- A child template must define only needed blocks
- Partials must have a clear responsibility
- Includes must receive required variables explicitly if project convention allows it
- CRUD pages must remain coherent with layout, flashes, navigation, and existing actions

### i18n and Content (suggestion)

- User strings hardcoded while the project is translated
- Inconsistent translation domain
- Plural not handled (`transchoice` legacy or `trans` with ICU according to version)
- Dates, numbers, amounts, or currencies formatted manually
- Button or link text not explicit
- Missing error message or empty state

### UX (suggestion)

- Missing empty state on a list
- Flash messages not displayed or inconsistent with layout
- Missing pagination, sorting, or search while the list can grow
- Missing confirmation on deletion or irreversible action
- Active navigation, breadcrumbs, or back-to-list link inconsistent with project convention
- Form errors hard to understand

### Twig Emails (important)

- Use absolute URLs for external links (`url()` rather than `path()`)
- Avoid depending on JS or external CSS unsupported by email clients
- Provide alternative text or text version if the project already does so
- Check sensitive data, expirable links, and user context
- Keep styles email-compatible if the project uses inline styles
- Images with `alt` and dimensions if needed

## Useful Commands

Adapt to the project:

- `bin/console lint:twig templates/`
- `bin/console debug:twig`
- `bin/console debug:router route_name`
- `bin/console debug:translation locale`
- Targeted functional tests for the page
- Symfony Profiler for Doctrine queries and rendering time

Do not run destructive commands. Do not modify translations or assets without an explicit request.

## Do Not

- Do not add `|raw` to fix display without proven sanitization.
- Do not put complex business logic in Twig.
- Do not hide an authorization issue with a simple template-side `is_granted()`.
- Do not introduce Twig Components, Stimulus, or a design system if the project does not use them.
- Do not break legacy Symfony/Twig compatibility by using unsupported syntax.
- Do not remove `form_rest` or hidden fields without checking CSRF and method spoofing.
- Do not optimize a template by moving the problem to the controller without proof.

## Output Format

Present findings sorted by decreasing severity.

For each issue:

- File and line
- Category: security, performance, accessibility, form, architecture, i18n, UX, email, best practices
- Severity: blocking, important, or suggestion
- Evidence verified in the template or linked code
- Concrete impact
- Proposed fix
- Recommended test or verification

If no issue is found, say so clearly and mention analysis limitations: controller not read, variables unknown, profiler unavailable, tests not run.
