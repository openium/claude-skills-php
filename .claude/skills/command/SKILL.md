---
name: command
description: "Génère une commande console Symfony. Produit une commande avec arguments, options, validation des inputs, progress bar, output structuré et gestion d'erreur."
---

# Génération de commande Symfony Console

## Structure

### Attribut PHP 8+
```php
#[AsCommand(
    name: 'app:import-csv',
    description: 'Description courte',
    aliases: ['a:ic'],
)]
```

### Nommage
- Préfixe `app:`, format `app:action-name`.
- Utiliser le kebab-case pour les segments composés (`import-users`, `sync-products`, `delete-expired-sessions`).
- Garder les segments courts, explicites et en anglais.

### Description
- Utiliser une phrase courte et claire en anglais qui décrit l'action.

### Alias
- Ajouter un alias court au format `a:abc`.
- `abc` correspond aux premières lettres significatives du nom de l'action.
- Exemple : `app:import-csv` -> alias `a:ic`.
- Exemple : `app:sync-prices` -> alias `a:sp`.
- Ne pas créer d'alias ambigu si plusieurs commandes auraient le même alias.

## Règles

### Architecture
- La commande est un adaptateur mince : parse les inputs, appelle un service, affiche le résultat
- Pas de logique métier dans la commande
- Injecter les dépendances par constructeur

### Input
- Définir les arguments et options dans `configure()` avec des noms explicites
- Typer et valider chaque valeur récupérée depuis `InputInterface`
- Utiliser `InputArgument::REQUIRED` uniquement pour les valeurs indispensables
- Utiliser des options pour les comportements (`--dry-run`, `--force`, `--limit`, `--batch-size`, `--env`)
- Fournir une valeur par défaut sûre pour les options numériques
- Refuser les valeurs invalides avec un message clair et `Command::INVALID`
- Demander confirmation avec `$io->confirm()` avant une action destructive, sauf si `--force` est fourni
- Ne jamais lire directement `$_SERVER`, `$_ENV`, `$_GET`, `$_POST` dans la commande

### Questions interactives
- Utiliser les helpers de `SymfonyStyle` (`ask()`, `choice()`, `confirm()`) pour les questions simples
- Utiliser `QuestionHelper` et les classes `Question`, `ChoiceQuestion`, `ConfirmationQuestion` pour les cas avancés
- Ne poser une question que si la valeur n'est pas fournie par argument ou option
- Prévoir un mode non interactif : si `--no-interaction` est actif, utiliser les valeurs par défaut ou retourner `Command::INVALID`
- Toujours fournir une valeur par défaut explicite quand la commande peut continuer sans réponse
- Valider la réponse avec un validator de question ou une méthode dédiée
- Masquer les secrets avec `Question::setHidden(true)` et ne jamais les afficher dans l'output
- Pour les choix, utiliser `ChoiceQuestion` avec une liste fermée et refuser les réponses hors liste
- Pour les actions destructives, demander confirmation avec `confirm()` ou `ConfirmationQuestion`, sauf si `--force` est fourni

### Output
- Utiliser `SymfonyStyle`
- `$io->title()`, `$io->table()`, `$io->progressBar()`
- `$io->success()` / `$io->error()` pour le résultat final
- Adapter le détail avec les niveaux de verbosité (`-v`, `-vv`, `-vvv`)
- Afficher un résumé final : nombre d'éléments traités, ignorés, échoués
- Utiliser `$io->warning()` pour les cas non bloquants et `$io->note()` pour les informations utiles
- Pour les erreurs, afficher un message actionnable sans exposer de secret ni de donnée sensible
- Retourner le bon code : `Command::SUCCESS`, `Command::FAILURE` ou `Command::INVALID`
- Ne pas écrire directement avec `echo`, `var_dump()` ou `print_r()`
- Garder l'output stable et lisible pour une exécution en CI ou dans un cron

### Dry-run
- Supporter `--dry-run` quand la commande modifie des données
- Afficher ce qui serait fait sans persister

### Traitement par batch
Flush et clear Doctrine tous les 100 items dans les boucles.

## Format de sortie

Classe de commande complète avec imports, attribut, configuration, et méthode execute.
