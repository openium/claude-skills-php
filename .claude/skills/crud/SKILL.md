---
name: crud
description: "Génère ou complète un CRUD Symfony adapté au projet. Produit entité Doctrine, repository, migration, contrôleur Twig/API, form, DTOs, handlers, templates, tests et fixtures selon l'architecture existante, en respectant sécurité, validation, versions PHP/Symfony et conventions locales."
---

# Génération de CRUD Symfony

## Périmètre

Déterminer si l'utilisateur demande :

- Un CRUD complet pour une nouvelle ressource.
- Un CRUD pour une entité Doctrine existante.
- Un CRUD web Twig, API JSON, API Platform, back-office ou EasyAdmin.
- Une action ciblée : create, read, update, delete, listing, recherche, export.
- Une correction ou extension d'un CRUD existant.

Si le besoin est ambigu, demander le type d'interface attendu et les champs modifiables.

Ne pas générer un CRUD complet si l'utilisateur demande seulement une action ou un endpoint.

## État des lieux

Avant de générer, inspecter selon le projet :

- `composer.json` : version PHP, Symfony, Doctrine, Twig, Form, Validator, Security, API Platform, EasyAdmin
- Architecture : MVC classique, API-only, API Platform, hexagonale, CQRS, back-office
- Entités, repositories, DTOs, handlers, forms, controllers et templates existants
- Routing : attributs PHP, annotations, YAML ou XML
- Sécurité : `security.yaml`, voters, `#[IsGranted]`, access control, rôles existants
- Doctrine : migrations, conventions de mapping, types custom, relations, index
- Tests : structure `tests/`, fixtures, base classes, conventions de nommage
- Templates : layout, composants, partials, pagination, design system existant

Ne jamais lire ni modifier `.env.local`.

## Processus

1. Comprendre l'entité demandée (champs, types, relations, contraintes)
2. Détecter l'architecture du projet et la version PHP/Symfony
3. Identifier les conventions locales avant de créer de nouveaux fichiers
4. Définir le type de CRUD : Twig, API JSON, API Platform, back-office, CQRS
5. Générer les fichiers nécessaires dans le bon ordre
6. Ajouter ou proposer migration, fixtures et tests ciblés
7. Lancer les validations adaptées si possible

## Détection du contexte

- Templates Twig présents ? -> CRUD web classique
- API Platform installé ? -> Resource API Platform
- Uniquement des contrôleurs JSON ? -> CRUD API sans Twig
- Architecture hexagonale ? -> Séparer Domain/Application/Infrastructure
- EasyAdmin installé ? -> CRUD back-office via dashboard/controller EasyAdmin
- DTOs/handlers existants ? -> Ne pas hydrater l'entité directement depuis la requête
- Projet legacy PHP 7.x/Symfony ancien ? -> Suivre annotations/YAML/XML et éviter attributs PHP 8
- Projet PHP 8+/Symfony récent ? -> Attributs, readonly, enums ou types modernes seulement si déjà compatibles

## Fichiers générés

Adapter la liste au contexte réel. Ne pas créer les fichiers inutiles pour le type de CRUD demandé.

### 1. Entité Doctrine

- Mapping selon la convention du projet : attributs PHP 8+, annotations, YAML ou XML
- Contraintes de validation (`#[Assert\NotBlank]`, etc.)
- Relations si demandées
- Méthodes métier (pas de setters aveugles si architecture hexagonale)
- Types Doctrine cohérents : string length, text, decimal, datetime immutable, enum si supporté
- Champs sensibles exclus des formulaires et sorties publiques par défaut
- Méthodes d'association qui maintiennent les deux côtés des relations bidirectionnelles

### 2. Repository

- Étend `ServiceEntityRepository`
- Méthodes `save()` et `remove()` avec flush optionnel
- Méthodes de recherche spécifiques si identifiables
- Requêtes paginées ou filtrées si listing, recherche ou tri demandés
- Pas de logique métier dans le repository
- Types PHPDoc/generics si PHPStan ou conventions projet les utilisent

### 3. Migration

- Générer ou signaler la migration nécessaire après modification d'entité
- Vérifier `NOT NULL`, valeurs par défaut, contraintes uniques, index et clés étrangères
- Ajouter un index sur les colonnes utilisées en `WHERE`, `JOIN`, `ORDER BY`
- Éviter les opérations destructives non demandées
- Pour une entité existante, ne pas supposer que la table est vide

### 4. Contrôleur Twig

- Routes selon la convention du projet : attributs, annotations, YAML ou XML
- Actions : `index`, `show`, `new`, `edit`, `delete`
- Contrôle d'accès sur chaque action : `#[IsGranted]`, `denyAccessUnlessGranted`, voter ou `access_control`
- Flash messages en français
- Redirection après création/modification/suppression
- CSRF obligatoire sur suppression et actions mutantes
- Gestion explicite du `404` via repository nullable ou param converter selon convention
- Pas de logique métier lourde dans le contrôleur

### 5. Contrôleur API JSON

- DTO input/output si le projet en utilise ou si l'endpoint est exposé à l'extérieur
- Validation serveur avant modification
- Codes HTTP cohérents : `201`, `200`, `204`, `400`, `403`, `404`, `422`
- Contrôle d'accès par action et vérification propriétaire si nécessaire
- Ne pas hydrater directement une entité Doctrine depuis une requête externe
- Mapping input DTO vers entité via handler, service, factory ou processor

### 6. API Platform

- Respecter les conventions du projet : resource sur entité ou classe dédiée
- Utiliser provider/processor si la logique dépasse le CRUD trivial
- Input/output DTO pour protéger le contrat public si nécessaire
- Security expressions, voters ou processors pour les droits
- Normalization/denormalization groups seulement si déjà convention projet

