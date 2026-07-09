---
name: plan
description: "Planifie l'implémentation d'une feature, bugfix, refactor, migration ou ticket dans un projet PHP/Symfony. Analyse le contexte, clarifie les inconnues, identifie fichiers, étapes, tests, risques, commandes de validation et ordre d'exécution sans modifier le code avant validation."
---

# Planification d'implémentation

## Périmètre

Déterminer si l'utilisateur demande un plan pour :

- Feature métier
- Bugfix
- Refactor
- Migration Doctrine ou données
- Endpoint API ou contrat DTO
- Interface Twig/form/back-office
- Commande Symfony
- Message ou workflow Messenger
- Upgrade PHP/Symfony/dépendance
- Audit sécurité/performance ou correction ciblée

Si l'utilisateur demande seulement un plan, ne pas modifier le code.

Si le périmètre est ambigu, poser uniquement les questions bloquantes. Pour les points non bloquants, formuler des hypothèses explicites.

## Objectif

Produire un plan exécutable, ordonné et adapté au projet.

Le plan doit :

- Respecter les conventions existantes.
- Garder le projet compilable à chaque étape si possible.
- Séparer code, config, migration, fixtures et tests.
- Identifier les risques avant l'implémentation.
- Éviter l'over-engineering.
- Donner les commandes de validation utiles.

Ne pas transformer le plan en refactor large non demandé.

## État des lieux

Inspecter selon le sujet :

- `composer.json` : version PHP, Symfony, dépendances clés, scripts
- Architecture : MVC, hexagonale, CQRS, API Platform, Twig, Messenger, legacy PHP/Symfony
- Fichiers proches du périmètre : entités, repositories, services, handlers, controllers, forms, templates, DTOs
- Configuration : routes, services, security, messenger, doctrine, serializer, validator
- Tests existants et conventions de `tests/`
- Migrations et schéma Doctrine si données impactées
- Fixtures si tests fonctionnels ou intégration nécessaires
- CI, PHPStan, Rector, linters si pertinents

Ne jamais lire ni modifier `.env.local`.

## Questions ouvertes

Lister uniquement :

- Les décisions métier nécessaires pour éviter une mauvaise implémentation.
- Les choix techniques qui changent fortement l'architecture ou le coût.
- Les informations sans lesquelles le plan serait trompeur.

Pour le reste, indiquer les hypothèses :

- Version PHP/Symfony supposée
- Architecture retenue
- Format API ou UI supposé
- Niveau de compatibilité ou de BC attendu
- Volume de données supposé

## Structure du plan

### 1. Résumé

Une phrase qui reformule le besoin pour validation.

### 2. Analyse de l'existant

Fichiers et classes impactés, patterns déjà utilisés, contraintes identifiées.

### 3. Plan d'implémentation

Pour chaque étape : fichier à créer ou modifier, ce qui change et pourquoi, dépendances avec les autres étapes. Ordonner pour que le code compile à chaque étape intermédiaire.

### 4. Fichiers à créer

Chemin complet, responsabilité, namespace et couche architecturale.

### 5. Fichiers à modifier

Chemin complet, nature de la modification, risque d'impact.

### 6. Tests à écrire

Tests unitaires, fonctionnels, cas limites à couvrir.

### 7. Risques et points d'attention

Impact sur l'existant, migrations nécessaires, changements de configuration, compatibilité ascendante.

### 8. Estimation de complexité

Nombre de fichiers impactés, complexité relative (simple / modéré / complexe), pré-requis éventuels.

## Analyse d'architecture

Adapter le plan au style du projet :

- MVC Symfony classique : controller, form, Twig, repository, service si nécessaire.
- API JSON : DTO input/output, validation, handler/service, réponse et erreurs structurées.
- API Platform : resource, provider, processor, input/output DTO si convention projet.
- Hexagonal/CQRS : Domain, Application, Infrastructure séparés.
- Messenger : message immutable, handler idempotent, transport, retry/failure.
- Legacy Symfony/PHP 7.x : annotations/YAML/XML, pas d'attributs PHP 8, pas de readonly/enums.
- PHP 8+/Symfony récent : attributs et types modernes seulement si compatibles avec le projet.

