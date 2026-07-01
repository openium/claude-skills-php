---
name: test
description: "Génère des tests PHPUnit pour un fichier ou une classe PHP/Symfony. Couvre le cas nominal, les cas limites et les cas d'erreur. Utilise par défaut la structure Unit/Functional/Integration, s'adapte aux projets existants en miroir src/ vers tests/, et centralise les mocks/stubs dans tests/Mock."
---

# Génération de tests PHPUnit

## Processus

1. Lire le fichier cible pour comprendre la logique
2. Identifier le type de test adapté (unitaire ou fonctionnel)
3. Analyser les conventions du projet (structure tests/, nommage, base classes)
4. Générer le test complet

## Conventions à détecter

Avant de générer, inspecter le dossier `tests/` pour identifier :

- Structure actuelle : `tests/Unit/`, `tests/Functional/`, `tests/Integration/`, miroir de `src/`, ou autre convention existante
- Base class utilisée : `TestCase`, `KernelTestCase`, `WebTestCase`, custom
- Nommage des méthodes : `testCamelCase()`, ou attribut `#[Test]`
- Utilisation de data providers : `#[DataProvider('providerName')]`
- Setup/Teardown patterns du projet
- Présence de `tests/Mock/` et configuration de services de test dans `config/services.yaml` ou `config/services_test.yaml`

## Structure des tests

### Convention par défaut

Si le projet n'a pas encore de convention claire, utiliser une structure explicite par type de test :

Exemples :

- `src/Service/AcmeService.php` -> `tests/Unit/Service/AcmeServiceTest.php`
- `src/Controller/UserController.php` -> `tests/Functional/Controller/UserControllerTest.php`
- `src/Repository/UserRepository.php` -> `tests/Integration/Repository/UserRepositoryTest.php`

Le namespace suit la structure des tests, par exemple `App\Tests\Unit\Service` pour `tests/Unit/Service/AcmeServiceTest.php`.

### Adaptation à l'existant

- Si le projet utilise déjà une structure miroir de `src/` dans `tests/`, respecter cette architecture.
- Exemple : `src/Service/AcmeService.php` -> `tests/Service/AcmeServiceTest.php`.
- Si plusieurs conventions coexistent, suivre celle utilisée par les tests les plus proches de la classe cible.
- Ne pas déplacer ou renommer les tests existants sans demande explicite.
- Pour un nouveau projet ou un projet sans tests, appliquer la convention `tests/Unit/`, `tests/Functional/`, `tests/Integration/`.

## Règles

### Tests unitaires (classes Domain, ValueObjects, Services sans dépendance externe)

- Placer dans `tests/Unit/` en miroir de la structure `src/`, sauf convention existante différente
- Aucune dépendance au kernel Symfony
- Mocker les interfaces, pas les classes concrètes
- Couvrir :
  - Le cas nominal (happy path)
  - Au moins un cas limite (null, vide, valeur extrême)
  - Au moins un cas d'erreur (exception attendue)

### Tests fonctionnels (contrôleurs, commandes, intégration)

- Placer dans `tests/Functional/` pour les contrôleurs et commandes, ou `tests/Integration/` pour les tests avec kernel, base de données ou services réels, sauf convention existante différente
- Ne jamais mocker la base de données
- Utiliser les fixtures du projet si elles existent
- Tester les codes de réponse HTTP, le contenu, les redirections

### Mocks et stubs

- Placer les mocks, stubs et fakes réutilisables dans `tests/Mock/`.
- Reproduire dans `tests/Mock/` la même structure que dans `src/`.
- Exemple : mock pour `src/Client/AcmeClient.php` -> `tests/Mock/Client/AcmeClientMock.php`.
- Préférer un mock dédié dans `tests/Mock/` quand il est partagé par plusieurs tests.
- Garder les mocks locaux au test quand ils sont simples et utilisés une seule fois.
- Les mocks de services Symfony doivent être enregistrés uniquement en environnement de test.

Configuration attendue dans `config/services.yaml` si le projet n'a pas déjà une configuration équivalente :

```yaml
when@test:
    services:
        App\Tests\Mock\:
            resource: '../tests/Mock/'
            autowire: true
            autoconfigure: true
```

Adapter le namespace si la configuration du projet utilise un namespace de tests différent.

### Data providers

Utiliser un data provider quand :
- Plus de 2 cas testent la même méthode avec des entrées différentes
- Les cas limites sont nombreux (validation, parsing, conversion)

Format :
```php
#[DataProvider('exampleProvider')]
public function test_example(string $input, string $expected): void
{
    // ...
}

public static function exampleProvider(): iterable
{
    yield 'cas nominal' => ['input', 'expected'];
    yield 'cas limite' => ['', ''];
}
```

### Assertions

- Préférer les assertions spécifiques (`assertSame` > `assertEquals` > `assertTrue`)
- Un test = un comportement. Pas 15 assertions dans un seul test.
- Nommer les méthodes de test de manière descriptive en camelCase : `testCreateUserWithInvalidEmailThrowsException`
- Déclarer systématiquement `void` comme type de retour des méthodes de test

## Format de sortie

Générer le fichier de test complet, prêt à exécuter, avec imports, classe et méthodes de test. Si un mock partagé est nécessaire, générer aussi le fichier dans `tests/Mock/` et l'ajustement `when@test` de configuration si absent.
