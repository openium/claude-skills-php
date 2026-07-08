---
name: upgrade
description: "Assiste les upgrades PHP/Symfony et dépendances Composer. Analyse les versions actuelles, deprecations, breaking changes et conflits, puis produit ou applique un plan de migration par étapes. Ne monte jamais PHP sans validation explicite."
---

# Montée de version PHP/Symfony

## Périmètre

Déterminer si l'utilisateur demande :

- Un diagnostic ou plan d'upgrade : ne pas modifier le code.
- L'application d'une migration : procéder par étapes vérifiables.
- Un périmètre ciblé : PHP, Symfony, dépendance Composer, bundle, config ou code applicatif.

Si la version cible n'est pas donnée, analyser l'état actuel et demander la cible avant de modifier.

## État des lieux

Inspecter selon le projet :

- `composer.json`, `composer.lock`, `symfony.lock`
- Version PHP requise, `config.platform.php`, extensions PHP
- Packages `symfony/*`, bundles tiers, Doctrine, PHPUnit, PHPStan, Rector
- `config/bundles.php`, `config/packages/`, `config/routes/`
- `phpunit.xml*`, `phpstan.neon*`, `rector.php`
- Dockerfile, `docker-compose*.yml`, CI GitHub/GitLab, scripts Composer

Ne jamais lire ni modifier `.env.local`.

## Stratégie

- Préférer une migration incrémentale : dernière minor stable avant changement de major.
- Corriger les deprecations avant un changement de major Symfony.
- Garder les packages `symfony/*` cohérents sur la même version cible.
- Traiter d'abord les conflits Composer et bundles tiers, puis le code applicatif.
- Séparer upgrade, refactor et changement fonctionnel.
- Ne pas modifier `composer.lock` à la main.

## Version PHP

Ne jamais monter la version PHP sans validation explicite.

Si Symfony ou une dépendance impose une version PHP supérieure :

- Identifier la version PHP minimale requise.
- Vérifier où PHP est fixé : `composer.json`, `config.platform.php`, Docker, CI, hébergement.
- Expliquer l'impact sur l'environnement d'exécution.
- Demander confirmation avant toute modification liée à PHP.

Ne pas introduire enums, readonly, `#[\Override]` ou autre syntaxe récente uniquement parce que la version cible les supporte.

## Symfony

À vérifier :

- Annotations remplacées par des attributs PHP 8+
- Classes, méthodes, options et services marqués `@deprecated`
- Configurations YAML dépréciées ou déplacées
- Changements de recipes Flex et fichiers générés
- Changements de signature, événements, security voters, authenticator, serializer, forms, validation
- Compatibilité des bundles tiers avec la version cible

Comparer les recipes Flex avec prudence et conserver les adaptations projet.

## Composer et dépendances

- Utiliser `composer why-not` pour expliquer un blocage de version.
- Utiliser `composer update --dry-run` avant une mise à jour risquée.
- Ne pas forcer une contrainte incompatible sans comprendre le package bloquant.
- Ne pas supprimer un bundle sans vérifier ses usages dans code, config et templates.
- Vérifier `composer audit` si disponible.

## Rector

Si Rector est déjà présent (`rector.php`, dépendance `rector/rector` ou script Composer dédié), il peut être utilisé pour les migrations mécaniques.

Règles :

- Ne pas installer Rector sans demande explicite.
- Lancer d'abord Rector en dry-run si possible.
- Limiter le périmètre si l'utilisateur a demandé une migration ciblée.
- Relire le diff généré avant de considérer la migration comme valide.
- Refuser un changement Rector qui modifie le comportement métier.
- Lancer les tests après application.

## Commandes utiles

Lancer seulement les commandes adaptées au projet :

- `composer outdated`
- `composer why-not vendor/package version`
- `composer update --dry-run`
- `composer audit`
- `bin/console debug:container --deprecations`
- `vendor/bin/rector process --dry-run` si Rector est présent
- Tests ciblés, suite complète, PHPStan si présents

Rapporter les commandes lancées, leur résultat et les blocages.

## Ne pas faire

- Ne pas faire un saut majeur non demandé.
- Ne pas masquer ou ignorer les deprecations.
- Ne pas mélanger upgrade majeure et refactor large.
- Ne pas changer le comportement métier pour satisfaire une upgrade.
- Ne pas supprimer un package ou bundle sans preuve qu'il n'est plus utilisé.
- Ne pas modifier les fichiers d'environnement locaux ou secrets.

## Format de sortie

Pour un diagnostic :

- État actuel : PHP, Symfony, dépendances clés
- Cible demandée ou recommandée
- Blockers Composer
- Deprecations et breaking changes identifiés
- Fichiers ou packages impactés
- Plan d'exécution ordonné

Après application :

- Fichiers modifiés
- Dépendances mises à jour
- Corrections appliquées
- Commandes lancées
- Tests ou vérifications
- Risques ou étapes restantes
