# CLAUDE.md

Instructions projet pour Claude Code.

Ce fichier doit rester court, concret et maintenu avec le projet. Il complète les skills spécialisés disponibles dans `.claude/skills/`.

## Contexte Projet

- Nom du projet : `<nom-du-projet>`
- Domaine métier : `<description courte>`
- Type d'application : `<monolithe Symfony, API, back-office, worker, bundle, librairie>`
- Environnement principal : `<Docker, local PHP, VM, CI uniquement>`
- Niveau de criticité : `<faible, moyen, élevé>`

## Stack Technique

- PHP : `<7.0, 7.4, 8.1, 8.2, 8.3, ...>`
- Symfony : `<3.4, 4.4, 5.4, 6.4, 7.x, ...>`
- Base de données : `<MySQL, MariaDB, PostgreSQL, SQLite>`
- ORM : `<Doctrine ORM, Doctrine DBAL, autre>`
- Frontend : `<Twig, Webpack Encore, Vite, React, Vue, aucun>`
- Queue / async : `<Symfony Messenger, RabbitMQ, Redis, SQS, aucun>`
- Observabilité : `<Monolog, Sentry, Blackfire, Datadog, New Relic, Grafana, aucun>`

## Règles Générales

- Répondre en français technique, concis et actionnable.
- Préférer les conventions existantes du projet aux choix génériques.
- Lire le code avant de proposer une modification.
- Ne pas supposer une version récente de PHP ou Symfony sans preuve dans le projet.
- Respecter la compatibilité legacy quand le projet cible PHP 7.x ou une version Symfony LTS ancienne.
- Avant une modification importante, lister les fichiers et changements prévus puis attendre validation si la demande l'indique.
- Ne pas faire de refactor opportuniste sans lien direct avec la demande.

## Sécurité Et Données Sensibles

- Ne jamais lire, afficher ou committer `.env.local`.
- Ne jamais exposer secrets, tokens, mots de passe, cookies, clés API, DSN ou données personnelles sensibles.
- Ne jamais ajouter de credentials de test réalistes dans le code, les fixtures ou la documentation.
- Demander confirmation avant toute commande destructive : suppression de données, migration irréversible, reset Git, drop database, purge de cache partagé.

## Workflow De Modification

Avant de modifier :

1. Identifier le besoin exact.
2. Lire les fichiers concernés.
3. Vérifier les conventions locales.
4. Proposer un plan court si la modification touche plusieurs fichiers.

Pendant la modification :

- Garder le diff minimal.
- Ajouter ou adapter les tests quand le comportement change.
- Conserver les noms, patterns et abstractions déjà présents.
- Éviter les changements de style non demandés.

Après la modification :

- Résumer les fichiers modifiés.
- Indiquer les commandes lancées.
- Signaler clairement les tests non lancés ou impossibles à lancer.
- Mentionner les risques résiduels si nécessaire.

## Commandes Projet

Adapter cette section au projet.

```bash
# Installer les dépendances
composer install

# Lancer les tests
vendor/bin/phpunit

# Analyse statique
vendor/bin/phpstan analyse

# Qualité de code
vendor/bin/php-cs-fixer fix --dry-run --diff

# Migrations Doctrine
bin/console doctrine:migrations:status
bin/console doctrine:migrations:migrate --dry-run
```

## Skills Disponibles

Utiliser le skill adapté quand la demande correspond :

- `/review` pour une revue de code.
- `/debug` pour diagnostiquer une erreur ou une stacktrace.
- `/plan` pour préparer une implémentation avant code.
- `/test` pour créer ou corriger des tests.
- `/phpstan` pour corriger l'analyse statique.
- `/security` pour un audit sécurité.
- `/performance` pour une lenteur ou un problème de charge.
- `/doctrine` pour les entités, repositories, relations, transactions et requêtes.
- `/migration` pour les migrations Doctrine.
- `/fixtures` pour les jeux de données de test.
- `/dto` pour les DTOs et mappings.
- `/crud` pour générer ou modifier un CRUD.
- `/messenger` pour Symfony Messenger.
- `/twig` pour les templates Twig.
- `/docker` pour l'environnement local.
- `/observability` pour logs, traces, métriques et alerting.

## Conventions PHP / Symfony

