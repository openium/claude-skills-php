---
name: api-doc
description: "Generates or improves OpenAPI documentation for Symfony endpoints with NelmioApiDocBundle. Uses OpenApi, Model, and Security attributes, Symfony routes, DTO/Form/Entities, and serialization groups."
---

# OpenAPI Documentation with NelmioApiDocBundle

## Process

1. Check the project context:
   - `composer.json`: presence of `nelmio/api-doc-bundle`, PHP version, Symfony version.
   - `config/packages/nelmio_api_doc.yaml`: `documentation`, `areas`, `models`, `use_validation_groups`.
   - `config/routes/nelmio_api_doc.yaml`: possible JSON route and UI route.
   - Target controller, DTOs, FormTypes, entities, serialization groups, and Validator constraints.
2. Analyze each endpoint: route, HTTP method, parameters, body, response, security, and possible errors.
3. Generate or fix OpenAPI attributes directly in the controller.

## Imports to Prefer

```php
use Nelmio\ApiDocBundle\Attribute\Model;
use Nelmio\ApiDocBundle\Attribute\Security;
use OpenApi\Attributes as OA;
```

## Nelmio Rules

- Do not manually define the `path` in OpenAPI attributes: Nelmio infers it from `#[Route]`.
- Use `#[Model(type: ...)]` whenever a DTO, FormType, entity, or PHP object describes the payload or response.
- Use `groups` in `Model` to align documentation with the Serializer.
- If `nelmio_api_doc.use_validation_groups: true` is active, align `Model` groups with Validator groups.
- For a JSON collection, use `OA\JsonContent(type: 'array', items: new OA\Items(ref: new Model(...)))`.
- For a simple JSON object, use `content: new Model(type: XxxDto::class, groups: [...])`.
- Add `#[Security(name: 'Bearer')]` if the route uses the Bearer scheme defined in `nelmio_api_doc.documentation.components.securitySchemes`.
- Do not write a manual `OA\Schema` if an existing PHP model can generate the schema.

## Nelmio Configuration to Check

```yaml
nelmio_api_doc:
    documentation:
        info:
            title: 'API'
            description: 'API documentation'
            version: '1.0.0'
        components:
            securitySchemes:
                Bearer:
                    type: http
                    scheme: bearer
                    bearerFormat: JWT
    areas:
        path_patterns:
            - ^/api(?!/doc$)
```

## What to Document for Each Endpoint

- `#[OA\Tag]`: logical grouping
- `#[OA\Parameter]`: each path and query parameter that is not already obvious through the route
- `#[OA\RequestBody]`: body for POST/PUT/PATCH or endpoint with payload
- `#[OA\Response]`: each response code that can actually occur (200, 201, 204, 210, 400, 401, 403, 404, 409, 412, 422)
- `#[Security]`: security scheme if the route is protected

## Common Responses

- `200`: read or modification with response body
- `201`: successful creation
- `204`: deletion or action without body
- `400`: invalid request
- `401`: unauthenticated
- `403`: unauthorized
- `404`: resource not found
- `409`: conflict
- `412`: precondition not satisfied
- `422`: validation errors

Document a code only if it can actually be produced by the endpoint or by the framework layer used.

## Useful Patterns

### Object Response

```php
#[OA\Response(
    response: 200,
    description: 'Returns the resource details.',
    content: new Model(type: UserDto::class, groups: ['user:read'])
)]
```

### Collection Response

```php
#[OA\Response(
    response: 200,
    description: 'Returns the resource list.',
    content: new OA\JsonContent(
        type: 'array',
        items: new OA\Items(ref: new Model(type: UserDto::class, groups: ['user:list']))
    )
)]
```

### Creation or Modification Body

```php
#[OA\RequestBody(
    required: true,
    content: new Model(type: CreateUserDto::class, groups: ['user:create'])
)]
```

### Bearer Security

```php
#[Security(name: 'Bearer')]
```

## Validation

Propose or run according to context:

```bash
bin/console lint:yaml config/packages/nelmio_api_doc.yaml
bin/console lint:container
bin/console debug:router
```

If a Nelmio JSON route exists, verify that `/api/doc.json` responds and that OpenAPI JSON is generated.

## Output Format

Provide the imports to add, the complete attributes to place above each method, any `nelmio_api_doc.yaml` adjustments, then the verification commands.
