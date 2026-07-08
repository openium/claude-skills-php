---
name: phpstan
description: "Corrige les erreurs PHPStan dans un projet PHP/Symfony. Analyse un rapport, un fichier ou le projet, interprète les erreurs de typage, generics, Doctrine, Symfony et applique des corrections sans @phpstan-ignore, sans réduire le niveau et sans aggraver la baseline."
---

# Correction des erreurs PHPStan

## Périmètre

Si l'utilisateur fournit une erreur PHPStan, corriger cette erreur en priorité.

Si l'utilisateur précise un fichier ou dossier, limiter l'analyse à ce périmètre.

Sinon, utiliser la commande PHPStan du projet si elle existe. Ne pas installer PHPStan sans demande explicite.

## État des lieux

Avant de corriger, inspecter selon le projet :

- `phpstan.neon`, `phpstan.neon.dist` et fichiers inclus
- `composer.json` : scripts, version PHP, dépendances PHPStan
- Niveau PHPStan actuel, chemins analysés, extensions Symfony/Doctrine/PHPUnit
- Baseline éventuelle et règles custom

Ne pas modifier le niveau, `ignoreErrors`, la baseline ou les chemins analysés pour faire disparaître une erreur.

## Processus

1. Reproduire l'erreur ou lire le message PHPStan exact.
2. Identifier le fichier, la ligne, le type attendu et le type réellement possible.
3. Lire le code appelé et les usages publics avant de modifier une signature.
4. Corriger la cause racine : type, contrôle de flux, validation, initialisation ou contrat.
5. Relancer PHPStan sur le périmètre ciblé si possible.
6. Lancer les tests ciblés si le comportement runtime peut être impacté.

## Règles absolues

- Ne JAMAIS ajouter `@phpstan-ignore-*` pour masquer une erreur
- Ne JAMAIS réduire le niveau PHPStan dans la configuration
- Ne JAMAIS ajouter d'entrée dans `ignoreErrors` du fichier `phpstan.neon`
- Chaque correction doit résoudre le problème réel, pas le symptôme
- Ne pas ajouter `mixed`, `array` ou `object` si un type précis est disponible
- Ne pas rendre nullable uniquement pour satisfaire PHPStan
- Ne pas ajouter de `@var` mensonger
- Ne pas changer le comportement métier pour satisfaire le typage
- Ne pas supprimer du code mort sans vérifier les usages indirects Symfony, Doctrine, Serializer ou reflection

## Corrections courantes

### Appel de méthode sur un type nullable

- Ne pas utiliser `?->` comme solution par défaut.
- Si null est impossible, corriger le contrat ou ajouter une garde explicite.
- Si null est légitime, gérer le cas avec exception, early return ou branche métier claire.

### Type de retour incorrect

- Ajouter le type de retour correct.
- Si plusieurs types sont réels, utiliser une union ou extraire un objet résultat.
- Ne pas élargir vers `mixed` pour éviter de comprendre le flux.

### Propriété non initialisée

- Initialiser dans le constructeur quand la valeur est requise.
- Ajouter une valeur par défaut seulement si elle a un sens métier.
- Marquer nullable uniquement si l'absence de valeur est un état valide.

### Dead code

Supprimer le code mort si confirmé. Vérifier qu'il n'y a pas de magie (reflection, container) avant de supprimer.

### Generics et templates

- Ajouter `@template`, `@extends`, `@implements` ou `@use` quand la classe est réellement générique.
- Typer les collections Doctrine : `Collection<int, Entity>`.
- Typer les repositories : `ServiceEntityRepository<Entity>` ou équivalent.
- Typer les iterables : `iterable<int, Foo>` ou `Generator<int, Foo, mixed, void>`.

### Tableaux et shapes

- Préférer un DTO quand la structure est métier ou traverse plusieurs couches.
- Utiliser une array shape pour une structure locale et stable.
- Vérifier `isset`, `array_key_exists` ou une validation avant accès à une clé optionnelle.
- Utiliser `non-empty-string`, `positive-int`, `class-string<T>` seulement si le code garantit réellement cette contrainte.

### Callables et closures

- Décrire les signatures avec `callable(Input): Output` ou une interface dédiée.
- Typer les paramètres et retours des closures passées à `array_map`, `filter`, collections ou promises.

## Symfony et Doctrine

- Les valeurs venant de `Request`, `InputInterface`, `ParameterBag`, env vars ou config doivent être validées ou converties.
- Après `Repository::find()`, gérer le cas `null` avant d'utiliser l'entité.
- Ne pas utiliser `/** @var Entity $entity */` après un `find()` nullable sans garde réelle.
- Typer les collections, repositories, query builders et résultats de requêtes.
- Pour les services, préférer l'interface existante quand elle représente le contrat réel.
- Ne pas récupérer un service depuis le container pour contourner une erreur de type.

## PHPDoc et annotations

PHPDoc est acceptable pour :

- Generics, templates, array shapes, callable signatures
- Types Doctrine non exprimables en PHP natif
- Précision locale après une validation runtime réelle

PHPDoc ne doit pas mentir sur une valeur possible au runtime. Préférer les types natifs quand ils suffisent.

## Validation

- Relancer PHPStan sur le fichier ou le périmètre modifié si possible.
- Si la commande complète est trop coûteuse, lancer le plus petit périmètre utile.
- Si PHPStan ne peut pas être lancé, expliquer pourquoi et indiquer la commande à exécuter.
- Lancer les tests ciblés quand une correction touche le comportement, pas seulement les types.

## Format de sortie

Résumer :

- Erreurs PHPStan initiales corrigées
- Fichiers modifiés
- Cause racine
- Correction appliquée
- Commandes PHPStan lancées et résultat
- Tests lancés si pertinents
- Risques ou erreurs restantes
