---
name: deprecations
description: "Scanne et corrige les deprecations PHP et Symfony dans un projet PHP/Symfony, y compris legacy PHP 7.0+. Identifie le code obsolète, la version de suppression, l'origine projet ou vendor, et propose des remplacements compatibles avec la version cible sans imposer d'upgrade."
---

# Détection des deprecations

## Périmètre

Déterminer si l'utilisateur demande :

- Un diagnostic uniquement : ne pas modifier le code.
- L'application de corrections : procéder par lots vérifiables.
- Un périmètre ciblé : PHP, Symfony, Doctrine, bundle, configuration, tests, fichier ou dossier.
- Une version cible : PHP 7.x, PHP 8.x, Symfony 2.8/3.4/4.4/5.4/6.4/7.x, ou une dépendance précise.

Si aucune version cible n'est donnée, analyser la version actuelle du projet et signaler les deprecations pertinentes sans supposer une montée de version.

Ne jamais monter la version PHP, Symfony ou une dépendance sans validation explicite.

## État des lieux

Inspecter selon le projet :

- `composer.json`, `composer.lock`, `symfony.lock`
- Version PHP requise, `config.platform.php`, contraintes Composer et extensions PHP
- Version Symfony, bundles tiers, Doctrine, PHPUnit, PHPStan, Rector
- `phpunit.xml*`, `phpstan.neon*`, `rector.php`
- `config/bundles.php`, `config/packages/`, `config/routes/`
- Scripts Composer de test, analyse statique ou deprecation helper
- CI, Dockerfile ou configuration runtime si la version PHP réelle est ambiguë

Ne jamais lire ni modifier `.env.local`.

## Processus

1. Identifier les versions actuelles et la cible éventuelle.
2. Reproduire ou collecter les deprecations : logs, tests, `debug:container --deprecations`, PHPUnit, PHPStan, Rector dry-run si présent.
3. Classer chaque deprecation : PHP, Symfony, Doctrine, PHPUnit, bundle tiers, code projet, configuration.
4. Vérifier la version où l'API est supprimée ou devient incompatible.
5. Corriger d'abord le code projet et la configuration, puis proposer une stratégie pour les dépendances tierces.
6. Relancer la commande qui exposait la deprecation.
7. Lancer les tests ciblés si le comportement runtime peut être impacté.

Ne pas mélanger nettoyage de deprecations, upgrade majeure et refactor large sauf demande explicite.

## Priorités

- **Bloquant** : API supprimée dans la version cible, deprecation qui casse les tests avec `SYMFONY_DEPRECATIONS_HELPER`, incompatibilité runtime certaine.
- **Important** : deprecation sur chemin critique, bruit massif qui masque d'autres erreurs, correction nécessaire avant upgrade proche.
- **Suggestion** : remplacement moderne utile mais non requis pour la version cible actuelle.

## Deprecations PHP

Toujours vérifier la version PHP actuelle et la version cible avant de corriger. Pour un projet legacy, proposer un remplacement compatible avec la version minimale supportée.

### PHP 7.0

- Constructeurs style PHP 4 (`function ClassName()`) : utiliser `__construct()`.
- `preg_replace()` avec modificateur `/e` supprimé : remplacer par `preg_replace_callback()`.
- Extensions supprimées ou obsolètes selon le runtime : `mysql_*`, `ereg_*`, `mcrypt` à traiter avec prudence selon la version réelle.
- Méthodes ou usages incompatibles avec le typage scalaire si le projet commence à typer progressivement.

### PHP 7.1

- Appels à des fonctions avec trop peu d'arguments : corriger la signature ou l'appel.
- `mcrypt` déprécié : migrer vers `openssl` ou `sodium` selon le besoin.
- `each()` déprécié : remplacer par `foreach`.
- `create_function()` déprécié : remplacer par closure.

### PHP 7.2

- `__autoload()` déprécié : utiliser `spl_autoload_register()`.
- `count()` sur type non countable : vérifier le type ou utiliser une garde.
- `assert()` avec string : remplacer par expression booléenne.
- `parse_str()` sans second argument : fournir un tableau de sortie.

### PHP 7.3

