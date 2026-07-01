---
name: api-doc
description: "Génère ou améliore la documentation OpenAPI d'endpoints Symfony avec NelmioApiDocBundle. Utilise les attributs OpenApi, Model, Security, les routes Symfony, les DTO/Form/Entities et les groupes de sérialisation."
---

# Documentation OpenAPI avec NelmioApiDocBundle

## Processus

1. Vérifier le contexte projet :
   - `composer.json` : présence de `nelmio/api-doc-bundle`, version PHP, version Symfony.
   - `config/packages/nelmio_api_doc.yaml` : `documentation`, `areas`, `models`, `use_validation_groups`.
   - `config/routes/nelmio_api_doc.yaml` : route JSON et route UI éventuelle.
   - Contrôleur cible, DTOs, FormTypes, entités, groupes de sérialisation et contraintes Validator.
2. Analyser chaque endpoint : route, méthode HTTP, paramètres, body, réponse, sécurité et erreurs possibles.
3. Générer ou corriger les attributs OpenAPI directement dans le contrôleur.

## Imports à privilégier

```php
use Nelmio\ApiDocBundle\Attribute\Model;
use Nelmio\ApiDocBundle\Attribute\Security;
use OpenApi\Attributes as OA;
```

## Règles Nelmio

- Ne pas définir manuellement le `path` dans les attributs OpenAPI : Nelmio le déduit de `#[Route]`.
- Utiliser `#[Model(type: ...)]` dès qu'un DTO, FormType, entité ou objet PHP décrit le payload ou la réponse.
- Utiliser les `groups` dans `Model` pour aligner la documentation avec le Serializer.
- Si `nelmio_api_doc.use_validation_groups: true` est actif, aligner les groupes `Model` avec les groupes Validator.
- Pour une collection JSON, utiliser `OA\JsonContent(type: 'array', items: new OA\Items(ref: new Model(...)))`.
- Pour un objet JSON simple, utiliser `content: new Model(type: XxxDto::class, groups: [...])`.
- Ajouter `#[Security(name: 'Bearer')]` si la route utilise le schéma Bearer défini dans `nelmio_api_doc.documentation.components.securitySchemes`.
- Ne pas écrire de `OA\Schema` manuel si un modèle PHP existant permet de générer le schéma.

## Configuration Nelmio à vérifier

```yaml
nelmio_api_doc:
    documentation:
        info:
            title: 'API'
            description: 'Documentation API'
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

## Ce qu'il faut documenter pour chaque endpoint

- `#[OA\Tag]` : groupement logique
- `#[OA\Parameter]` : chaque paramètre path et query qui n'est pas déjà évident via la route
- `#[OA\RequestBody]` : body pour POST/PUT/PATCH ou endpoint avec payload
- `#[OA\Response]` : chaque code de réponse réellement possible (200, 201, 204, 210, 400, 401, 403, 404, 409, 412, 422)
- `#[Security]` : schéma de sécurité si la route est protégée

## Réponses fréquentes

- `200` : lecture ou modification avec body de réponse
- `201` : création réussie
- `204` : suppression ou action sans body
- `400` : requête invalide
- `401` : non authentifié
- `403` : non autorisé
- `404` : ressource non trouvée
- `409` : conflit
- `412` : précondition non satisfaite
- `422` : erreurs de validation

Ne documenter un code que s'il peut réellement être produit par l'endpoint ou par la couche framework utilisée.

## Patterns utiles

### Réponse objet

```php
#[OA\Response(
    response: 200,
    description: 'Retourne le détail de la ressource.',
    content: new Model(type: UserDto::class, groups: ['user:read'])
)]
```

### Réponse collection

```php
#[OA\Response(
    response: 200,
    description: 'Retourne la liste des ressources.',
    content: new OA\JsonContent(
        type: 'array',
        items: new OA\Items(ref: new Model(type: UserDto::class, groups: ['user:list']))
    )
)]
```

### Body de création ou modification

```php
#[OA\RequestBody(
    required: true,
    content: new Model(type: CreateUserDto::class, groups: ['user:create'])
)]
```

### Sécurité Bearer

```php
#[Security(name: 'Bearer')]
```

## Validation

Proposer ou lancer selon le contexte :

```bash
bin/console lint:yaml config/packages/nelmio_api_doc.yaml
bin/console lint:container
bin/console debug:router
```

Si une route JSON Nelmio existe, vérifier que `/api/doc.json` répond et que le JSON OpenAPI est généré.

## Format de sortie

Fournir les imports à ajouter, les attributs complets à placer au-dessus de chaque méthode, les ajustements éventuels de `nelmio_api_doc.yaml`, puis les commandes de vérification.
