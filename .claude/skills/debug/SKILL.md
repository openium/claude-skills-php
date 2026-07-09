---
name: debug
description: "Aide au débogage pour projet PHP/Symfony. Analyse erreur, stacktrace, logs, test en échec ou comportement inattendu. Reproduit le problème si possible, isole la cause racine et propose un fix vérifiable sans masquer l'erreur."
---

# Débogage PHP/Symfony

## Périmètre

Si l'utilisateur fournit une erreur, une stacktrace, un log, un test en échec, une URL, une commande ou un fichier, commencer par ce signal.

Sinon :

- Lire `git status` pour repérer les changements récents.
- Chercher les logs Symfony disponibles dans `var/log/`.
- Identifier les scripts utiles dans `composer.json`.
- Demander le symptôme exact si aucun signal exploitable n'est présent.

Ne jamais lire ni afficher `.env.local`. Ne pas afficher de secret, token, cookie, mot de passe ou donnée personnelle sensible présent dans les logs.

## Objectif

Trouver la cause racine minimale et proposer une correction ciblée.

Ne pas se contenter de :

- masquer l'exception avec un `try/catch` trop large ;
- ignorer une erreur de test ;
- vider le cache comme solution permanente ;
- modifier une configuration globale sans preuve ;
- changer le comportement métier pour faire disparaître le symptôme.

## Processus

1. Capturer le symptôme exact : message, code HTTP, commande, entrée utilisateur, environnement, stacktrace, test ou log.
2. Reproduire le problème avec la plus petite commande ou action possible.
3. Lire la première frame applicative de la stacktrace, puis les appels voisins nécessaires.
4. Formuler une hypothèse vérifiable et chercher la preuve dans le code, la configuration, la base ou les logs.
5. Corriger la cause racine avec le plus petit changement cohérent avec le projet.
6. Relancer la commande, le test ou le scénario qui reproduisait le bug.
7. Ajouter ou proposer un test de non-régression quand le comportement métier est touché.

Si la reproduction est impossible, expliquer ce qui manque et fournir les commandes ou données à collecter.

## Analyse par type d'erreur

### Exceptions Symfony

- Remonter la stacktrace jusqu'à la première frame applicative.
- Distinguer exception métier attendue, mauvaise configuration, service absent, erreur de sérialisation et bug runtime.
- Vérifier les fichiers de configuration concernés dans `config/packages/`, `config/routes/`, `config/services.yaml` et `config/bundles.php`.
- Pour une erreur de container, comparer l'argument attendu, le service injecté, l'autowiring et les aliases.
- Pour une erreur de routing, vérifier attributs, YAML/XML, préfixes, méthodes HTTP, host, locale et ordre des routes.

### Erreurs PHP

- `TypeError` : vérifier l'appelant, les valeurs nullables, les DTO et les conversions depuis `Request`, env vars ou config.
- `ArgumentCountError` : vérifier signature, injection de dépendances, factory, callable et listener.
- `Undefined array key` : valider l'entrée, distinguer clé optionnelle et contrat incomplet.
- `Call to a member function ... on null` : identifier pourquoi `null` est possible au runtime au lieu d'ajouter `?->` par défaut.
- `Cannot access uninitialized property` : initialiser dans le constructeur ou rendre l'état explicitement nullable si métier.
- `Memory exhausted` : chercher chargement massif, pagination absente, collection Doctrine lazy chargée en boucle, export ou serializer trop large.

### Erreurs Doctrine

- `MappingException` : vérifier les annotations/attributs de l'entité
- `QueryException` : vérifier la syntaxe DQL et les noms de champs
- `UniqueConstraintViolationException` : identifier la contrainte et les données en doublon
- `TableNotFoundException` : migration manquante ou non exécutée
- `NotNullConstraintViolationException` : vérifier validation, valeur par défaut, formulaire, DTO et migration.
- `ForeignKeyConstraintViolationException` : vérifier ordre de persistance/suppression, cascade, orphanRemoval et données existantes.
- `EntityManager is closed` : chercher l'exception précédente qui a fermé l'EntityManager.
- `A new entity was found through the relationship` : vérifier cascade persist ou persistance explicite de l'entité liée.
- Hydratation ou proxy cassé : vérifier constructeur, propriétés readonly, types PHP et colonnes nullables.
- Écart schéma/entités : lancer ou proposer `doctrine:schema:validate` et inspecter les migrations récentes.

### Erreurs HTTP

- 404 : vérifier le routing (`debug:router`)
- 403 : vérifier les voters, firewalls, `security.yaml`
- 405 : vérifier méthodes HTTP, formulaires, routes API et préflight CORS.
- 400 : vérifier validation, désérialisation, payload JSON, query parameters et contraintes de DTO.
- 401 : vérifier authenticator, token, session, cookies, remember me et firewalls.
- 422 : vérifier validation serveur, groupes de validation et mapping des erreurs.
- 500 : lire les logs (`var/log/dev.log`, `var/log/test.log` ou `var/log/prod.log`) et remonter la cause applicative.
- Redirection inattendue : vérifier access control, login path, trailing slash, locale et HTTPS.

### Erreurs de cache

Symptômes : modification de code sans effet, service introuvable après modification, template obsolète, config non prise en compte.

