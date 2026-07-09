---
name: dto
description: "Génère et structure des DTOs pour projet PHP/Symfony. Produit des DTOs readonly typés, validés, compatibles Serializer/API Platform/Messenger, avec mapping explicite depuis les entités et application vers les entités via service, handler ou factory."
---

# Génération de DTOs

## Périmètre

Si l'utilisateur précise une entité, un endpoint, un formulaire, une commande, un message Messenger, un use case ou un fichier, limiter le travail à ce périmètre.

Sinon :

- Identifier les DTOs existants et leurs conventions.
- Lire l'entité, le contrôleur, le handler, le form type ou le serializer concerné.
- Vérifier l'architecture du projet : MVC classique, API Platform, hexagonale, CQRS, Messenger.
- Demander le cas d'usage si le sens du DTO n'est pas clair : input, output, update, query, command ou message.

Ne pas générer un DTO générique qui expose toute l'entité sans besoin identifié.

## État des lieux

Avant de générer ou modifier des DTOs, inspecter selon le projet :

- `composer.json` : version PHP, Symfony Validator, Serializer, API Platform, Messenger
- DTOs existants : namespace, suffixe, readonly, validation, mapping
- Entités Doctrine liées : champs, nullabilité, relations, enums, value objects
- Contrôleurs, handlers, processors, providers, forms ou commands qui consomment les DTOs
- Règles de validation existantes et groupes de validation
- Configuration serializer si utile : groupes, name converter, normalizers custom

Ne pas lire `.env.local`.

## Types de DTO

### Input DTO (création/modification)

- Classes `readonly`, promotion de propriétés dans le constructeur
- Contraintes de validation (`#[Assert\...]`)
- Ne contient que les champs modifiables par le client
- Porte des scalaires, enums, value objects simples ou identifiants de relation, pas des entités Doctrine
- Peut être spécialisé par opération : `CreateUserDto`, `UpdateUserDto`, `ChangeUserEmailDto`
- Pour un update partiel, distinguer champ absent, champ présent à `null` et champ présent avec valeur si l'API le permet

### Output DTO (lecture)

- Pas de contraintes de validation
- Peut contenir des champs calculés ou formatés
- Méthode statique `fromEntity()` pour le mapping
- Expose le contrat public attendu, pas forcément toute l'entité
- Peut agréger plusieurs entités ou valeurs calculées si le use case l'exige
- Ne doit pas déclencher de requêtes cachées non maîtrisées via des relations lazy

### Command/Query DTO (CQRS)

- Command : intention de modification
- Query : demande de données
- Immutables (readonly)
- Ne contient que les données nécessaires au use case
- Peut être indépendant du transport HTTP, CLI ou Messenger

### Filter/Search DTO

- Représente des critères de recherche, tri, pagination et filtres.
- Valide les bornes : page positive, limite maximale, tri autorisé, enum de statut.
- Utilise des valeurs nullable seulement si l'absence de critère est un état valide.

### Message DTO (Messenger)

- Message immutable et sérialisable.
- Contient des identifiants stables plutôt que des entités Doctrine.
- Ne contient pas de service, ressource, closure, objet non sérialisable ou données sensibles inutiles.
- Reste compatible entre deux déploiements successifs si des messages peuvent rester en file.

### Form DTO

- Peut découpler un formulaire Symfony de l'entité Doctrine.
- Porte les contraintes de validation proches de l'input utilisateur.
- Le mapping vers l'entité doit rester dans le contrôleur, un form handler ou un service applicatif.

## Règles

- Classes `readonly` (PHP 8.2+)
- Pas de logique métier dans le DTO
- Pas de dépendance vers Doctrine ou Symfony (sauf Assert)
- Nommage : `Create{Entity}Dto`, `Update{Entity}Dto`, `{Entity}Dto`
- Propriétés typées précisément : éviter `mixed`, `array` ou `object` si une forme précise existe
- Préférer les types natifs, enums et value objects simples aux chaînes magiques
- Utiliser PHPDoc seulement pour les generics, array shapes ou types non exprimables en PHP natif
- Ne pas rendre nullable uniquement pour faciliter le mapping
- Ne pas exposer un champ sensible par défaut : mot de passe hashé, token, secret, donnée personnelle non nécessaire
- Ne pas réutiliser le même DTO pour create, update et output si les contrats divergent

## Validation