### 7. Form Type

- Champs typés avec les bons FormType : TextType, DateType, EnumType, EntityType, FileType, MoneyType, etc.
- Labels en français
- Contraintes de validation cohérentes avec l'entité
- Options (placeholder, required, help)
- Exclure les champs non modifiables : id, owner, createdAt, updatedAt, rôles sensibles, statut système
- Gérer upload, enums, relations et champs optionnels explicitement
- `data_class` cohérent : entité ou DTO selon architecture

### 8. Templates Twig (si web)

- `index.html.twig` : liste paginée avec tableau
- `show.html.twig` : détail de l'entité
- `new.html.twig` : formulaire de création
- `edit.html.twig` : formulaire de modification
- `_form.html.twig` : partial formulaire partagé entre new et edit
- `_delete_form.html.twig` : formulaire de suppression avec confirmation
- Étendre le layout existant
- Utiliser les composants, classes CSS et conventions Twig du projet
- Échapper les données par défaut, éviter `|raw` sauf contenu déjà sanitized
- Afficher les actions selon les permissions si la convention projet le fait
- Prévoir état vide, pagination, tri ou recherche si demandés

### 9. DTOs, handlers et services

- DTO input pour create/update si API ou architecture applicative
- DTO output pour contrat public si API ou serializer sensible
- Handler/service applicatif pour appliquer les modifications
- Factory si la création d'entité a des invariants
- Voter si les droits dépendent de l'objet ou du propriétaire

### 10. Tests

- Test unitaire de l'entité : validation, invariants, méthodes métier
- Test fonctionnel du contrôleur Twig : routes, codes HTTP, redirections, formulaires, CSRF
- Test API : payload valide, payload invalide, `404`, `403`, `422`, suppression
- Test sécurité : accès anonyme, rôle insuffisant, IDOR/propriétaire
- Test repository si requête custom, pagination ou filtre complexe
- Fixtures minimales adaptées aux scénarios testés

### 11. Fixtures

- Créer seulement les données nécessaires aux tests ou scénarios
- Respecter relations, contraintes uniques et validation
- Utiliser le format déjà présent : DoctrineFixturesBundle, Alice, Foundry ou fixtures dédiées

## Conventions

- Nommage des routes : `app_entity_action` (ex: `app_user_index`, `app_user_new`)
- Nommage des templates : `entity/action.html.twig`
- Messages flash : `success`, `error`
- Pagination : KnpPaginatorBundle si installé, sinon pagination manuelle
- Namespaces, suffixes et dossiers selon les conventions existantes
- Ne pas introduire un nouveau style si le projet en a déjà un
- Pour PHP 7.x, ne pas générer d'attributs, enums, readonly ou syntaxe PHP 8
- Pour Symfony ancien, respecter annotations/YAML/XML si c'est la convention

## Sécurité

- Chaque action sensible doit avoir un contrôle d'accès.
- Pour les ressources propriétaires, vérifier l'accès à l'objet, pas seulement le rôle.
- Protéger les suppressions avec CSRF côté Twig.
- Ne pas exposer champs sensibles dans API, templates ou formulaires.
- Ne pas faire confiance aux ids envoyés par l'utilisateur pour propriétaire, organisation ou tenant.
- Journaliser ou gérer proprement les actions critiques si le projet le fait déjà.

## Validation

- Les règles métier doivent vivre dans l'entité, le domain service ou le handler, pas seulement dans le formulaire.
- Les contraintes `Assert` doivent correspondre aux contraintes DB.
- Gérer les champs optionnels, nullables et valeurs par défaut explicitement.
- Pour les relations, valider que l'entité liée existe et que l'utilisateur peut l'utiliser.
- Pour les uploads, valider taille, MIME réel, extension et stockage.

## Ne pas faire

- Ne pas générer un CRUD complet sans besoin explicite.
- Ne pas contourner les méthodes métier avec des setters aveugles en architecture domaine.
- Ne pas hydrater directement une entité Doctrine depuis une requête externe sensible.
- Ne pas ajouter d'accès admin ou de route mutante sans contrôle d'accès.
- Ne pas supprimer ou modifier des migrations existantes déjà exécutées sans confirmation.
- Ne pas modifier `.env.local`.
- Ne pas introduire une dépendance, un bundle ou une architecture nouvelle sans demande explicite.
- Ne pas créer une API publique qui expose toute l'entité par défaut.

## Commandes utiles

Adapter au projet :

- `bin/console make:entity` ou `make:crud` seulement si le projet l'utilise et si l'utilisateur accepte le scaffold
- `bin/console doctrine:migrations:diff`
- `bin/console doctrine:migrations:migrate --dry-run`
- `bin/console doctrine:schema:validate`
- `bin/console lint:twig templates/`
- `bin/console lint:container`
- `vendor/bin/phpunit --filter NomDuTest`
- PHPStan sur le périmètre modifié si présent

Ne pas lancer de migration réelle, purge de base ou commande destructive sans confirmation explicite.

## Format de sortie

Pour une génération ou modification, fournir :

- Architecture retenue : Twig, API JSON, API Platform, back-office, CQRS
- Fichiers créés ou modifiés
- Entité, relations, contraintes et migration nécessaires
- Sécurité appliquée par action
- Validation et mapping retenus
- Tests et fixtures ajoutés ou proposés
- Commandes lancées et résultat
- Hypothèses ou limites restantes

Générer chaque fichier avec son chemin complet, prêt à utiliser, et indiquer l'ordre de création : entité, migration, repository, DTO/handler, controller, form/templates, fixtures, tests.
