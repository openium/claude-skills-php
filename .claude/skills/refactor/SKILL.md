---
name: refactor
description: "Refactoring sûr pour projet PHP/Symfony. À utiliser pour simplifier une classe ou méthode, extraire une logique métier, réduire la duplication, déplacer du code entre Domain/Application/Infrastructure, ou améliorer la conception sans changer le comportement observable."
---

# Refactoring PHP/Symfony

## Périmètre

Si l'utilisateur précise un fichier, une classe, un dossier ou un diff, limiter le refactor à ce périmètre.

Si le périmètre est ambigu ou trop large, demander une cible avant de modifier le code.

## Processus

1. Lire le code cible et ses tests existants.
2. Rechercher les appels entrants et usages publics de la classe ou méthode.
3. Identifier l'architecture réelle du projet : MVC classique, hexagonale, DDD partiel ou autre.
4. Choisir le plus petit refactor qui améliore le code sans changer le comportement.
5. Appliquer les modifications par étapes cohérentes, en gardant le code compilable.
6. Lancer les tests ciblés si possible, ou indiquer précisément ceux à lancer.

## Principes

- Préserver le comportement observable : mêmes entrées, sorties, exceptions, effets de bord et contrats publics.
- Respecter les conventions existantes du projet avant d'appliquer une règle générale.
- Garder les contrôleurs, commandes, handlers et listeners minces : parsing, délégation, output.
- Déplacer la logique métier vers un service applicatif, un use case ou le Domain selon l'architecture existante.
- Préférer des noms qui décrivent l'intention métier plutôt que la mécanique technique.
- Réduire le couplage sans disperser inutilement le code.

## Architecture

- Ne pas imposer une architecture hexagonale complète à un projet MVC simple.
- Si le projet est hexagonal, le Domain ne dépend pas de Symfony, Doctrine, Infrastructure ou Framework.
- Si des use cases existent, les placer dans Application/ et leur laisser l'orchestration.
- Les repositories exécutent les requêtes ; les décisions métier restent dans Domain/Application.
- Les adapters Infrastructure implémentent les ports ou interfaces attendus par Domain/Application.
- Ne pas déplacer une classe seulement pour suivre une théorie si le projet n'a pas cette organisation.

### Principes SOLID

- **S** : une classe a une responsabilité identifiable.
- **O** : préférer l'extension seulement quand plusieurs variantes existent ou sont prévues explicitement.
- **L** : un sous-type doit rester substituable sans surprise.
- **I** : préférer des interfaces petites et ciblées.
- **D** : dépendre d'abstractions quand cela protège réellement le Domain ou un point d'extension.

## Refactors courants

- Méthode > 20 lignes : extraire en méthodes privées nommées par intention
- Classe > 250 lignes : identifier les responsabilités avant d'extraire
- Contrôleur avec logique métier : extraire vers un service applicatif ou un use case
- Commande console lourde : extraire la logique vers un service, garder la commande pour l'I/O
- Listener, subscriber ou handler lourd : extraire vers un service dédié
- Repository avec logique métier : extraire vers un service du Domain
- Conditions imbriquées : privilégier early returns, méthodes privées nommées ou stratégie si plusieurs variantes réelles existent
- Duplication de validation : extraire vers DTO, constraint, validator ou service selon les conventions du projet
- Construction complexe d'objet : extraire une factory seulement si la construction est réutilisée ou porte une règle métier

## Abstractions

- Ne pas créer d'abstraction pour un seul cas d'utilisation.
- Interface acceptable pour une dépendance sortante du Domain ou un point d'extension réel.
- Stratégie acceptable si au moins deux comportements existent ou sont explicitement demandés.
- Factory acceptable si la construction est complexe, réutilisée ou métier.
- Service dédié acceptable si le comportement extrait porte un nom clair et réduit une responsabilité.
- Éviter les abstractions "au cas où".

## Tests et validation

- Lancer les tests ciblés quand ils existent.
- Lancer PHPStan si le projet l'utilise et que le refactor touche les types, signatures ou generics.
- Ajouter ou adapter un test seulement si nécessaire pour sécuriser le refactor.
- Si aucun test ne couvre le comportement refactoré, signaler le risque dans la réponse.

## Ne pas faire

- Ne pas extraire une interface si une seule classe l'implémente (sauf pour le Domain)
- Ne pas renommer sans raison
- Ne pas changer une signature publique sans nécessité
- Ne pas modifier les payloads API, codes HTTP, exceptions métier ou règles de validation
- Ne pas modifier une migration existante déjà potentiellement appliquée
- Ne pas changer les noms de routes, commandes, services ou handlers sans demande explicite
- Ne pas mélanger refactor et changement fonctionnel dans la même modification

## Format de sortie

Résumer :

- Fichiers modifiés
- Refactors appliqués et raison courte
- Comportement préservé
- Tests ou vérifications lancés
- Risques ou limites restantes
