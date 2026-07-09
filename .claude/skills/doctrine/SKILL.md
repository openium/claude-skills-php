---
name: doctrine
description: "Audite et corrige l'usage Doctrine ORM/DBAL dans un projet PHP/Symfony. Analyse entités, mappings, relations, repositories, QueryBuilder, transactions, UnitOfWork, cascades, index, performances et cohérence avec migrations sans se limiter aux migrations."
---

# Audit Doctrine ORM/DBAL

## Périmètre

Déterminer si l'utilisateur demande :

- Audit d'une entité, relation, repository, requête ou service Doctrine.
- Correction d'une erreur Doctrine : mapping, query, transaction, flush, cascade, proxy.
- Conception d'un mapping ou d'une relation.
- Optimisation d'une requête ou d'un chargement.
- Analyse d'un diff qui touche Doctrine.

Si aucun périmètre n'est donné :

- Lire `git status`.
- Analyser les fichiers Doctrine modifiés : entités, repositories, migrations, services qui utilisent l'EntityManager.
- À défaut, demander le périmètre.

Ne pas modifier une migration déjà exécutée sans validation explicite.

## État des lieux

Inspecter selon le sujet :

- `composer.json` : Doctrine ORM, DBAL, migrations, extensions, version Symfony
- `config/packages/doctrine.yaml`, mappings, types custom, naming strategy
- Entités, repositories, services, handlers, controllers ou normalizers concernés
- Migrations récentes si le schéma change
- Tests et fixtures qui couvrent le comportement
- Version PHP/Symfony pour attributs, annotations, enums ou readonly

Ne jamais lire `.env.local`.

## Sévérités

- **Bloquant** : mapping invalide, perte de données, relation incohérente, transaction absente sur écritures critiques, cascade dangereuse, requête incorrecte, N+1 massif.
- **Important** : index manquant, hydratation excessive, flush en boucle, repository qui porte de la logique métier, migration absente, nullabilité incohérente.
- **Suggestion** : nommage, typage PHPDoc/generics, extraction de requête, simplification ou alignement conventionnel.

## Mapping et entités

- Vérifier cohérence entre propriétés PHP, mapping Doctrine, contraintes DB et validation Symfony.
- Respecter la convention du projet : attributs, annotations, XML ou YAML.
- Pour PHP 7.x, ne pas introduire attributs, enums ou readonly.
- Initialiser les collections dans le constructeur.
- Préférer `DateTimeImmutable` si le projet l'utilise déjà.
- Vérifier types `decimal`, `json`, `datetime`, `uuid`, enums et value objects.
- Ne pas exposer des setters aveugles si l'entité porte des invariants métier.
- Éviter la logique applicative, appels réseau ou accès service dans une entité.

## Relations

- Vérifier owning side / inverse side.
- Maintenir les deux côtés d'une relation bidirectionnelle dans les méthodes `add/remove`.
- Définir `nullable`, `onDelete`, cascade et `orphanRemoval` explicitement selon le métier.
- Ne pas utiliser `cascade={"remove"}` ou `orphanRemoval=true` sans comprendre l'impact données.
- Ajouter un index sur les colonnes FK utilisées en recherche, jointure ou tri.
- Pour ManyToMany, vérifier table de jointure, contraintes uniques et cas de suppression.
- Pour relations volumineuses, éviter de charger toute la collection pour compter, filtrer ou paginer.

## Repositories et requêtes

- Les repositories contiennent des requêtes, pas la logique métier.
- Binder tous les paramètres DQL/SQL.
- Éviter les concaténations DQL/SQL avec données utilisateur.
- Retourner un type clair : entité, liste typée, scalar result, DTO projection ou paginator.
- Gérer explicitement `null` après `find()` / `findOneBy()`.
- Éviter `findAll()` sur tables volumineuses.
- Utiliser pagination pour les listes.
- Vérifier les N+1 et lazy loads déclenchés par Twig, Serializer ou normalizers.
- Pour lecture pure, envisager scalar results, DTO projection ou query dédiée.

## Transactions et écritures

- Utiliser une transaction explicite pour plusieurs écritures cohérentes.
- Ne pas faire `flush()` dans une boucle sauf besoin prouvé.
- Pour batch, utiliser chunks, `flush()` et `clear()` périodiques.
- Ne pas mélanger transaction longue et appels HTTP/I/O externes.
- Vérifier idempotence si l'écriture est appelée depuis Messenger ou un retry.
- Ne pas avaler une exception Doctrine qui laisse l'EntityManager fermé.

## Migrations et schéma

- Toute modification d'entité persistée doit avoir une migration correspondante.
- Vérifier nullabilité, valeurs par défaut, contraintes uniques, index et clés étrangères.
- Éviter les migrations destructives sans stratégie de données.
- Ne pas supposer que la table est vide.
- Pour tables volumineuses, signaler les risques de lock et recommander une stratégie zero-downtime si nécessaire.

## Performance

- Identifier N+1, hydratation excessive, jointures multiples de collections, pagination absente.
- Vérifier index pour `WHERE`, `JOIN`, `ORDER BY`, contraintes uniques.
- Proposer `EXPLAIN` / `EXPLAIN ANALYZE` si accès DB disponible.
- Ne pas optimiser sans preuve ou hypothèse explicite.
- Ne pas cacher une requête lente sans stratégie d'invalidation.

## Tests et validation

Prévoir selon le risque :

- Test unitaire d'invariant d'entité ou value object.
- Test d'intégration repository avec vraie base.
- Test fonctionnel si Twig/API/Serializer déclenche le chargement.
- Test de transaction ou idempotence pour écriture critique.
- Fixtures minimales cohérentes avec les contraintes.

Commandes utiles :

- `bin/console doctrine:schema:validate`
- `bin/console doctrine:migrations:diff`
- `bin/console doctrine:migrations:migrate --dry-run`
- `vendor/bin/phpunit --filter NomDuTest`
- PHPStan si typage repositories/entities présent

Ne pas lancer de migration réelle, purge ou commande destructive sans confirmation explicite.

## Ne pas faire

- Ne pas injecter l'EntityManager partout si un repository ou service applicatif suffit.
- Ne pas hydrater directement une entité depuis une requête externe sensible.
- Ne pas ignorer les problèmes de nullabilité entre PHP, DB et validation.
- Ne pas supprimer cascade, FK ou contrainte unique sans vérifier l'impact données.
- Ne pas mettre de logique métier lourde dans repository ou controller.
- Ne pas modifier `.env.local`.

## Format de sortie

Présenter les findings triés par sévérité décroissante.

Pour chaque finding :

- Fichier et ligne
- Sévérité : bloquant, important ou suggestion
- Catégorie : mapping, relation, repository, transaction, migration, performance, sécurité
- Preuve vérifiée
- Impact concret
- Correction proposée
- Test ou commande de validation conseillé

Si aucun problème n'est trouvé, mentionner les limites : DB non accessible, migrations non lancées, profiler absent, tests non exécutés.