- Types stricts : `<oui/non/selon fichier existant>`
- Style de typage : `<PHPDoc, types natifs, mixte>`
- Injection de dépendances : préférer l'injection constructeur sauf convention contraire.
- Contrôleurs : garder fins, déléguer la logique métier aux services.
- Doctrine : éviter les requêtes implicites en boucle et documenter les choix transactionnels.
- DTO : ne pas exposer directement les entités si le projet utilise déjà des DTOs.
- Exceptions : utiliser les exceptions métier existantes quand elles existent.

## Tests

- Framework : `<PHPUnit, Pest, Behat, Panther, autre>`
- Emplacement : `<tests/, tests/Unit, tests/Functional, ...>`
- Fixtures de test : `<DoctrineFixturesBundle, Alice, Foundry, factories custom>`
- Base de test : `<SQLite, PostgreSQL, MySQL, containers>`

Quand un bug est corrigé, ajouter un test de régression si le projet le permet.

## Git

- Ne pas modifier les fichiers sans rapport avec la demande.
- Ne pas revert des changements utilisateur sans demande explicite.
- Ne pas inclure les fichiers IDE, caches, logs ou artefacts locaux.
- Les commits doivent être scopés et décrire le changement réel.

### Branches

- `main` représente le code en production.
- `develop` représente le code déployé sur l'environnement de développement.
- Ne pas pousser directement sur `main` ou `develop`.
- Passer par une pull request pour intégrer une branche dans `develop`.
- Passer par une pull request de `develop` vers `main` pour préparer une mise en production.
- Créer les branches de travail depuis `develop`, sauf `hotfix` qui part de `main`.

Format recommandé :

```text
<type>/<id_demande>_<nom-court-en-kebab-case>
```

Types usuels :

- `feature` ou `feat` : nouvelle fonctionnalité.
- `bugfix` ou `fix` : correction d'un bug présent sur `develop`.
- `hotfix` : correction urgente d'un bug présent sur `main`.
- `chore` : nettoyage, maintenance, tâches techniques sans changement fonctionnel.
- `update` : mise à jour de dépendance, configuration ou code existant.

Exemples :

```text
feature/US-123_export-factures
fix/TICKET-456_total-commande
hotfix/TICKET-789_erreur-paiement
chore/nettoyage-services-obsoletes
```

### Commits

- Utiliser la convention Conventional Commits.
- Utiliser la référence du ticket ou de l'User Story comme scope.
- Si aucun ticket n'existe, utiliser un scope court décrivant le périmètre technique.
- Format minimal :

```text
<type>(<ticket-ou-scope>): <description courte>
```

Exemples :

```text
feat(US-123): add invoice export
fix(TICKET-456): handle refused transaction
chore(deps): update symfony packages
test(TICKET-789): add regression test
docs(readme): document local setup
```

Types fréquents :

- `feat` : fonctionnalité.
- `fix` : correction de bug.
- `docs` : documentation.
- `style` : formatage sans changement logique.
- `refactor` : refactoring sans changement fonctionnel.
- `perf` : amélioration de performance.
- `test` : ajout ou correction de tests.
- `build` : build, dépendances, packaging.
- `ci` : intégration continue.
- `chore` : maintenance.
- `revert` : annulation d'un commit.

### Pull Requests

- Une pull request doit avoir un périmètre clair et limité.
- La personne qui crée la pull request reste responsable du merge.
- En équipe, demander une review à au moins une autre personne.
- Ne pas merger une pull request marquée WIP ou dépendante d'une sous-branche non mergée.
- Mentionner dans la PR les commandes de vérification lancées et les risques connus.

### Tags Et Déploiements

- Les tags de déploiement sont créés automatiquement par la CI/CD.
- Ne pas créer ou pousser de tag manuellement sans demande explicite.
- Un merge sur `develop` peut déclencher un déploiement en développement.
- Un merge sur `main` peut déclencher un déploiement en production.

### Garde-Fous Pour Claude

- Avant un commit, vérifier le diff staged et non staged.
- Ne jamais committer `.env.local`, fichiers IDE, caches, logs, dumps ou artefacts générés.
- Ne pas utiliser `git reset --hard`, `git checkout --`, `git clean` ou une commande destructive sans confirmation explicite.
- Ne pas réécrire l'historique partagé sans confirmation explicite.
- Si des changements non liés sont présents, les laisser hors du commit.

## Notes Projet

Ajouter ici les décisions locales importantes :

- `<decision 1>`
- `<decision 2>`
- `<decision 3>`