Vérifier :

- environnement utilisé (`APP_ENV`, `--env=test`, `--env=prod`) sans lire `.env.local` ;
- cache Symfony dans `var/cache/` ;
- OPcache en prod ou dans le conteneur PHP ;
- cache Doctrine metadata/query/result ;
- cache HTTP, reverse proxy ou CDN si le symptôme est côté réponse HTTP.

Vider le cache est un diagnostic ou une étape de validation, pas une correction durable si le problème revient.

### Erreurs Messenger

- Message non consommé : vérifier le routing et le transport
- Handler non appelé : vérifier `#[AsMessageHandler]` et l'autoconfigure
- Message dans failed transport : inspecter l'exception originale avec `messenger:failed:show`.
- Retry en boucle : vérifier idempotence, stratégie retry, exception transitoire ou permanente.
- Handler dupliqué : vérifier tags, autoconfiguration, bus et handlers multiples.
- Désérialisation impossible : vérifier classe du message, propriétés readonly, compatibilité après déploiement et transport.
- Transaction et async : vérifier dispatch avant/après commit, outbox éventuelle et effets de bord non rejouables.

### Erreurs de tests

- Lire le premier échec réel avant les échecs en cascade.
- Distinguer bug applicatif, fixture manquante, horloge non contrôlée, ordre de tests, dépendance externe et assertion trop fragile.
- Pour PHPUnit, relancer le test ciblé avec un filtre si possible.
- Pour tests fonctionnels, vérifier kernel reboot, base de test, transactions, fixtures et client authentifié.
- Ne pas modifier l'assertion pour l'aligner sur un comportement cassé sans preuve que le nouveau comportement est attendu.

### Erreurs de configuration et environnement

- Vérifier `composer.json`, `composer.lock`, version PHP, extensions PHP, bundles activés et scripts.
- Comparer environnement CLI, HTTP, worker Messenger et conteneur Docker si le bug n'apparaît que dans un contexte.
- Vérifier variables d'environnement requises par nom uniquement, sans afficher leurs valeurs sensibles.
- Vérifier permissions de fichiers pour cache, logs, uploads et sessions.
- Pour Docker, vérifier service, réseau, ports, volumes et variables injectées.

### Erreurs de performance ou timeout

- Identifier la ressource saturée : CPU, mémoire, base, réseau, disque, verrou.
- Chercher requêtes en boucle, N+1 Doctrine, absence de pagination, sérialisation massive, appel HTTP bloquant.
- Vérifier les logs SQL ou profiler si disponibles.
- Proposer un fix ciblé : index, pagination, batch, eager loading contrôlé, cache, timeout explicite ou traitement async.

## Commandes utiles

Lancer seulement les commandes adaptées au projet :

- `composer test`, `composer phpunit` ou script équivalent si présent
- `vendor/bin/phpunit --filter NomDuTest`
- `bin/console about`
- `bin/console debug:router`
- `bin/console debug:router nom_route`
- `bin/console debug:container Service\\Id`
- `bin/console debug:event-dispatcher`
- `bin/console debug:autowiring`
- `bin/console lint:container`
- `bin/console lint:yaml config/`
- `bin/console messenger:failed:show`
- `bin/console messenger:failed:retry`
- `bin/console doctrine:schema:validate`
- `bin/console doctrine:migrations:status`
- `composer diagnose`
- `composer show vendor/package`

Ne pas lancer de commande destructive (`migrate`, retry massif, purge, truncate, reset DB, cache pool clear prod) sans confirmation explicite.

## Corrections attendues

- Corriger le contrat d'entrée si une donnée invalide arrive depuis une requête, commande, message ou config.
- Ajouter une garde explicite quand le cas d'erreur est métier.
- Corriger l'injection de dépendance ou la configuration si le container est la cause.
- Corriger la requête, le mapping ou la migration si Doctrine est la cause.
- Corriger la route, le voter, le firewall ou l'authenticator si HTTP/security est la cause.
- Ajouter un test de non-régression ciblé quand la correction touche un comportement observable.

## Ne pas faire

- Ne pas masquer une exception sans traiter la cause.
- Ne pas remplacer une erreur par un `return null` silencieux.
- Ne pas ajouter `@phpstan-ignore`, `@psalm-suppress` ou une assertion mensongère pour contourner le problème.
- Ne pas supprimer un test en échec sans prouver qu'il est obsolète.
- Ne pas vider ou modifier des données locales sans accord.
- Ne pas exposer secrets, tokens, cookies, headers Authorization ou données personnelles dans la réponse.
- Ne pas supposer que l'erreur vient du cache sans preuve.

## Format de sortie

Répondre avec :

1. **Diagnostic** : cause racine en une phrase, avec niveau de confiance si nécessaire.
2. **Preuves** : fichiers, lignes, stacktrace, commande ou log qui justifient le diagnostic.
3. **Correction** : changement appliqué ou patch proposé, limité à la cause.
4. **Validation** : commande relancée, résultat, ou raison pour laquelle elle n'a pas pu être lancée.
5. **Prévention** : test de non-régression, garde, monitoring ou règle projet utile.

Si la cause n'est pas encore prouvée, présenter les hypothèses triées par probabilité et l'action de diagnostic suivante pour chacune.
