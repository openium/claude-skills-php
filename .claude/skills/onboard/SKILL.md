---
name: onboard
description: "Génère un guide d'onboarding technique pour un projet PHP/Symfony. Analyse le repo pour documenter stack, installation locale, architecture, workflows, commandes, conventions, CI/CD et points d'attention vérifiables pour un nouveau développeur."
---

# Onboarding technique

## Périmètre

Par défaut, produire un guide pour le projet entier.

Si l'utilisateur précise un module, dossier ou objectif, limiter l'onboarding à ce périmètre.

Ne pas modifier le repository, sauf demande explicite.

## Sources à inspecter

Lire selon leur présence :

- `README*`, `docs/`
- `composer.json`, `composer.lock`, `symfony.lock`
- `Makefile`, `Taskfile*`, scripts Composer
- `Dockerfile`, `docker-compose*.yml`
- `.env`, `.env.dist`, `.env.example`
- `config/bundles.php`, `config/packages/`, `config/routes/`
- `src/`, `tests/`, `migrations/`, `templates/`
- `assets/`, `package.json`, `yarn.lock`, `pnpm-lock.yaml`, `package-lock.json`
- `.github/workflows/`, `.gitlab-ci.yml`

Ne jamais lire ni afficher `.env.local`.

## Règles

- Ne documenter que ce qui est vérifiable dans le repo.
- Distinguer les faits vérifiés des points à confirmer.
- Citer les fichiers sources consultés quand cela aide à comprendre.
- Ne pas inventer de commandes : préférer Makefile, scripts Composer, Docker Compose ou scripts npm réellement présents.
- Si README, Docker, Composer ou CI se contredisent, signaler l'écart.
- Ne jamais afficher de secret. Pour les variables d'environnement, documenter seulement le nom et le rôle si vérifiable.
- Si un secret semble committé, le signaler sans le recopier.

## Sections du guide

### Vue d'ensemble

Objectif du projet, type d'application, composants principaux et services externes identifiables.

### Stack technique

Version PHP, Symfony, base de données, outils (Redis, RabbitMQ, etc.), bundles principaux.

### Installation locale

Prérequis, installation des dépendances, configuration d'environnement, démarrage applicatif, Docker si présent.

### Configuration

Variables d'environnement documentées, fichiers de configuration importants, bundles activés, routes principales.

### Base de données

Type de base, migrations, fixtures, commandes de création/reset si vérifiables.

### Architecture

Structure des dossiers `src/`, pattern architectural (MVC, hexagonal, DDD, CQRS), couches et responsabilités, conventions de nommage.

### Workflows de développement

Commandes de dev, cache, assets, workers, Messenger, cron ou tâches planifiées si présents.

### Tests et qualité

Tests, linter, PHPStan, CS-Fixer, migrations, cache, assets, Makefile.

### Conventions de code

Standard de codage (PSR-12, custom), organisation des tests, git workflow, CI/CD.

### CI/CD

Pipelines, jobs, versions PHP, étapes de test, build, déploiement si vérifiables.

### Points d'attention

Zones de dette technique prouvées par TODO/FIXME, documentation contradictoire, fichiers sensibles, particularités du projet, dépendances obsolètes si vérifiables.

## Format de sortie

Produire un document Markdown factuel et concis.

Inclure :

- Commandes utiles en blocs `sh`
- Références aux fichiers consultés quand pertinent
- Section "À confirmer" pour les informations non vérifiables
- Section "Écarts ou manques de documentation" si nécessaire

Ne pas inclure de secrets ni de contenu `.env.local`.
