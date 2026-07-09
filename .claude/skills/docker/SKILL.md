---
name: docker
description: "Génère ou audite un environnement Docker pour projet PHP/Symfony. Produit Dockerfile, docker-compose, configs web, services DB/cache/queue/mail/assets, Makefile et validations, en respectant versions PHP/Symfony, sécurité, performance, dev/test/CI et conventions existantes."
---

# Environnement Docker pour PHP/Symfony

## Périmètre

Déterminer si l'utilisateur demande :

- Création d'un environnement Docker local complet.
- Audit ou correction d'un Dockerfile / docker-compose existant.
- Ajout d'un service : DB, Redis, RabbitMQ, Mailpit, Node, worker, search.
- Environnement de dev, test, CI ou production.
- Correction ciblée : permissions, extensions PHP, Xdebug, performance, healthcheck, volumes.

Si le besoin est ambigu, demander l'environnement cible : local dev, test, CI ou prod.

Ne pas écraser une configuration Docker existante sans analyser ses conventions.

## État des lieux

Inspecter selon le projet :

- `composer.json` : version PHP, extensions requises, scripts Composer, Symfony
- `composer.lock` si présent pour confirmer extensions et packages
- `.env` et `.env.dist` si présents, sans lire `.env.local`
- Dockerfile, `docker-compose.yml`, `compose.yaml`, overrides et scripts existants
- `package.json`, lockfiles, Vite, Webpack Encore, AssetMapper, npm/yarn/pnpm
- Config Symfony : Doctrine, Messenger, Mailer, Redis/cache, Mercure, search
- CI GitHub/GitLab si Docker doit s'intégrer aux tests
- Contraintes legacy : PHP 7.x, anciennes extensions, Apache existant, Symfony ancien

Ne jamais lire ni afficher `.env.local` ou des secrets.

## Détection des besoins

Depuis `composer.json`, `.env`, et la configuration :

- Version PHP, extensions nécessaires
- Base de données : `DATABASE_URL`
- Redis : `REDIS_URL`
- RabbitMQ : `MESSENGER_TRANSPORT_DSN` avec `amqp://`
- Mailer : `MAILER_DSN`
- Node.js : `package.json` présent
- Search : Elasticsearch, OpenSearch, Meilisearch selon packages/config
- Mercure : config ou package Mercure
- Stockage/fichiers : uploads, S3/MinIO si package ou config explicite
- Tests : base dédiée ou override `when@test`

## Compatibilité PHP/Symfony

- Aligner l'image PHP sur la contrainte du projet, pas sur la dernière version disponible.
- Pour projets legacy PHP 7.x, utiliser images compatibles et extensions disponibles pour cette version.
- Ne pas introduire syntaxe, outils ou images qui imposent une montée PHP non validée.
- Respecter le serveur web existant si le projet a déjà Apache, Nginx ou Caddy.
- Vérifier les extensions requises par Symfony et les bundles : `intl`, `pdo_*`, `zip`, `opcache`, `amqp`, `redis`, `gd`, `imagick`, `sodium`, `mbstring`, `xml`, `curl`.

## Services

- **PHP-FPM** : version alignée sur composer.json, Xdebug configurable, OPcache
- **Caddy** (recommandé) ou **Nginx**
- **PostgreSQL/MySQL** selon DATABASE_URL, volume persistant
- **Redis**, **RabbitMQ**, **Mailpit** selon les besoins
- **Node** si package.json présent
- **Worker Messenger** optionnel, seulement si le projet utilise des transports async
- **Search** : Elasticsearch/OpenSearch/Meilisearch seulement si nécessaire
- **Mercure** seulement si package/config explicite

## Environnements

### Dev local

- Volumes bind pour le code source.
- Xdebug désactivable par variable.
- OPcache configuré pour dev ou désactivé selon convention.
- Mailpit/Mailhog pour emails.
- Ports exposés seulement pour les services utiles localement.

### Test

- Base de données isolée ou nom de base distinct.
- Transport Messenger de test ou in-memory si le projet le prévoit.
- Services externes remplacés par fakes/stubs si possible.
- Commandes de test reproductibles via Makefile ou Composer.

### CI

- Configuration minimale et déterministe.
- Pas de dépendance à des ports locaux spécifiques si évitable.
- Cache Composer/Node à prévoir côté CI plutôt que dans le compose local.
- `docker compose config`, build et tests comme étapes séparées.

### Production

- Ne pas générer une configuration production complète par défaut.
- Si demandé explicitement : images immuables, secrets gérés hors git, user non-root, healthchecks, logs stdout/stderr, OPcache activé, pas de Xdebug, pas de volumes source.

