---
name: observability
description: "Audite et améliore l'observabilité d'un projet PHP/Symfony. Analyse logs Monolog, erreurs, métriques, traces, correlation IDs, Sentry, Messenger, HTTP clients, données sensibles et alerting pour diagnostiquer la production sans exposer de secrets."
---

# Observabilité PHP/Symfony

## Périmètre

Déterminer si l'utilisateur demande :

- Audit des logs, erreurs, traces ou métriques.
- Ajout de logs autour d'un flux métier, endpoint, commande ou handler Messenger.
- Intégration ou correction Sentry, Monolog, OpenTelemetry, Prometheus ou outil équivalent.
- Diagnostic d'un manque d'information en production.
- Réduction de bruit ou fuite de données sensibles dans les logs.

Si aucun périmètre n'est donné :

- Lire `git status` et le diff si l'audit porte sur les changements récents.
- Inspecter Monolog/config et les chemins critiques modifiés.
- Demander le flux ou l'incident à rendre observable si nécessaire.

Ne pas lire ni afficher `.env.local`, DSN, tokens ou données personnelles sensibles.

## État des lieux

Inspecter selon le projet :

- `composer.json` : MonologBundle, Sentry, OpenTelemetry, Prometheus, Messenger, HttpClient
- `config/packages/monolog.yaml`, `sentry.yaml`, messenger, framework/http_client
- Services, controllers, commands, handlers, listeners et clients externes concernés
- Logs existants, processors Monolog, channels, handlers, format JSON ou texte
- CI/prod config si l'observabilité est déployée par environnement
- Tests qui vérifient logs ou comportements d'erreur si présents

Ne jamais exposer de secret ou payload sensible en réponse.

## Objectifs

Une bonne observabilité doit permettre de répondre :

- Qu'est-ce qui a échoué ?
- Pour quel utilisateur, tenant, organisation ou ressource, sans exposer de donnée sensible ?
- Quelle entrée métier ou identifiant stable permet de retrouver le contexte ?
- Quelle dépendance externe ou composant est impliqué ?
- Est-ce transitoire, permanent, métier ou technique ?
- Comment corréler logs, traces, métriques et message Messenger ?

## Sévérités

- **Bloquant** : fuite de secrets/PII, erreur critique silencieuse, exception avalée, paiement/email/écriture critique non traçable, logs contenant tokens.
- **Important** : absence de correlation ID, logs insuffisants sur flux critique, bruit massif, métrique ou alerte manquante, failed messages non observables.
- **Suggestion** : enrichissement de contexte, nommage, channel dédié, structuration JSON, dashboard ou alerte utile.

## Logs

- Utiliser des logs structurés avec contexte plutôt que concaténer des chaînes.
- Inclure identifiants stables : request id, user id, tenant id, entity id, message id, external id.
- Ne pas logger mots de passe, tokens, cookies, Authorization header, DSN, payload complet sensible ou données personnelles non nécessaires.
- Choisir le niveau correct : debug, info, notice, warning, error, critical.
- Éviter les logs en boucle ou sur chemin chaud sans garde.
- Utiliser channels Monolog dédiés pour flux critiques si le projet le fait déjà.
- Ne pas avaler une exception après log si le retry/failure doit se déclencher.

## Erreurs et exceptions

- Distinguer erreur métier attendue et incident technique.
- Ne pas masquer une exception par un retour silencieux.
- Ajouter contexte métier aux exceptions ou aux logs, sans fuite sensible.
- Vérifier que les erreurs critiques remontent vers l'outil d'alerting.
- Pour API, ne pas exposer stacktrace ou détails internes au client.
- Pour commandes, retourner un code de sortie cohérent.

## Correlation IDs et traces

- Vérifier propagation d'un request id / correlation id.
- Propager le contexte vers appels HTTP sortants si convention projet.
- Propager un identifiant de corrélation dans les messages Messenger si utile.
- Éviter de générer un nouvel id à chaque couche si un id existe déjà.
- Pour OpenTelemetry/APM, instrumenter les frontières : HTTP, DB, queue, clients externes.

## Metrics et alerting

- Identifier les métriques utiles : taux d'erreur, latence, durée de job, backlog queue, failed messages, retries, timeouts externes.
- Ajouter des métriques métier seulement si elles servent une alerte ou un dashboard.
- Prévoir labels à cardinalité maîtrisée : pas d'email, UUID utilisateur massif ou payload dynamique en label.
- Définir seuils et symptômes, pas seulement compteur brut.
- Éviter les alertes trop bruyantes qui ne déclenchent aucune action.

## Messenger et async

- Logger début/fin/échec des handlers critiques avec message id et identifiants métier.
- Observer retry count, failed transport, durée de traitement et mémoire si pertinent.
- Ne pas logger le message complet si payload sensible.
- Différencier erreur transitoire et permanente.
- Conserver l'exception pour que Messenger puisse retry/failure si nécessaire.
- Vérifier que `messenger:failed:*` ou dashboard équivalent est exploitable.

## HTTP clients et dépendances externes

- Toujours avoir timeout explicite.
- Logger service cible, opération, statut, durée, retry count et correlation id.
- Ne pas logger headers sensibles ni payload complet par défaut.
- Classer erreurs réseau, 4xx métier, 5xx fournisseur et timeout.
- Ajouter métriques ou traces sur dépendances critiques.

## Sécurité et conformité

- Redacter secrets, tokens, cookies, Authorization, reset-password tokens, magic links.
- Limiter données personnelles : email complet, téléphone, adresse, nom complet seulement si nécessaire et autorisé.
- Vérifier logs d'emails, webhooks, paiements, fichiers uploadés.
- Éviter d'envoyer données sensibles vers services tiers d'observabilité sans filtrage.
- Ne pas lire `.env.local` pour vérifier un DSN.

## Tests et validation

Prévoir selon le changement :

- Test que l'exception n'est pas avalée.
- Test qu'un événement critique logge le contexte attendu.
- Test que données sensibles ne sont pas dans le contexte loggé si facilement testable.
- Test fonctionnel d'erreur API sans fuite de stacktrace.
- Test handler Messenger sur erreur transitoire/permanente.

Commandes utiles :

- `bin/console debug:config monolog`
- `bin/console debug:container monolog`
- `bin/console debug:event-dispatcher`
- `bin/console messenger:failed:show`
- Tests ciblés

Ne pas déclencher d'appels externes réels ou alertes production sans confirmation explicite.

## Ne pas faire

- Ne pas logger des secrets ou payloads complets pour "debugger vite".
- Ne pas remplacer une exception par un simple log.
- Ne pas ajouter des logs verbeux sur chemin chaud sans niveau ou sampling adapté.
- Ne pas créer des métriques à cardinalité incontrôlée.
- Ne pas envoyer de données personnelles à un outil tiers sans filtrage.
- Ne pas modifier `.env.local`.
- Ne pas ajouter une dépendance d'observabilité sans demande explicite.

## Format de sortie

Présenter les findings triés par sévérité décroissante.

Pour chaque finding :

- Fichier et ligne
- Sévérité : bloquant, important ou suggestion
- Catégorie : logs, erreurs, traces, metrics, alerting, Messenger, HTTP client, sécurité
- Preuve vérifiée
- Impact opérationnel
- Correction proposée
- Test ou vérification conseillé

Après modification, résumer :

- Fichiers modifiés
- Contexte ajouté ou retiré
- Données sensibles protégées
- Commandes/tests lancés
- Limites restantes