- Mettre les contraintes Symfony Validator sur les input DTOs.
- Ne pas mettre de validation utilisateur sur les output DTOs.
- Utiliser `#[Assert\NotBlank]` pour une chaîne obligatoire non vide, `#[Assert\NotNull]` pour une valeur obligatoire pouvant être `0` ou `false`.
- Ajouter `#[Assert\Email]`, `#[Assert\Length]`, `#[Assert\Choice]`, `#[Assert\Regex]`, `#[Assert\Uuid]`, `#[Assert\Positive]` selon le contrat réel.
- Pour les collections, utiliser `#[Assert\All]`, `#[Assert\Count]`, et documenter le type : `@param list<string> $tags`.
- Pour les objets imbriqués, utiliser `#[Assert\Valid]`.
- Pour les dates, préciser le type attendu (`DateTimeImmutable`, chaîne ISO 8601, date seule) et convertir à la frontière applicative.
- Pour les fichiers uploadés, utiliser `UploadedFile` seulement côté input/form et valider taille, MIME réel et extension si nécessaire.
- Utiliser les groupes de validation seulement si le projet les utilise déjà ou si le cas create/update le justifie clairement.

## Mapping

### Entité vers Output DTO

Une méthode statique `fromEntity()` sur l'output DTO est acceptable pour un mapping simple :

```php
public static function fromEntity(User $user): self
{
    return new self(
        id: $user->getId(),
        email: $user->getEmail(),
        fullName: $user->getFullName(),
    );
}
```

Règles :

- Le DTO peut connaître l'entité qu'il expose.
- Le mapping doit rester simple : lecture de champs, transformation légère, format public.
- Ne pas mettre de requêtes, repository, EntityManager ou règles métier dans `fromEntity()`.
- Pour une liste, prévoir une factory explicite ou une méthode dédiée si le projet le fait déjà.
- Pour éviter les N+1, vérifier que les relations utilisées par le DTO sont chargées par la requête.

### Input DTO vers entité

Éviter `toEntity()` ou `applyToEntity()` dans l'input DTO.

Préférer :

- une factory pour créer une entité ;
- un handler applicatif pour appliquer une commande ;
- une méthode métier de l'entité appelée par le service ;
- un processor API Platform si le projet l'utilise.

Exemple :

```php
final class UpdateUserHandler
{
    public function handle(User $user, UpdateUserDto $dto): void
    {
        $user->rename($dto->firstName, $dto->lastName);
        $user->changeEmail($dto->email);
    }
}
```

Règles :

- Le DTO ne doit pas dépendre de Doctrine, d'un repository ou du container.
- Les relations entrantes doivent être représentées par des IDs ou des codes métier.
- Le handler résout les entités liées, vérifie les droits et applique les invariants métier.
- Ne pas hydrater directement une entité Doctrine depuis une requête externe.
- Ne pas bypasser les méthodes métier de l'entité avec des setters aveugles si le domaine expose des méthodes explicites.

## Serializer et API

- Séparer le contrat public du modèle Doctrine.
- Utiliser les groupes serializer seulement s'ils sont déjà la convention du projet.
- Ne pas dépendre uniquement des groupes serializer pour protéger des champs sensibles : définir un output DTO explicite si l'API est sensible.
- Garder les noms de champs stables pour les clients API.
- Prévoir un DTO d'erreur ou un format de validation cohérent si l'endpoint expose des erreurs structurées.
- Pour API Platform, respecter les conventions du projet : input/output DTO, provider, processor, resource class ou state options.

## Tests et validation

À ajouter ou proposer selon le risque :

- Test de validation d'un input DTO : cas valide, champs manquants, formats invalides, limites.
- Test de mapping `fromEntity()` si le DTO contient des champs calculés, formats publics ou relations.
- Test de handler/factory pour le mapping input DTO vers entité.
- Test fonctionnel de désérialisation et validation pour un endpoint API.
- Test de non-régression pour les champs sensibles non exposés.

Commandes utiles si présentes :

- `vendor/bin/phpunit --filter NomDuTest`
- `bin/console lint:container`
- `bin/console debug:validator App\\Dto\\ExampleDto`
- PHPStan sur le périmètre modifié

## Format de sortie

Pour une génération ou modification, fournir :

- Fichiers créés ou modifiés
- Type de DTO et use case couvert
- Contraintes de validation ajoutées
- Stratégie de mapping retenue
- Choix de typage importants
- Tests ou commandes de validation lancés
- Limites ou hypothèses restantes

Générer les classes DTO complètes avec namespace, imports, types, validation, mapping pertinent et exemple d'utilisation dans le handler, contrôleur ou processor si nécessaire.
