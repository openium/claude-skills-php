---
name: migration
description: "Analyse les migrations Doctrine d'un projet PHP/Symfony. Détecte pertes de données, incohérences up/down, verrous longs, opérations non zero-downtime, index manquants et migrations incompatibles avec PostgreSQL, MySQL ou MariaDB."
---

# Vérification des migrations Doctrine

## Périmètre

Si l'utilisateur précise une migration, un dossier, un diff, une branche ou une PR, analyser uniquement ce périmètre.

Sinon :

- Lire `git status`.
- Analyser les migrations modifiées dans `migrations/` ou `src/Migrations/`.
- À défaut, analyser la migration la plus récente.
- Si aucune migration n'existe ou si le périmètre est ambigu, demander le fichier à vérifier.

Ne pas modifier une migration déjà exécutée en production sans validation explicite. Dans ce cas, recommander une nouvelle migration corrective.

## État des lieux

Avant de conclure, inspecter selon le projet :

- Version Doctrine Migrations et configuration (`doctrine_migrations.yaml`, `migrations_paths`)
- SGBD cible : PostgreSQL, MySQL, MariaDB ou autre
- Entités Doctrine liées au changement
- Repositories, requêtes, contrôleurs, DTO, serializers ou templates qui utilisent les colonnes touchées
- Contraintes existantes : clés étrangères, uniques, index, `NOT NULL`, valeurs par défaut
- Données potentiellement volumineuses ou tables critiques

Ne jamais lire `.env.local`. Si le SGBD n'est pas identifiable, signaler l'hypothèse utilisée.

## Processus

1. Identifier les méthodes `up()` et `down()`, les requêtes SQL et les appels Doctrine DBAL.
2. Classer chaque opération : création, suppression, renommage, modification de type, contrainte, index, backfill, update massif.
3. Vérifier la perte de données, la réversibilité, les verrous, la durée probable et la compatibilité applicative.
4. Contrôler la cohérence avec les entités et le code encore susceptible de tourner pendant le déploiement.
5. Proposer une correction ou une stratégie en plusieurs migrations si nécessaire.

## Sévérités

- **Bloquant** : perte de données possible, migration non réversible alors qu'elle devrait l'être, rupture applicative probable, SQL invalide pour le SGBD cible, migration déjà exécutée modifiée.
- **Important** : verrou long probable, opération non zero-downtime, index manquant sur table volumineuse, backfill massif non borné, `down()` incomplet mais acceptable seulement avec justification.
- **Suggestion** : lisibilité, regroupement d'opérations, commentaire utile, amélioration de nommage ou de robustesse sans risque immédiat.

## Critères d'analyse

### Perte de données (bloquant)

- `DROP TABLE` sans vérification que la table est vide ou que les données ont été migrées
- `DROP COLUMN` sans migration de données préalable
- `TRUNCATE` dans une migration
- Modification de type de colonne qui tronque des données (VARCHAR(255) vers VARCHAR(50))
- Suppression d'une contrainte UNIQUE qui pourrait créer des doublons
- Suppression d'une clé étrangère sans vérifier les écritures applicatives concurrentes
- Renommage de table ou colonne sans période de compatibilité
- `UPDATE` qui écrase une valeur existante sans condition de sécurité
- `DELETE` sans clause `WHERE` ou sans sauvegarde/migration de données

### Cohérence Doctrine et applicative (bloquant)

- Migration absente alors qu'une entité modifie le schéma
- Migration présente mais entité non cohérente avec le schéma final
- Colonne supprimée encore référencée dans le code, une requête DQL/SQL, un serializer, un formulaire ou un template
- Contrainte `NOT NULL` ajoutée alors que le code peut encore écrire `NULL`
- Nouvelle relation Doctrine sans clé étrangère, index ou stratégie de cascade cohérente
- Changement de nom de table, colonne ou index incompatible avec les conventions Doctrine du projet

### Cohérence up/down (bloquant)

- Méthode `down()` absente
- `down()` qui ne fait pas l'inverse exact de `up()`
- `down()` qui tente de recréer une colonne supprimée sans restaurer les données
- `down()` qui supprime des données créées avant la migration
- Ordre de suppression incohérent avec les clés étrangères
- Types, valeurs par défaut, nullabilité, contraintes ou index non restaurés

Si l'opération est volontairement non réversible, le `down()` doit contenir une exception explicite ou un commentaire clair, et le risque doit être signalé.

### Performance (important)