- `FILTER_FLAG_SCHEME_REQUIRED` et `FILTER_FLAG_HOST_REQUIRED` dépréciés.
- `case-insensitive constants` dépréciées : utiliser des constantes sensibles à la casse.
- Certaines syntaxes heredoc/nowdoc anciennes peuvent nécessiter une normalisation avant upgrade.

### PHP 7.4

- Accès array/string avec accolades : `$foo{0}` -> `$foo[0]`.
- `get_magic_quotes_gpc()` et magic quotes : supprimer les branches mortes.
- `implode($pieces, $glue)` ancien ordre : utiliser `implode($glue, $pieces)`.
- Propriétés typées à introduire seulement si compatible avec la version minimale du projet.

### PHP 8.0

- Signature de méthodes incompatibles avec les interfaces internes : aligner les signatures.
- Paramètres obligatoires après paramètres optionnels : réordonner ou rendre la signature explicite.
- `match`, nullsafe, attributes disponibles mais ne pas les introduire si la compatibilité PHP 7.x doit rester.
- Warnings transformés en `TypeError` ou `ValueError` : corriger les entrées plutôt que masquer l'erreur.

### PHP 8.1

- `FILTER_SANITIZE_STRING`, `strftime()`, `utf8_encode()`/`utf8_decode()`
- Retour implicite de `null` pour méthodes internes non compatible : aligner les types.
- Passage de `null` à des paramètres internes non nullable : ajouter conversion ou garde explicite.
- Serializable déprécié sans `__serialize()` / `__unserialize()`.
- `DateTime` et `Intl` : vérifier les formats dépendants de locale.

### PHP 8.2

- Propriétés dynamiques, `${var}` string interpolation, callables partiellement supportés
- `utf8_encode()` et `utf8_decode()` dépréciés : utiliser `mb_convert_encoding()` ou une stratégie charset explicite.
- `FILTER_SANITIZE_STRING` supprimé : remplacer par validation/normalisation adaptée au contexte.
- `DateTimeInterface::ISO8601` à éviter selon le contexte : préférer `DateTimeInterface::ATOM` si le contrat le permet.

### PHP 8.3

- `NumberFormatter::TYPE_CURRENCY`
- Vérifier les deprecations des extensions `intl`, `mbstring`, `random`, `DateTime` selon les usages.
- Corriger seulement si la version cible ou les tests les exposent.

### PHP 8.4

- Constantes de classes sans visibilité explicite
- Paramètres implicitement nullable via `Type $param = null` dépréciés : utiliser `?Type $param = null`.
- Nouvelles deprecations des extensions internes : vérifier la documentation et les logs du projet.

## Deprecations Symfony

Toujours vérifier la version Symfony actuelle, la version cible et la version PHP minimale avant de proposer un remplacement. Les projets legacy peuvent nécessiter une correction compatible avec annotations, YAML/XML, PHP 7.x ou anciens composants.

### Symfony 6.4 LTS

- Corriger les deprecations avant Symfony 7 : options de configuration supprimées, signatures typées, services ou aliases retirés.
- Vérifier les bundles tiers compatibles Symfony 7.
- Privilégier attributs, injection explicite et types modernes si la version PHP minimale du projet le permet.
- Vérifier Security, Messenger, Serializer, Notifier, HttpClient et Doctrine Bridge.

### Symfony 7.x

- Vérifier les suppressions issues des deprecations Symfony 6.4.
- Les remplacements doivent rester compatibles avec la version mineure ciblée.
- Ne pas utiliser des APIs introduites en Symfony 7.1+ si le projet cible Symfony 7.0.

- Annotations au lieu d'attributs PHP 8
- `ContainerAwareTrait` / `ContainerAwareCommand`
- `AbstractController::getDoctrine()`
- `@Route` annotation au lieu de `#[Route]`
- `EventSubscriberInterface` remplacé par `#[AsEventListener]` (Symfony 6.2+)
- Services récupérés directement depuis le container au lieu de l'injection explicite
- Alias de services, paramètres ou options de configuration renommés
- Config YAML déplacée ou renommée entre versions Symfony
- Security : guard authenticator remplacé, voters ou access control à adapter selon la version
- Forms : options dépréciées, types ou extensions renommés
- Validator : contraintes, options ou annotations remplacées par attributs selon la cible
- Serializer : normalizers, context options et groupes à vérifier
- Routing : annotations Doctrine vs attributs PHP selon la version PHP minimale
- Event Dispatcher : noms d'événements string remplacés par classes ou attributs selon le composant
- Doctrine Bridge : registry, annotations, mapping et migrations à vérifier

