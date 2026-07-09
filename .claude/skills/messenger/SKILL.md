---
name: messenger
description: "Audite et corrige Symfony Messenger. Analyse transports, routing, buses, handlers, retry/failure, serializers, workers, idempotence, transactions et performance, en tenant compte des versions Symfony/PHP et des conventions projet."
---

# Vérification Symfony Messenger

## Périmètre

Déterminer si l'utilisateur demande :

- Un audit de configuration Messenger.
- Le debug d'un message non consommé, failed ou traité plusieurs fois.
- L'ajout d'un message, handler, transport ou routing.
- Une correction de retry, failure transport, serializer ou worker.
- Un audit de performance ou fiabilité de workers.
- Un périmètre ciblé : message, handler, transport, bus, environnement ou fichier.

Si aucun périmètre n'est donné, analyser la configuration Messenger et les messages/handlers du projet.

Ne pas lancer de worker long-running ou de retry massif sans confirmation explicite.

## État des lieux

Inspecter selon le projet :

- `composer.json` : version Symfony, Messenger, transports installés, Serializer, Doctrine
- Configuration Messenger : `config/packages/messenger.yaml`, `.php`, `.xml` et variantes `when@test`, `when@prod`
- Transports, failure transports, routing, buses, middleware, serializers
- Messages et handlers dans `src/`, tags services, attributs ou interfaces
- `config/services.yaml`, `config/services_test.yaml`, autowire/autoconfigure
- Tests Messenger, transports in-memory/test, fixtures
- Logs Symfony et messages failed si le problème est runtime
- Configuration worker/supervision si présente : Docker, systemd, Supervisor, Kubernetes, CI

Ne jamais lire `.env.local`. Vérifier les noms de variables d'environnement sans afficher leurs valeurs sensibles.

## Processus

1. Lire la configuration Messenger (`config/packages/messenger.yaml` ou `.php`)
2. Scanner tous les messages et handlers
3. Croiser les informations et détecter les incohérences
4. Identifier la version Symfony/PHP et les conventions de déclaration des handlers
5. Vérifier routing, transports, retry, failure transport, serializer et bus
6. Proposer les corrections et validations ciblées

## Compatibilité Symfony

- Symfony ancien / PHP 7.x : handlers souvent déclarés par tags services ou autoconfigure, pas par attributs PHP.
- Symfony récent / PHP 8+ : `#[AsMessageHandler]` possible si déjà convention projet.
- Ne pas convertir tags vers attributs si le projet doit rester compatible PHP 7.x.
- Respecter la configuration existante : YAML, PHP, XML, env-specific config.
- Vérifier les différences de commandes et options selon la version Symfony installée.

## Sévérités

- **Bloquant** : message sans handler, transport routé non défini, fallback sync involontaire sur traitement long, retry infini, serializer incompatible avec transport partagé, handler non idempotent sur message rejouable critique.
- **Important** : failure transport absent, retry policy fragile, mauvais bus, handler trop lourd, appel externe sans timeout, message contient une entité Doctrine, transaction mal maîtrisée.
- **Suggestion** : nommage, logs, découpage handler, simplification de routing, observabilité ou tests supplémentaires.

## Critères d'analyse

### Routage (bloquant)

- Message sans handler enregistré
- Handler enregistré pour un message qui n'existe pas
- Message routé vers un transport qui n'est pas défini
- Message synchrone qui devrait être asynchrone (traitement long, appel externe)
- Message asynchrone sans transport explicite (fallback sur sync silencieux)
- Message routé vers le mauvais bus
- Routing trop large qui capture des messages non prévus
- Command et event traités de la même manière alors que leurs garanties diffèrent
- Message critique non routé en environnement `prod` mais routé en `dev` ou `test`
- Transport de test différent du comportement prod sans justification

### Transports (important)

- Transport Doctrine utilisé pour fort volume sans vérifier polling, index et nettoyage
- AMQP/RabbitMQ sans exchange/queue/routing key cohérents
- Redis/SQS sans options de visibilité, délai ou retry adaptées
- Transport sync utilisé involontairement
- Transport in-memory utilisé uniquement en test, jamais supposé en prod
- DSN manquante ou variable d'environnement non documentée par nom
- Noms de queues incohérents avec l'environnement ou le domaine
- Absence de séparation entre messages critiques et traitements lents

### Retry et failure (important)

- Transport async sans retry policy définie
- Retry infini (pas de `max_retries`)
- Absence de failure transport pour les transports async
- Retry identique pour erreurs permanentes et transitoires
- Backoff trop agressif ou absent sur dépendance externe
- Pas de jitter alors que plusieurs messages peuvent réessayer en même temps
- Failure transport non surveillé ou non documenté
- Commandes de retry/remove utilisées sans inspection préalable de l'exception

### Message design (important)

