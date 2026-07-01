---
name: security
description: "Audit de sécurité rapide pour projet PHP/Symfony. Analyse les fichiers modifiés ou un périmètre donné. Détecte injections SQL, XSS, CSRF, exposition de données, permissions manquantes. Basé sur OWASP Top 10. Utiliser avant un commit ou pour auditer un périmètre."
---

# Audit de sécurité PHP/Symfony

## Périmètre

Par défaut, analyse les fichiers modifiés (`git diff`). Si l'utilisateur précise un fichier, un dossier ou un périmètre, analyser celui-ci.

## Contexte Symfony à vérifier

Avant l'audit, inspecter selon le périmètre :

- `config/packages/security.yaml` : firewalls, access control, password hashers, remember me
- `config/packages/framework.yaml` : CSRF, session, trusted proxies, trusted hosts
- `config/packages/nelmio_cors.yaml` si présent : configuration CORS
- Contrôleurs API, voters, authenticators, services applicatifs et serializers
- Templates Twig utilisés pour les emails
- `composer.json` et `composer.lock`
- `.env` et `.env.dist` si présents

Ne jamais lire ni afficher le contenu de `.env.local`.

## Critères d'analyse (OWASP Top 10)

### A01 - Broken Access Control (bloquant)

- Route sensible sans contrôle d'accès visible via `#[IsGranted]`, `denyAccessUnlessGranted()`, `access_control`, voter ou service applicatif
- Route sensible accessible sans authentification
- Vérification d'accès basée sur l'ID utilisateur passé en paramètre (IDOR)
- Voter absent pour les opérations sur des entités avec propriétaire
- `security.yaml` : firewall avec `security: false` en production
- Accès direct à une entité sans vérifier que l'utilisateur courant y a droit

### A02 - Cryptographic Failures (bloquant)

- Mot de passe stocké en clair ou hashé avec MD5/SHA1
- Token/secret en dur dans le code source
- Fichier `.env` avec des secrets committé
- Clé de chiffrement faible ou prévisible
- Communication HTTP au lieu de HTTPS pour des données sensibles

### A03 - Injection (bloquant)

- Requête DQL/SQL construite par concaténation de variables
- `$connection->executeQuery("SELECT ... WHERE id = $id")` sans binding
- Commande shell construite avec des données utilisateur sans `escapeshellarg()`
- Expression régulière construite avec des données utilisateur
- Requête LDAP sans échappement

### A04 - Insecure Design (important)

- Absence de rate limiting sur login, reset password, API publique
- Pas de validation côté serveur (uniquement côté client)
- Logique d'autorisation dans le template Twig au lieu du contrôleur/voter
- Endpoint qui retourne plus de données que nécessaire (sur-exposition)
- DTO d'input absent sur un endpoint sensible qui accepte des données utilisateur
- Serializer configuré avec des groupes trop larges ou absents
- Désérialisation qui hydrate directement une entité Doctrine exposée côté API
- Réponse API qui expose des champs sensibles via le serializer

### A05 - Security Misconfiguration (important)

- `APP_DEBUG=true` en production
- `kernel.debug` activé en prod
- Profiler Symfony accessible en production
- CORS trop permissif (`allow_origin: '*'`)
- Headers de sécurité manquants (CSP, X-Frame-Options, HSTS)

### A07 - XSS (bloquant)

- `|raw` sur des données utilisateur dans Twig
- `{{ variable|raw }}` sans sanitization préalable
- Injection dans des attributs HTML (`href="{{ user_input }}"`)
- Données utilisateur dans du JavaScript inline
- Templates d'email qui injectent des données utilisateur sans échappement ou sanitization

### A08 - Insecure Deserialization (bloquant)

- `unserialize()` sur des données utilisateur
- Désérialisation JSON sans validation de schéma avant traitement
- `Serializer::deserialize()` sans groupes de dénormalisation

### Uploads de fichiers (bloquant)

- Upload sans validation du type MIME réel
- Extension de fichier utilisée comme seule preuve du type
- Nom de fichier utilisateur conservé tel quel
- Fichier uploadé dans un dossier public exécutable
- Absence de limite de taille côté serveur

### Logs et erreurs (important)

- Logs contenant tokens, mots de passe, Authorization header ou données personnelles sensibles
- Message d'erreur API qui expose une stacktrace, une requête SQL ou une information interne
- Exception métier retournée telle quelle au client
- Données sensibles affichées dans les logs Messenger ou les logs HTTP client

### Dépendances (important)

- Vulnérabilités détectées par `composer audit`
- Package abandonné ou non maintenu utilisé sur un chemin critique
- Contraintes Composer trop larges sur une dépendance sensible
- `composer.lock` absent du repository

### Fichiers sensibles

- `.env` ou `.env.dist` avec des secrets dans le git
- `composer.lock` absent du repository (versions non verrouillées)
- Fichiers de configuration avec des credentials

## Format de sortie

Pour chaque vulnérabilité :
- Fichier et ligne
- Catégorie OWASP
- Sévérité (bloquant / important / suggestion)
- Preuve vérifiée dans le code ou la configuration
- Scénario d'exploitation réaliste
- Code vulnérable
- Code corrigé
- Test ou vérification de non-régression conseillé
- Référence (lien doc Symfony Security si applicable)