Ne pas remplacer automatiquement annotations par attributs si le projet doit rester compatible PHP 7.x.

## Deprecations Doctrine, PHPUnit et outils

- Doctrine annotations vers attributs : seulement si PHP 8+ est validé.
- Doctrine DBAL : types, plateformes, `executeUpdate()` vers `executeStatement()`, méthodes de fetch remplacées.
- Doctrine ORM : annotations, proxy, mapping driver, repository signatures.
- PHPUnit : annotations vers attributs seulement si la version PHP cible le permet.
- PHPUnit assertions ou signatures de hooks (`setUp`, `tearDown`) à aligner avec la version installée.
- PHPStan/Psalm : ne jamais ajouter d'ignore pour masquer une deprecation.

## Dépendances et vendor

- Ne jamais modifier `vendor/`.
- Si la deprecation vient d'un package tiers, identifier le package et la version.
- Vérifier si une version plus récente corrige la deprecation.
- Utiliser `composer why`, `composer why-not` ou `composer outdated` pour expliquer le blocage.
- Proposer un update, un remplacement de package ou une issue upstream si le projet ne peut pas corriger directement.
- Ne pas supprimer un bundle sans vérifier ses usages dans code, config, templates et tests.

## Rector

Si Rector est déjà présent (`rector.php`, dépendance `rector/rector` ou script Composer dédié), il peut aider aux migrations mécaniques.

Règles :

- Ne pas installer Rector sans demande explicite.
- Lancer d'abord Rector en dry-run.
- Limiter le périmètre si l'utilisateur a demandé un fichier, dossier ou type de deprecation.
- Relire le diff avant de considérer la correction valide.
- Refuser les changements qui modifient le comportement métier.
- Ne pas introduire de syntaxe PHP plus récente que la version minimale supportée.

## Commandes utiles

Adapter les commandes au projet :

- `bin/console debug:container --deprecations`
- `bin/console about`
- `composer outdated`
- `composer why vendor/package`
- `composer why-not vendor/package version`
- `vendor/bin/phpunit`
- `SYMFONY_DEPRECATIONS_HELPER=weak vendor/bin/phpunit`
- `SYMFONY_DEPRECATIONS_HELPER=max[self]=0 vendor/bin/phpunit`
- `vendor/bin/phpstan analyse`
- `vendor/bin/rector process --dry-run` si Rector est présent

Rapporter les commandes lancées et leur résultat. Ne pas lancer de commande coûteuse ou destructive sans raison claire.

## Ne pas faire

- Ne pas masquer ou ignorer les deprecations.
- Ne pas changer la version PHP, Symfony ou Composer sans validation explicite.
- Ne pas modifier `.env.local` ou des secrets.
- Ne pas introduire attributs, enums, readonly, nullsafe ou autre syntaxe PHP 8 dans un projet qui supporte PHP 7.x.
- Ne pas remplacer une API par une autre sans vérifier la version minimale qui l'introduit.
- Ne pas faire de refactor large pour corriger une deprecation locale.
- Ne pas modifier `composer.lock` à la main.
- Ne pas supprimer un package ou bundle sans preuve qu'il n'est plus utilisé.

## Format de sortie

Pour un diagnostic :

- Versions détectées : PHP, Symfony, dépendances clés
- Cible analysée ou hypothèse utilisée
- Liste des deprecations avec fichier, ligne, origine, message, version de suppression
- Remplacement recommandé compatible avec la version minimale du projet
- Priorité : bloquant, important, suggestion
- Commandes lancées et résultat
- Plan de correction ordonné

Après correction :

- Fichiers modifiés
- Deprecations corrigées
- Remplacements appliqués
- Commandes relancées et résultat
- Tests exécutés ou non exécutés
- Risques, deprecations restantes ou dépendances tierces bloquantes