Ne pas introduire un nouveau style architectural si le projet a déjà une convention claire.

## Découpage des étapes

Ordonner les étapes pour réduire les risques :

1. Adapter ou créer le modèle de données minimal.
2. Ajouter migration et stratégie de données si nécessaire.
3. Ajouter services/handlers/use cases.
4. Ajouter interface HTTP, CLI, Messenger ou UI.
5. Ajouter validation, sécurité et gestion d'erreurs.
6. Ajouter fixtures si utiles.
7. Ajouter tests ciblés.
8. Lancer validations.

Pour une migration de données ou un changement incompatible, proposer un déploiement en plusieurs étapes si nécessaire.

## Risques à analyser

- **BC break** : signature publique, contrat API, payload, route, événement, message.
- **Données** : migration destructive, backfill, contraintes, rollback.
- **Sécurité** : droits, IDOR, CSRF, exposition de champs sensibles, validation serveur.
- **Performance** : N+1, pagination, batch, index, appels externes.
- **Concurrence** : transaction, verrou, double traitement, idempotence.
- **Messenger** : retry, failure transport, dispatch avant commit, messages incompatibles.
- **UX** : erreurs formulaire, états vides, redirections, messages flash.
- **Tests** : comportement critique non couvert, fixtures manquantes, test trop fragile.

## Stratégie de tests

Prévoir selon le changement :

- Unitaires : value objects, services, règles métier, handlers simples.
- Fonctionnels : controller, form, Twig, routes, sécurité, redirections.
- Intégration : repository, Doctrine, Messenger, services réels.
- API : payload valide/invalide, codes HTTP, erreurs, sérialisation.
- Migration : schema validate, dry-run, rollback si disponible.
- Performance : cas volumineux, pagination, batch.
- Régression : bugfix avec test qui échoue avant correction.

Indiquer les fixtures nécessaires et éviter de mocker la base pour un comportement Doctrine important.

## Validation technique

Proposer seulement les commandes adaptées au projet :

- `vendor/bin/phpunit --filter NomDuTest`
- Script Composer de tests si présent
- PHPStan sur le périmètre modifié
- `bin/console lint:container`
- `bin/console lint:twig templates/`
- `bin/console lint:yaml config/`
- `bin/console doctrine:schema:validate`
- `bin/console doctrine:migrations:migrate --dry-run`
- `bin/console debug:router`
- `bin/console debug:messenger`

Ne pas lancer de commande destructive, migration réelle, purge ou retry massif dans un plan.

## Règles

- Ne pas proposer d'over-engineering
- Respecter les patterns existants du projet
- Signaler les opportunités de refactoring sans les inclure sauf demande explicite
- Ne pas ajouter de nouveau bundle, abstraction ou architecture sans bénéfice clair
- Ne pas mélanger feature, refactor large et upgrade sauf demande explicite
- Ne pas masquer une incertitude importante : la lister comme décision à valider
- Ne pas supposer une version PHP/Symfony récente sans preuve
- Ne pas proposer une solution qui contourne sécurité, validation ou tests

## Format de sortie

Présenter le plan avec :

1. **Résumé** : besoin reformulé en une phrase.
2. **Hypothèses** : choix supposés si non confirmés.
3. **Contexte existant** : fichiers, conventions et contraintes observées.
4. **Plan étape par étape** : ordre, fichier, changement, raison, dépendances.
5. **Fichiers à créer** : chemin, responsabilité, couche.
6. **Fichiers à modifier** : chemin, nature du changement, risque.
7. **Tests** : tests à écrire ou adapter, fixtures nécessaires.
8. **Validation** : commandes à lancer.
9. **Risques** : impacts et mitigations.
10. **Complexité** : simple, modéré ou complexe, avec justification courte.
11. **Décisions à valider** : seulement si nécessaire avant implémentation.
