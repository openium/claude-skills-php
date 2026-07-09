---
name: fixtures
description: "Génère et structure des fixtures Doctrine pour projet PHP/Symfony. Produit des données réalistes, déterministes et adaptées aux tests, avec Alice, DoctrineFixturesBundle, Foundry ou le format existant, en respectant relations, contraintes, sécurité et performance."
---

# Génération de fixtures

## Périmètre

Si l'utilisateur précise une entité, un test, un scénario métier, une API, une commande ou un dossier, limiter les fixtures à ce besoin.

Sinon :

- Identifier les fixtures existantes et leur format.
- Lire les entités Doctrine concernées.
- Lire les tests ou endpoints qui consommeront ces données si le contexte est donné.
- Demander le scénario attendu si le besoin n'est pas clair : jeu minimal, scénario fonctionnel, validation, démo, seed local ou volumétrie.

Ne pas générer des fixtures pour tout le modèle sans besoin explicite.

## État des lieux

Avant de générer ou modifier des fixtures, inspecter selon le projet :

- `composer.json` : DoctrineFixturesBundle, Alice, HautelookAliceBundle, Foundry, Faker
- Dossiers existants : `src/DataFixtures/`, `fixtures/`, `tests/DataFixtures/`, `tests/fixtures/`
- Conventions de références, groupes, factories, stories ou processors
- Entités Doctrine : champs, types, nullabilité, relations, cascades, contraintes uniques, enums
- Contraintes Symfony Validator (`#[Assert\...]`)
- Tests fonctionnels ou d'intégration qui chargent les fixtures

Ne jamais lire `.env.local`.

## Processus

1. Lire l'entité cible et ses annotations/attributs Doctrine
2. Identifier les contraintes de validation (`#[Assert\...]`)
3. Détecter les relations (ManyToOne, OneToMany, ManyToMany)
4. Déterminer le format utilisé dans le projet
5. Définir le scénario couvert et les références nécessaires
6. Générer les fixtures dans le format existant
7. Proposer ou lancer une validation ciblée si possible

## Détection du format

Inspecter le projet pour identifier le système de fixtures en place :

- `fixtures/*.yaml` ou `*.yml` : Alice (hautelook/alice-bundle ou fidry/alice-data-fixtures)
- `src/DataFixtures/*.php` : DoctrineFixturesBundle
- `tests/fixtures/` ou `tests/DataFixtures/` : fixtures de test dédiées
- `src/Factory/`, `tests/Factory/`, `Story` : Zenstruck Foundry

Priorité :

1. Utiliser le format déjà présent dans le projet.
2. Si plusieurs formats coexistent, suivre le format des fixtures les plus proches du test ou de l'entité cible.
3. Si aucun format n'est en place, proposer DoctrineFixturesBundle pour des fixtures PHP simples, ou Foundry si le projet utilise déjà des factories ailleurs.
4. Ne pas installer de bundle sans demande explicite.

## Types de jeux de données

### Jeu minimal

- Données strictement nécessaires pour un test ou un scénario.
- Peu d'objets, références explicites et stables.
- À privilégier pour les tests fonctionnels rapides.

### Scénario métier

- Données cohérentes qui racontent un cas d'usage : utilisateur actif, commande payée, facture en retard, etc.
- Nommage des références basé sur le scénario.
- Relations complètes et lisibles.

### Fixtures de validation

- Données invalides uniquement dans un fichier, groupe ou provider séparé.
- Ne pas mélanger fixtures invalides avec le jeu de données chargé par défaut.
- Documenter l'erreur attendue si la fixture sert à tester une contrainte.

### Fixtures de volume

- Générer seulement sur demande explicite.
- Séparer des fixtures par défaut.
- Prévoir batch, flush périodique et désactivation éventuelle du SQL logger si le projet le fait.
- Ne pas charger 10k objets dans les tests fonctionnels standards.

## Règles

### Données réalistes

