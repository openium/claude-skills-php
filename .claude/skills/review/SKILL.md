---
name: review
description: "Revue de code PHP/Symfony pour diff git, fichiers modifiés, branche ou périmètre donné. À utiliser pour une review, relecture, audit de PR ou vérification avant commit. Priorise bugs, sécurité, architecture, régressions, tests manquants, Doctrine et migrations."
---

# Revue de code PHP/Symfony

## Périmètre

Si l'utilisateur précise un fichier, dossier, commit, branche ou périmètre, analyser uniquement celui-ci.

Sinon :

- Analyser le diff staged (`git diff --cached`) s'il existe.
- À défaut, analyser le diff non staged (`git diff`).
- Si aucun diff n'existe, demander le périmètre à reviewer.

## Processus

1. Lire `git status` et un résumé du diff (`git diff --stat` ou équivalent).
2. Lire le diff détaillé des fichiers concernés.
3. Inspecter les fichiers voisins uniquement si nécessaire pour vérifier un risque.
4. Identifier les conventions du projet avant de signaler une violation.
5. Remonter seulement les problèmes prouvés par le code, la configuration ou le diff.

Ne pas modifier le code pendant une review, sauf demande explicite.

## Sévérités

- **Bloquant** : bug avéré, faille de sécurité, perte de données, régression, BC break non maîtrisé, violation d'architecture certaine.
- **Important** : risque réel et plausible, test manquant sur comportement critique, dette qui bloque une évolution proche.
- **Suggestion** : lisibilité, convention, simplification ou amélioration sans risque immédiat.

## Points d'analyse

### Bugs et régressions

- Changement de comportement non couvert ou non annoncé
- Cas limite cassé : null, chaîne vide, collection vide, valeur extrême
- Exception avalée ou remplacée par une erreur silencieuse
- Changement de signature, type de retour, payload API ou exception publique
- Valeur par défaut dangereuse ou incompatible avec l'existant

### Architecture

- Logique métier dans un contrôleur, une commande console, ou un listener
- Dépendance du Domain vers l'Infrastructure ou vers le Framework
- Import Symfony dans une classe du Domain (Entity, ValueObject, Service métier)
- Repository qui contient de la logique métier au lieu de requêtes
- Service applicatif qui accède directement à `$_GET`, `$_POST`, `$_SESSION` ou `Request`
- Abstraction ajoutée sans besoin réel ou incompatible avec les conventions du projet

### Sécurité

- Requêtes DQL/SQL construites par concaténation au lieu de paramètres bindés
- Données utilisateur affichées dans Twig sans échappement (`|raw` sur des données non fiables)
- Route sensible sans contrôle d'accès (`#[IsGranted]`, `denyAccessUnlessGranted`, voter, `access_control`)
- Tokens, mots de passe, clés API en dur dans le code
- Désérialisation de données utilisateur sans validation
- Upload de fichier sans vérification du type MIME réel
- Commande shell construite avec une donnée utilisateur sans échappement
- Log contenant token, mot de passe, header Authorization ou donnée personnelle sensible

### Tests

- Comportement métier ajouté sans test correspondant
- Régression possible sans test de non-régression
- Test fonctionnel qui mocke la base de données au lieu de la tester réellement
- Absence de data provider quand plusieurs cas similaires sont testés
- Test qui ne vérifie pas réellement le comportement modifié

### Doctrine et migrations

- Entité modifiée sans migration correspondante
- Migration destructive sans garde-fou ou sans valeur par défaut sûre
- Relation sans cascade appropriée
- Requête N+1 détectable dans le diff
- Absence d'index sur une colonne ajoutée et utilisée dans un WHERE, JOIN ou ORDER BY
- Flush dans une boucle au lieu d'un flush unique
- Plusieurs écritures cohérentes sans transaction explicite

### API, DTO et validation

- Endpoint qui accepte une donnée utilisateur sans validation serveur
- Hydratation directe d'une entité Doctrine depuis une requête externe
- Serializer avec groupes trop larges ou absents sur une réponse sensible
- DTO d'input absent pour une opération complexe ou sensible
- Code HTTP, format d'erreur ou contrat de réponse incompatible avec l'existant

### Messenger et async

- Handler non idempotent alors que le message peut être rejoué
- Retry dangereux pour une opération non répétable
- Message contenant trop de données au lieu d'identifiants stables
- Absence de log ou de gestion d'échec sur un traitement critique

### Performance

- Requête en boucle, pagination absente ou chargement excessif
- Lazy loading déclenché dans une boucle
- Appel réseau ou I/O bloquant dans une boucle sans batch
- Serializer qui expose ou charge plus de données que nécessaire

### Conventions

- Violations PSR-12 (nommage, espacement, structure)
- Nommage incohérent avec le reste du projet
- Méthode de plus de 30 lignes sans extraction
- Classe de plus de 300 lignes
- Ligne de plus de 100 caractères
- 2 lignes vides ou plus consécutives
- Injection par constructeur manquante (utilisation de `new` pour un service)

## Ne pas signaler

- Une absence de test si le diff ne change pas de comportement.
- Une convention si le projet suit clairement une convention différente.
- Une amélioration stylistique sans impact quand des problèmes plus importants existent.
- Un risque spéculatif sans chemin d'exécution crédible.
- Le même problème répété ligne par ligne : grouper la remarque.

## Format de sortie

Présenter les findings en premier, triés par sévérité décroissante.

Pour chaque finding :

- Fichier et ligne
- Sévérité : bloquant, important ou suggestion
- Preuve vérifiée dans le code ou le diff
- Impact concret
- Correction proposée
- Test ou vérification conseillé

Si aucun problème n'est trouvé, le dire clairement et mentionner les éventuels tests non exécutés ou limites de la revue.