- Message qui contient une entité Doctrine au lieu d'un identifiant stable
- Message non sérialisable : closure, ressource, service, proxy Doctrine, objet mutable complexe
- Message contenant trop de données ou des données sensibles inutiles
- Message mutable alors qu'il devrait être immutable
- Message incompatible entre deux déploiements successifs
- Nom de message trop technique ou pas aligné avec l'intention métier

### Idempotence et transactions (bloquant)

- Handler non idempotent alors que le message peut être rejoué
- Envoi d'email, paiement, appel externe ou écriture critique répété sans garde
- Dispatch d'un message avant commit DB avec risque de lecture d'état non persisté
- Transaction Doctrine trop large autour d'appels externes
- Absence de verrou ou contrainte unique pour éviter les doubles traitements
- Pas d'outbox alors que le projet en utilise une pour garantir dispatch après commit

### Handlers (important)

- Handler qui contient de la logique métier lourde (> 50 lignes)
- Handler qui fait des appels HTTP synchrones sans timeout
- Handler qui flush Doctrine dans une boucle
- Handler sans gestion d'erreur pour les cas d'échec récupérables
- Handler qui récupère des services depuis le container au lieu d'injection explicite
- Handler qui mélange orchestration, logique métier et I/O externe
- Handler sans logs utiles pour message critique
- Handler qui avale une exception et empêche retry/failure
- Handler qui ne distingue pas exception transitoire et erreur métier permanente
- Handler batch sans `flush()`/`clear()` périodique ou limite de mémoire

### Serialization (important)

- Transport externe (RabbitMQ, SQS) sans serializer explicite
- Serializer PHP natif sur un transport partagé entre applications
- Changement de classe ou namespace de message sans stratégie de compatibilité
- Propriétés readonly ou types stricts incompatibles avec le serializer utilisé
- Données sensibles sérialisées alors qu'un identifiant suffirait
- Format non versionné pour messages échangés entre applications

### Workers et supervision (important)

- Worker sans stratégie de redémarrage sur déploiement
- Pas de `messenger:stop-workers` ou équivalent après release
- Pas de limite mémoire, temps ou nombre de messages si le projet a des fuites possibles
- Prefetch/concurrency non adaptés au coût des messages
- Pas de supervision ou healthcheck visible pour workers critiques
- Logs insuffisants pour corréler message, handler et erreur

### Performance (important)

- Message trop volumineux
- Handler qui charge trop d'entités en mémoire
- N+1 Doctrine dans un handler
- Appels HTTP séquentiels dans un handler
- Traitement long qui devrait être découpé en plusieurs messages
- Transport Doctrine non nettoyé ou table messenger_messages trop volumineuse

## Tests et validation

- Test unitaire de handler avec dépendances mockées si logique d'orchestration
- Test fonctionnel de dispatch et routing si le projet le permet
- Test idempotence pour messages rejouables
- Test d'erreur transitoire vs permanente si retry/failure est critique
- Transport in-memory ou test transport pour vérifier qu'un message est dispatché
- Ne pas utiliser de vrais transports externes dans les tests sauf test d'intégration dédié

## Commandes utiles

Adapter au projet et à la version Symfony :

- `bin/console debug:messenger`
- `bin/console debug:container --tag=messenger.message_handler`
- `bin/console messenger:failed:show`
- `bin/console messenger:failed:show --stats`
- `bin/console messenger:failed:retry`
- `bin/console messenger:failed:remove`
- `bin/console messenger:consume async -vv`
- `bin/console messenger:stop-workers`
- `bin/console lint:container`
- Tests ciblés du handler ou du dispatch

Ne pas lancer `messenger:consume` sans limite, retry massif ou remove de failed messages sans confirmation explicite.

## Ne pas faire

- Ne pas mettre une entité Doctrine dans un message.
- Ne pas supposer qu'un message async sera traité exactement une fois.
- Ne pas retry une opération non idempotente sans garde.
- Ne pas utiliser serializer PHP natif pour un transport partagé entre applications.
- Ne pas avaler les exceptions dans un handler critique.
- Ne pas router un traitement long vers sync par défaut.
- Ne pas ajouter un transport sans failure transport et retry policy adaptés.
- Ne pas afficher DSN, credentials, tokens ou payloads sensibles dans la réponse.

## Format de sortie

Présenter :

1. **Résumé** : nombre de messages, handlers, transports, buses et failure transports.
2. **Matrice de routage** : message, handler, bus, transport, retry, failure transport.
3. **Findings** triés par sévérité.

Pour chaque finding :

- Fichier et ligne
- Sévérité : bloquant, important ou suggestion
- Preuve : config, handler, message, commande ou log
- Impact concret : message perdu, traitement sync, doublon, retry infini, dette opérationnelle
- Correction proposée avec exemple de config ou code si utile
- Validation conseillée : commande, test ou scénario

Si l'analyse est limitée par l'absence de config, logs ou version Symfony, le signaler explicitement.