## Bonnes pratiques

- `.dockerignore` : exclure vendor/, node_modules/, var/, .git/
- Healthcheck sur chaque service
- Variables d'environnement dans .env
- Makefile avec les commandes courantes
- Images officielles ou images déjà utilisées par le projet
- Multi-stage build si image applicative ou prod demandée
- Cache Composer et Node optimisé
- Installer seulement les extensions nécessaires
- `COPY composer.*` avant `composer install` pour profiter du cache Docker
- Ne pas lancer migrations automatiquement au démarrage du conteneur sauf convention explicite
- Logs vers stdout/stderr
- Noms de services et réseaux lisibles

## Sécurité

- Ne jamais committer de secrets, credentials prod, tokens ou `.env.local`.
- Exposer uniquement les ports nécessaires en local.
- Éviter root au runtime si possible.
- Séparer secrets et variables non sensibles.
- Ne pas utiliser `latest` pour les images critiques si la reproductibilité est importante.
- Volumes avec permissions maîtrisées pour `var/`, cache, logs, uploads.
- Désactiver Xdebug hors dev.
- Ne pas inclure clés privées ou dumps de production dans l'image.

## Performance

- `.dockerignore` complet pour réduire le contexte de build.
- Couches Docker ordonnées pour maximiser le cache.
- Composer install avec flags adaptés à dev ou prod.
- OPcache activé pour prod ou environnement proche prod.
- Xdebug optionnel, jamais activé par défaut en perf/prod.
- Volumes adaptés à l'OS si le projet a des lenteurs de bind mounts.
- Node dependencies gérées avec lockfile et cache si pertinent.

## Doctrine et Messenger

- DB avec healthcheck.
- Volume persistant local pour DB, mais base test isolée.
- Migrations lancées manuellement ou via commande explicite.
- RabbitMQ/Redis seulement si transports async détectés.
- Worker Messenger séparé si nécessaire, avec commande explicite et limites si possible.
- Ne pas lancer un worker infini sans supervision ou sans commande claire.

## Node et assets

- Détecter le package manager : npm, yarn, pnpm.
- Respecter Vite, Webpack Encore, AssetMapper ou absence de build Node.
- Pour dev, proposer watcher si le projet l'utilise.
- Pour prod/build image, compiler les assets dans une étape dédiée si demandé.
- Ne pas ajouter Node si `package.json` absent et assets gérés par AssetMapper sans build.

## Makefile et commandes

Proposer seulement les commandes adaptées :

- `make up`
- `make down`
- `make build`
- `make shell`
- `make composer`
- `make console`
- `make test`
- `make logs`
- `make phpstan`
- `make fixtures`
- `make db-migrate`

Ne pas masquer les commandes Docker réelles : le Makefile doit rester un raccourci lisible.

## Validation

Proposer ou lancer selon le contexte :

- `docker compose config`
- `docker compose build`
- `docker compose up -d`
- `docker compose ps`
- `docker compose exec php composer install`
- `docker compose exec php bin/console about`
- `docker compose exec php bin/console doctrine:schema:validate`
- Tests ciblés si le projet est prêt

Ne pas lancer de migration réelle, purge de DB, suppression de volumes ou téléchargement massif sans confirmation explicite.

## Ne pas faire

- Ne pas lire `.env.local`.
- Ne pas écraser Dockerfile ou compose existant sans préserver les conventions utiles.
- Ne pas exposer DB, RabbitMQ, Redis ou services internes inutilement.
- Ne pas introduire un service non nécessaire.
- Ne pas imposer Caddy, Nginx, Apache, Node ou une version PHP si le projet indique autre chose.
- Ne pas utiliser credentials de production.
- Ne pas lancer automatiquement migrations, fixtures ou workers au démarrage sans demande explicite.
- Ne pas générer une configuration production en prétendant qu'elle est prête pour tous les hébergements.

## Format de sortie

Pour une génération ou modification, fournir :

- Fichiers créés ou modifiés
- Services générés et raison
- Versions PHP, DB, Node et images retenues
- Ports exposés
- Volumes et données persistantes
- Variables d'environnement à définir, sans secrets
- Commandes Makefile ou Docker utiles
- Commandes de validation lancées ou à lancer
- Hypothèses, limites et points de sécurité

Générer tous les fichiers nécessaires (`compose.yaml` ou `docker-compose.yml`, Dockerfile, configs web, `.dockerignore`, Makefile si utile), prêts à utiliser avec `docker compose up` dans le contexte demandé.
