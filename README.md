# Claude Skills PHP & Symfony

Skills Claude Code pour les développeurs PHP et Symfony. Chaque skill est un prompt spécialisé, versionné dans le projet, que l'équipe peut invoquer avec `/nom-du-skill`.

## Installation

Copiez les skills dans le projet cible :

```bash
# Un skill spécifique
cp -r .claude/skills/review /chemin/du/projet/.claude/skills/

# Tous les skills
cp -r .claude/skills /chemin/du/projet/.claude/
```

Commitez le dossier `.claude/skills/` dans le projet cible pour le rendre disponible à toute l'équipe.

## Template CLAUDE.md

Un modèle de fichier `CLAUDE.md` est disponible dans [`templates/CLAUDE.md`](templates/CLAUDE.md).

Il peut être copié à la racine d'un projet PHP/Symfony pour documenter le contexte projet, la stack, les commandes utiles, les règles de sécurité et le workflow attendu avec Claude Code.

Prompt recommandé pour générer un `CLAUDE.md` adapté à un projet :

```text
À partir du template `templates/CLAUDE.md`, génère un fichier `CLAUDE.md` à la racine de ce projet.

Avant d'écrire le fichier :
- analyse la structure du projet ;
- lis les fichiers de configuration utiles : `composer.json`, `composer.lock`, `symfony.lock`, `phpunit.xml*`, `phpstan*`, `.php-cs-fixer*`, `docker-compose*`, `Dockerfile*`, `Jenkinsfile`, `.github/workflows/*` s'ils existent ;
- identifie les versions PHP, Symfony, Doctrine et les outils réellement utilisés ;
- repère les commandes disponibles pour installer, tester, analyser et lancer le projet ;
- déduis les conventions existantes sans inventer de règles non visibles dans le dépôt.

Ensuite :
- remplace les placeholders du template par les informations du projet ;
- conserve les sections utiles ;
- supprime les exemples ou options qui ne s'appliquent pas ;
- garde les règles de sécurité, Git et données sensibles ;
- indique clairement les informations que tu n'as pas pu déterminer.

Avant modification, liste les changements prévus et attends ma validation.
```

## Skills Disponibles

### Qualité De Code

| Skill | Description |
|-------|-------------|
| [`/review`](.claude/skills/review/SKILL.md) | Revue de code PHP/Symfony : bugs, sécurité, architecture, tests, Doctrine, migrations. |
| [`/refactor`](.claude/skills/refactor/SKILL.md) | Refactoring sûr sans changer le comportement observable. |
| [`/phpstan`](.claude/skills/phpstan/SKILL.md) | Correction des erreurs PHPStan sans ignore, baseline ou baisse de niveau. |
| [`/deprecations`](.claude/skills/deprecations/SKILL.md) | Détection et correction des deprecations PHP/Symfony, y compris projets legacy. |
| [`/doctrine`](.claude/skills/doctrine/SKILL.md) | Audit Doctrine ORM/DBAL : mappings, relations, repositories, transactions, performances. |

### Tests Et Données

| Skill | Description |
|-------|-------------|
| [`/test`](.claude/skills/test/SKILL.md) | Génération de tests PHPUnit unitaires, fonctionnels ou d'intégration. |
| [`/fixtures`](.claude/skills/fixtures/SKILL.md) | Génération de fixtures Doctrine/Alice/Foundry réalistes et déterministes. |

### Sécurité, Performance Et Observabilité

| Skill | Description |
|-------|-------------|
| [`/security`](.claude/skills/security/SKILL.md) | Audit sécurité basé sur l'OWASP Top 10 pour PHP/Symfony. |
| [`/performance`](.claude/skills/performance/SKILL.md) | Audit performance : N+1, cache, Doctrine, Twig, HTTP, Messenger, batch. |
| [`/observability`](.claude/skills/observability/SKILL.md) | Logs, traces, métriques, alerting, Sentry/APM et protection des données sensibles. |

### Génération De Code

| Skill | Description |
|-------|-------------|
| [`/crud`](.claude/skills/crud/SKILL.md) | CRUD Symfony adapté au projet : Twig, API JSON, API Platform, DTOs, tests. |
| [`/dto`](.claude/skills/dto/SKILL.md) | DTOs readonly typés, validation, mapping entité/DTO via handler ou factory. |
| [`/command`](.claude/skills/command/SKILL.md) | Commandes Symfony Console avec arguments, options, validation, progress bar, dry-run. |
| [`/api-doc`](.claude/skills/api-doc/SKILL.md) | Documentation OpenAPI/NelmioApiDocBundle pour endpoints Symfony. |

### Infrastructure Et CI

| Skill | Description |
|-------|-------------|
| [`/docker`](.claude/skills/docker/SKILL.md) | Environnement Docker PHP/Symfony : PHP-FPM, web, DB, cache, queue, mail, assets. |

### Symfony

| Skill | Description |
|-------|-------------|
| [`/migration`](.claude/skills/migration/SKILL.md) | Audit de migrations Doctrine : perte de données, up/down, verrous, zero-downtime. |
| [`/messenger`](.claude/skills/messenger/SKILL.md) | Audit Symfony Messenger : transports, routing, retry, handlers, workers. |
| [`/twig`](.claude/skills/twig/SKILL.md) | Audit Twig : sécurité, accessibilité, performance, formulaires, i18n, UX. |

### Planification Et Onboarding

| Skill | Description |
|-------|-------------|
| [`/plan`](.claude/skills/plan/SKILL.md) | Plan d'implémentation avant code : étapes, fichiers, tests, risques. |
| [`/debug`](.claude/skills/debug/SKILL.md) | Diagnostic d'erreur, stacktrace, test en échec ou comportement inattendu. |
| [`/upgrade`](.claude/skills/upgrade/SKILL.md) | Assistance upgrade PHP/Symfony/dépendances Composer. |
| [`/onboard`](.claude/skills/onboard/SKILL.md) | Guide d'onboarding technique pour un projet PHP/Symfony. |

## Exemples D'Invocation

| Besoin | Skill | Exemple |
|--------|-------|---------|
| Relire un diff avant commit | `/review` | `/review analyse le diff staged avant commit` |
| Simplifier une classe sans changer le comportement | `/refactor` | `/refactor src/Service/InvoiceCalculator.php` |
| Corriger PHPStan | `/phpstan` | `/phpstan corrige cette erreur PHPStan dans UserRepository` |
| Préparer un upgrade | `/deprecations` | `/deprecations scanne les deprecations avant Symfony 6.4` |
| Auditer Doctrine hors migration | `/doctrine` | `/doctrine vérifie le mapping et les relations de Order` |
| Générer des tests | `/test` | `/test génère les tests pour src/Service/PriceCalculator.php` |
| Créer des fixtures | `/fixtures` | `/fixtures ajoute un scénario de commande payée pour les tests fonctionnels` |
| Auditer la sécurité | `/security` | `/security audite les contrôleurs modifiés` |
| Diagnostiquer une lenteur | `/performance` | `/performance analyse pourquoi cet endpoint fait trop de requêtes SQL` |
| Améliorer les logs | `/observability` | `/observability ajoute les logs utiles autour du handler de paiement` |
| Générer un CRUD | `/crud` | `/crud crée un CRUD Twig pour Product avec tests et migration` |
| Créer des DTOs | `/dto` | `/dto crée les DTO input/output pour l'endpoint de création utilisateur` |
| Créer une commande | `/command` | `/command génère une commande d'import CSV avec dry-run` |
| Documenter une API | `/api-doc` | `/api-doc documente l'endpoint POST /api/orders` |
| Ajouter Docker | `/docker` | `/docker génère un environnement local avec PostgreSQL, Redis et Mailpit` |
| Auditer une migration | `/migration` | `/migration vérifie la dernière migration Doctrine pour le zero-downtime` |
| Auditer Messenger | `/messenger` | `/messenger vérifie routing, retry et failed transport` |
| Auditer Twig | `/twig` | `/twig audite templates/order/show.html.twig` |
| Planifier une feature | `/plan` | `/plan prépare l'implémentation d'un export CSV de factures` |
| Déboguer une erreur | `/debug` | `/debug analyse cette stacktrace Symfony` |
| Monter de version | `/upgrade` | `/upgrade prépare le passage de Symfony 5.4 à 6.4` |
| Documenter un projet | `/onboard` | `/onboard génère un guide pour un nouveau développeur` |

## Convention De Structure

Chaque skill est un dossier dans `.claude/skills/<nom>/` avec un fichier obligatoire `SKILL.md`.

```text
.claude/
└── skills/
    └── nom-du-skill/
        └── SKILL.md
```

Le nom du dossier et le champ `name` doivent être identiques.

```yaml
---
name: nom-du-skill
description: "Description courte et précise du moment où utiliser ce skill."
---
```

Règles recommandées :

- Utiliser un nom court, en minuscules, avec tirets si nécessaire.
- Rédiger le skill en français technique, avec les termes natifs anglais quand ils viennent de l'écosystème : `DTO`, `handler`, `dry-run`, `readonly`, `failure transport`.
- Commencer par un **Périmètre** clair.
- Ajouter un **État des lieux** quand le skill doit inspecter un projet.
- Décrire les **Sévérités** pour les skills d'audit.
- Lister les commandes utiles, mais signaler les commandes destructives à éviter sans confirmation.
- Inclure une section **Ne pas faire** pour les garde-fous.
- Terminer par un **Format de sortie** exploitable.
- Ne jamais demander de lire `.env.local`.
- Ne jamais masquer secrets, tokens, cookies, DSN, mots de passe ou données personnelles sensibles.
- Respecter les projets legacy : ne pas supposer PHP 8+, attributs, enums, readonly ou Symfony récent sans preuve.

## Contribution

Pour ajouter ou modifier un skill :

1. Créer ou modifier `.claude/skills/<nom>/SKILL.md`.
2. Vérifier que le frontmatter contient `name` et `description`.
3. Vérifier que `name` correspond au dossier.
4. Ajouter le skill dans la liste du README.
5. Ajouter au moins un exemple d'invocation.
6. Relire les garde-fous : secrets, `.env.local`, commandes destructives, compatibilité legacy.

Un commit doit idéalement concerner un seul skill, sauf ajout coordonné comme un nouveau skill avec README.

## Licence

MIT

---

Maintenu par [Efficience IT](https://www.itefficience.com) - Expertise PHP & Symfony
