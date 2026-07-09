---
name: performance
description: "Audit et optimisation de performance pour projet PHP/Symfony. Analyse endpoints, commandes, workers, Doctrine, Twig, Serializer, cache, HTTP et jobs batch. Mesure avant d'optimiser, identifie la cause racine et propose des corrections vérifiables."
---

# Audit de performance PHP/Symfony

## Périmètre

Si l'utilisateur précise un endpoint, une commande, un worker Messenger, une requête Doctrine, une page Twig, une API, un diff ou un fichier, analyser uniquement ce périmètre.

Sinon :

- Lire `git status` et le diff si l'audit porte sur les changements récents.
- Identifier les chemins applicatifs sensibles : contrôleurs, repositories, handlers, commands, templates, normalizers.
- Demander le symptôme exact si aucune cible n'est donnée : lenteur, timeout, mémoire, N+1, charge DB, latence réseau.

Ne pas modifier le code pendant un audit sauf demande explicite.

## État des lieux

Inspecter selon le projet :

- `composer.json` : Symfony, Doctrine, Messenger, HttpClient, Cache, Serializer, DebugBundle, Blackfire
- Configuration Doctrine : DBAL, ORM, mappings, second-level cache, logging SQL
- Configuration cache : pools, adapters, tags, TTL, HTTP cache
- Configuration Messenger : transports, retry, workers, batch size éventuel
- Contrôleurs, repositories, services, handlers, templates ou normalizers du périmètre
- Tests, fixtures et commandes disponibles pour reproduire le scénario
- Logs Symfony, profiler ou traces existantes si disponibles

Ne jamais lire `.env.local`. Ne pas afficher de secrets ou données personnelles présents dans les logs.

## Mesurer avant d'optimiser

Avant de proposer un changement, chercher une preuve :

- Temps de réponse, durée de commande ou durée de message
- Nombre de requêtes SQL
- Requête SQL lente et paramètres importants
- Volume de lignes lues, retournées ou hydratées
- Mémoire consommée et pic mémoire
- Nombre d'appels HTTP ou I/O
- Taille du payload API ou du HTML généré
- Volume de données en entrée

Si la mesure directe n'est pas possible, signaler l'hypothèse et proposer la commande ou instrumentation à utiliser.

## Sévérités

- **Bloquant** : timeout, OOM, N+1 massif, flush en boucle critique, chargement de table complète sur chemin fréquent, lock DB probable, appel externe sans timeout sur chemin web critique.
- **Important** : requête lente plausible, index manquant, pagination absente, cache absent sur calcul coûteux, payload excessif, batch non maîtrisé.
- **Suggestion** : optimisation mineure, simplification, micro-optimisation sans impact mesuré majeur.

## Critères d'analyse

### Doctrine - Requêtes (bloquant)

- **N+1** : boucle sur une collection avec accès à une relation lazy-loadée
- **Flush dans une boucle** : `$em->flush()` appelé à chaque itération
- **SELECT *** : requête qui charge toutes les colonnes quand seules quelques-unes sont nécessaires
- **Requête sans index** : WHERE ou ORDER BY sur une colonne non indexée
- Pagination absente sur une liste potentiellement volumineuse
- `findAll()` ou `toArray()` sur une table ou collection non bornée
- Requête dans une boucle ou dans un normalizer appelé en collection
- Jointures non maîtrisées qui multiplient les lignes et explosent l'hydratation
- `COUNT` coûteux recalculé inutilement sur chaque requête
- Filtre ou tri non sargable : fonction sur colonne, wildcard initial, cast, expression non indexable

### Doctrine - Hydratation (important)

- Hydratation complète d'entités quand seuls quelques champs sont lus
- Chargement de collections entières en mémoire
- Entités inutilement trackées par l'UnitOfWork
- Serializer qui déclenche des lazy loads
- Fetch join de plusieurs collections qui produit un produit cartésien
- Absence de `clear()` en traitement batch
- Utilisation d'entités managées pour une lecture pure ou un export massif
- Collection `PersistentCollection` filtrée en PHP au lieu de filtrer en SQL

### Base de données (important)

- Index absent sur colonnes de `WHERE`, `JOIN`, `ORDER BY` ou contrainte unique
- Index inutilisable à cause d'une fonction, conversion ou wildcard initial
- Pagination offset profonde sans stratégie adaptée
- `LIKE '%term%'` sur gros volume sans index spécialisé
- Colonne JSON filtrée sans index adapté
- Verrou long ou transaction trop large
- Requête de suppression ou update massif sans batch ni limite
- Migration d'index risquée sans stratégie online/concurrente selon le SGBD

Pour PostgreSQL, proposer `EXPLAIN (ANALYZE, BUFFERS)` si l'accès DB est disponible.
Pour MySQL/MariaDB, proposer `EXPLAIN` et vérifier index, cardinalité, filesort et temporary tables.

### Cache (important)