- `ALTER TABLE` sur une table volumineuse (> 1M lignes) sans estimation de temps
- Ajout d'index sur une grosse table sans `CONCURRENTLY` (PostgreSQL) ou équivalent
- Plusieurs `ALTER TABLE` sur la même table au lieu d'un seul
- `UPDATE` massif sans clause `WHERE` limitante
- Backfill sans batch, limite, reprise possible ou condition idempotente
- Ajout d'une colonne `NOT NULL DEFAULT` susceptible de réécrire toute la table selon le SGBD/version
- Création de clé étrangère ou contrainte unique sans index préalable adapté
- Validation de contrainte sur table volumineuse sans phase `NOT VALID` puis `VALIDATE CONSTRAINT` quand PostgreSQL le permet
- Transaction longue qui combine DDL, backfill et contraintes sur une table critique

### Zero-downtime (important)

- Renommage de colonne (casse l'ancien code encore en prod)
- Suppression de colonne encore référencée dans le code
- Ajout de colonne NOT NULL sans valeur par défaut
- Modification de type de colonne
- Renommage de table
- Changement d'enum, check constraint ou valeur autorisée avant le déploiement du code compatible
- Ajout de contrainte unique sans nettoyage préalable des doublons
- Migration qui suppose que tout le trafic applicatif est arrêté
- Dépendance au déploiement exact code + migration dans le même instant

## Points spécifiques par SGBD

### PostgreSQL

- Utiliser `CREATE INDEX CONCURRENTLY` et `DROP INDEX CONCURRENTLY` pour les gros index quand possible.
- Désactiver la transaction Doctrine pour les index concurrents avec `public function isTransactional(): bool { return false; }`.
- Préférer `ADD CONSTRAINT ... NOT VALID` puis `VALIDATE CONSTRAINT` pour réduire les verrous.
- Vérifier les locks des `ALTER TYPE`, `ALTER TABLE ... SET NOT NULL`, `DROP COLUMN`, `RENAME COLUMN`.
- Éviter les `USING` casts destructifs sans contrôle des données.

### MySQL et MariaDB

- Vérifier l'impact de `ALTER TABLE` selon le moteur, la version, `ALGORITHM=INPLACE/INSTANT` et `LOCK=NONE`.
- Anticiper la copie de table sur les modifications de type, d'index ou de nullabilité.
- Vérifier les longueurs d'index avec `utf8mb4`.
- Préserver les options exactes de colonne dans `down()` : type, collation, charset, default, nullable, unsigned.
- Pour les tables volumineuses, recommander une stratégie online schema change si le projet en dispose.

## Pattern zero-downtime recommandé

Pour les opérations destructives ou incompatibles avec l'ancien code, recommander un déploiement en plusieurs étapes :

1. Migration A : ajouter la nouvelle colonne/table/contrainte permissive, sans supprimer l'ancien schéma.
2. Déploiement A : code compatible qui lit l'ancien et le nouveau, ou écrit dans les deux.
3. Backfill : migrer les données par batch, de manière idempotente et reprise possible.
4. Déploiement B : code qui ne dépend plus de l'ancien schéma.
5. Migration B : supprimer l'ancien schéma ou rendre la contrainte stricte.

## Requêtes et données

- Les requêtes de backfill doivent être idempotentes.
- Les migrations qui modifient des données métier doivent être explicitement justifiées.
- Les `UPDATE` et `DELETE` doivent avoir une clause `WHERE` précise ou une justification.
- Les valeurs par défaut doivent être compatibles avec les règles métier et la validation Symfony.
- Les migrations ne doivent pas dépendre de services applicatifs instables, du container Symfony ou de données externes.

## Commandes utiles

Adapter les commandes au projet, sans installer de dépendance sans demande explicite :

- `bin/console doctrine:migrations:status`
- `bin/console doctrine:migrations:diff`
- `bin/console doctrine:migrations:migrate --dry-run`
- `bin/console doctrine:schema:validate`
- `bin/console doctrine:schema:update --dump-sql`
- Tests ciblés des repositories, services ou endpoints touchés

Rapporter les commandes lancées, leur résultat et les limites éventuelles.

## Ne pas faire

- Ne pas considérer une migration comme sûre uniquement parce qu'elle a été générée par Doctrine.
- Ne pas ignorer les `down()` incohérents.
- Ne pas proposer de supprimer des données sans stratégie de sauvegarde ou migration préalable.
- Ne pas mélanger refactor applicatif large et correction de migration si ce n'est pas nécessaire.
- Ne pas modifier une migration historisée ou déjà exécutée sans confirmation.
- Ne pas masquer un risque zero-downtime par un simple commentaire.

## Format de sortie

Présenter les findings en premier, triés par sévérité décroissante.

Pour chaque finding :

- Fichier et ligne
- Sévérité : bloquant, important ou suggestion
- Opération SQL concernée
- Preuve vérifiée dans la migration ou le code
- Impact concret : données, disponibilité, rollback ou compatibilité applicative
- Correction proposée
- Stratégie zero-downtime si applicable
- Test ou commande de validation conseillé

Si aucun problème n'est trouvé, le dire clairement et mentionner le SGBD supposé, les commandes non lancées et les limites de l'analyse.