- Noms et prénoms français (pas de "John Doe")
- Emails valides et cohérents avec les noms
- Dates dans des plages réalistes (pas de dates en l'an 3000)
- Numéros de téléphone au format français
- Adresses françaises
- SIRET, TVA intracommunautaire si pertinent
- Données métier plausibles : statuts, montants, devises, périodes, slugs, références internes
- Pas de données personnelles réelles, secrets, tokens, mots de passe réels ou clés API

### Déterminisme

- Les tests doivent rester reproductibles.
- Préférer des valeurs explicites pour les scénarios critiques.
- Si Faker est utilisé, fixer une seed quand le framework le permet.
- Éviter les dates relatives non contrôlées (`now`, `+3 days`) dans les assertions sensibles.
- Utiliser une horloge contrôlée ou des dates fixes quand le comportement dépend du temps.
- Générer des emails, slugs et références uniques de manière prévisible.

### Couverture

- Minimum 5 entrées par entité
- Varier les cas : valeurs normales, valeurs aux limites, cas spéciaux
- Inclure au moins un cas avec des champs optionnels à null
- Inclure au moins un cas avec des valeurs longues (proches des limites VARCHAR)
- Ne pas surproduire des données si le test n'en a besoin que d'une ou deux.
- Couvrir les statuts ou transitions importantes du domaine si le scénario les utilise.

### Relations

- Créer les entités référencées en premier
- Utiliser des références nommées de manière explicite
- Couvrir les cas : relation présente, relation nulle (si nullable), relation multiple
- Pour DoctrineFixturesBundle, utiliser `DependentFixtureInterface` si une fixture dépend de références créées ailleurs.
- Pour Alice, utiliser les références `@reference_name` et des noms lisibles.
- Pour ManyToOne obligatoire, créer la cible avant la source.
- Pour OneToMany bidirectionnel, vérifier que la méthode d'ajout maintient les deux côtés de la relation.
- Pour ManyToMany, créer au moins un cas vide si autorisé et un cas avec plusieurs relations.
- Ne pas compter sur une cascade persist absente : persister explicitement les entités nécessaires.

### Contraintes de validation

- Chaque fixture doit respecter les contraintes `#[Assert\...]` de l'entité
- Respecter les contraintes DB : `NOT NULL`, unique, longueur, enum, clé étrangère
- Créer des fixtures invalides uniquement dans un groupe, fichier ou test séparé
- Pour les contraintes uniques, générer des valeurs explicitement distinctes
- Pour les mots de passe, utiliser le hasher du projet ou un hash de test connu, jamais un mot de passe réel
- Pour les fichiers, utiliser des fichiers de test dédiés et non des chemins absolus locaux

### Sécurité et environnements

- Ne jamais cibler ou documenter un chargement en production.
- Ne pas lire `.env.local`.
- Ne pas inclure de secrets, tokens, cookies, credentials ou données personnelles réelles.
- Ne pas charger des fixtures destructives sans confirmation explicite.
- Pour un chargement local ou test, préciser l'environnement attendu (`--env=test` si pertinent).

### Performance

- Éviter un `flush()` après chaque entité sauf besoin explicite.
- Utiliser un flush final pour les petits jeux de données.
- Pour un volume important, flush et clear par batch.
- Éviter les appels réseau, I/O externes ou services lents dans les fixtures.
- Garder les fixtures par défaut rapides à charger.

## Format Alice (YAML)

```yaml
App\Entity\User:
    user_admin:
        email: 'admin@example.com'
        firstName: 'Marie'
        lastName: 'Dupont'
        roles: ['ROLE_ADMIN']
    user_{1..5}:
        email: '<email()>'
        firstName: '<firstNameFemale()>'
        lastName: '<lastName()>'
        roles: ['ROLE_USER']
```

Règles Alice :

- Utiliser des références explicites pour les entités importantes.
- Garder les expressions Faker lisibles et compatibles avec la version utilisée.
- Éviter l'aléatoire non borné pour les champs validés ou uniques.
- Séparer les fichiers par domaine ou scénario si le projet le fait déjà.

## Format DoctrineFixturesBundle (PHP)

```php
public function load(ObjectManager $manager): void
{
    $user = new User();
    $user->setEmail('admin@example.com');
    $user->setFirstName('Marie');
    $user->setLastName('Dupont');
    $manager->persist($user);

    $this->addReference('user_admin', $user);
    $manager->flush();
}
```

Règles DoctrineFixturesBundle :

- Implémenter `DependentFixtureInterface` pour les dépendances entre fixtures.
- Utiliser `addReference()` et `getReference()` avec des noms stables.
- Utiliser le password hasher si des utilisateurs authentifiables sont créés.
- Éviter d'appeler des services métier complexes depuis les fixtures sauf convention projet claire.
- Pour des fixtures groupées, respecter `FixtureGroupInterface` si le projet l'utilise.

## Format Foundry

Si Foundry est déjà présent :

- Utiliser les factories existantes.
- Créer ou modifier une `Story` pour les scénarios réutilisables.
- Garder les states lisibles : `UserFactory::new()->admin()`, `OrderFactory::new()->paid()`.
- Préférer les overrides explicites pour les assertions importantes.
- Ne pas introduire Foundry dans un projet qui ne l'utilise pas sans demande explicite.

## Commandes utiles

Adapter les commandes au projet :

- `bin/console doctrine:fixtures:load --env=test`
- `bin/console doctrine:fixtures:load --group=nom --env=test`
- `bin/console doctrine:schema:validate`
- `vendor/bin/phpunit --filter NomDuTest`
- Commandes Hautelook Alice ou Foundry si elles existent dans le projet

Ne pas lancer de commande destructive sans confirmation explicite, notamment un chargement de fixtures qui purge la base.

## Format de sortie

Pour une génération ou modification, fournir :

- Fichiers créés ou modifiés
- Format retenu et raison
- Scénarios ou entités couverts
- Références créées et relations importantes
- Contraintes de validation ou DB prises en compte
- Commandes de validation lancées ou non lancées
- Limites ou hypothèses restantes

Générer le fichier de fixtures complet, prêt à utiliser, avec les imports, références, dépendances et groupes nécessaires pour les relations.