- Résultat de requête coûteuse non caché
- Configuration récupérée depuis la DB à chaque requête
- Données statiques recalculées à chaque appel
- Cache sans stratégie d'invalidation
- TTL absent ou trop long pour des données changeantes
- Risque de cache stampede sur calcul coûteux
- Cache par utilisateur qui explose la cardinalité
- Cache HTTP absent pour contenu public stable
- Warmup manquant pour données coûteuses utilisées au démarrage

### HTTP et réseau (important)

- Appels HTTP synchrones dans une requête web
- Absence de timeout sur les appels externes
- Appels HTTP séquentiels qui pourraient être parallèles
- Appels externes dans une boucle
- Retry sans limite ou sans backoff
- Pas de circuit breaker alors que le projet en utilise déjà ailleurs
- Payload externe trop large ou non paginé
- I/O disque ou stockage distant sur chemin web critique

### Templates Twig (suggestion)

- Requêtes Doctrine dans les templates (lazy loading)
- Boucles avec filtre coûteux sur de grandes collections
- Appels de méthodes coûteuses dans une boucle Twig
- Rendu de composants ou partials déclenchant chacun des requêtes
- Absence de pagination ou état vide coûteux à calculer
- Usage de `|length` sur collection lazy non initialisée

### Serializer et API (important)

- Serializer groups trop larges qui chargent trop de relations
- Normalizer qui appelle repository ou service externe par item
- Circular references contournées en exposant trop de données
- Payload non paginé ou trop volumineux
- Champs calculés coûteux sur chaque élément d'une collection
- DTO output absent alors que le contrat public pourrait limiter les données chargées

### Messenger et batch (important)

- Message ou commande qui charge trop d'entités en mémoire
- Absence de batch size, `flush()`/`clear()` périodique ou reprise possible
- Worker qui fuit en mémoire sur traitement long
- Retry sur opération non idempotente ou très coûteuse
- Traitement long exécuté en requête web au lieu d'être asynchrone
- Ack/retry/prefetch non adaptés au coût du message

### PHP runtime et algorithmique (important)

- Algorithme quadratique sur collections volumineuses
- `array_map`, `array_filter`, `usort` ou `in_array` répétés sur gros volumes sans indexation préalable
- Regex coûteuse ou exécutée en boucle
- Lecture complète de gros fichiers en mémoire au lieu de streaming
- Génération d'exports sans pagination ni streaming
- Autoload ou reflection répétés dans une boucle
- Logs volumineux sur chemin chaud

## Corrections attendues

- Remplacer N+1 par requête adaptée, fetch join maîtrisé, eager loading ciblé ou DTO projection.
- Ajouter pagination, limite ou streaming pour listes et exports.
- Utiliser scalar results, partial objects ou DTO quand l'entité complète n'est pas nécessaire.
- Ajouter index ou adapter la requête pour utiliser les index existants.
- Déplacer un traitement long vers Messenger si la réponse web ne doit pas attendre.
- Ajouter cache avec clé, TTL et invalidation explicites.
- Ajouter timeouts, retry borné ou parallélisation pour les appels HTTP.
- Pour batch Doctrine, utiliser chunks, `flush()` et `clear()` périodiques.

## Commandes utiles

Adapter au projet :

- Symfony Profiler pour nombre de requêtes, temps SQL, mémoire, timeline
- `bin/console about`
- `bin/console debug:container`
- `bin/console doctrine:schema:validate`
- `bin/console doctrine:migrations:status`
- Tests ciblés ou scénario de reproduction
- PHPStan si l'optimisation touche les types ou contrats
- Blackfire si présent dans le projet
- `EXPLAIN` / `EXPLAIN ANALYZE` à proposer si accès DB disponible

Ne pas lancer de charge lourde, benchmark destructif, purge cache globale ou migration sans confirmation explicite.

## Ne pas faire

- Ne pas optimiser sans preuve ou hypothèse clairement annoncée.
- Ne pas ajouter du cache sans invalidation ou TTL.
- Ne pas masquer une requête lente par un cache si la requête reste critique ailleurs.
- Ne pas charger toute une table ou collection en mémoire.
- Ne pas remplacer une requête simple par une abstraction complexe sans gain clair.
- Ne pas sacrifier les invariants métier pour optimiser.
- Ne pas faire de micro-optimisations PHP quand le coût principal est SQL, réseau ou I/O.
- Ne pas supprimer tests, logs utiles ou validations pour gagner artificiellement du temps.

## Format de sortie

Présenter les findings triés par sévérité décroissante.

Pour chaque problème :

- Fichier et ligne
- Sévérité : bloquant, important ou suggestion
- Type : Doctrine, DB, cache, HTTP, Twig, Serializer, Messenger, PHP runtime
- Preuve ou mesure disponible
- Impact concret : temps, mémoire, requêtes SQL, volume, risque timeout
- Cause racine
- Correction proposée
- Risque ou compromis de la correction
- Validation conseillée : profiler, test, EXPLAIN, benchmark ciblé
- Gain attendu si estimable

Si aucune mesure n'a été possible, le dire clairement et séparer les hypothèses des problèmes prouvés.
