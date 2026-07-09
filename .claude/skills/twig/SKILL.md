---
name: twig
description: "Audit et amélioration de templates Twig Symfony. Vérifie sécurité, accessibilité, performance, formulaires, architecture, i18n, UX, emails et conventions projet, en tenant compte des versions Symfony/Twig et des patterns existants."
---

# Audit de templates Twig

## Périmètre

Si l'utilisateur précise un template, dossier, diff, formulaire, layout, composant, email Twig, page back-office ou rendu HTML, analyser uniquement ce périmètre.

Sinon :

- Analyser les templates modifiés (`git diff`) s'il y en a.
- À défaut, demander le template ou le dossier à auditer.

Ne pas modifier les templates pendant un audit sauf demande explicite.

## État des lieux

Inspecter selon le périmètre :

- Template ciblé, layouts parents, partials, includes, macros ou components utilisés
- Contrôleur, form type, DTO ou view model qui fournit les variables
- Routes utilisées par `path()` / `url()`
- Traductions et domaines i18n si présents
- Assets, importmap, Webpack Encore, AssetMapper ou pipeline existant
- Design system, classes CSS, composants UI ou conventions du projet
- Tests fonctionnels qui couvrent la page si présents

Ne jamais lire `.env.local`.

## Compatibilité Symfony/Twig

- Ne pas supposer Twig Components, UX, Stimulus, AssetMapper ou importmap si le projet ne les utilise pas.
- Sur projet legacy, respecter macros, partials, includes et conventions existantes.
- Sur projet récent, utiliser Twig Components seulement si déjà installés et utilisés.
- Suivre le style de routing, formulaires, traductions et assets du projet.
- Ne pas introduire une nouvelle organisation de templates sans bénéfice clair.

## Sévérités

- **Bloquant** : XSS, CSRF absent sur action mutante, fuite de donnée sensible, URL dangereuse, include dynamique non fiable.
- **Important** : accessibilité cassée, N+1 probable, formulaire incomplet, logique métier lourde, duplication qui rend la page fragile.
- **Suggestion** : convention, lisibilité, extraction de partial, i18n, amélioration UX non critique.

## Critères d'analyse

### Sécurité (bloquant)

- `|raw` sur des données utilisateur sans sanitization
- Variables dans des attributs HTML sans échappement contextuel
- URLs construites à partir de données utilisateur (`href="{{ user_url }}"`)
- Formulaires sans token CSRF (`{{ csrf_token('intention') }}` ou `form_rest()`)
- Inclusion de template dynamique (`include(variable)`) avec des données non fiables
- Données utilisateur injectées dans JavaScript inline sans encodage JSON adapté
- `href`, `src`, `action` ou `formaction` alimentés par une valeur externe non validée
- Données sensibles affichées : token, email privé, rôle interne, identifiant technique, information personnelle non nécessaire
- Template d'email qui expose des données sensibles ou liens non expirables
- Logique d'autorisation uniquement dans Twig sans contrôle côté contrôleur/voter

### Performance (important)

- Requêtes Doctrine déclenchées dans le template (lazy loading via des getters de relation)
- Boucles sur des collections non paginées
- Filtres coûteux dans des boucles (`|sort`, `|filter`, `|map` sur de grandes collections)
- Assets non versionnés (cache busting absent)
- Blocs répétitifs qui pourraient être cachés avec `{% cache %}`
- `|length` sur une collection Doctrine lazy non initialisée
- Includes, embeds, components ou macros coûteux appelés dans une grande boucle
- Calcul ou formatage répété qui devrait être préparé dans le contrôleur/view model
- Images trop lourdes ou sans dimensions si visible dans le template
- Payload HTML excessif pour une liste non paginée

### Architecture (important)

- Logique métier dans le template (calculs, conditions complexes, formatage avancé). Extraire vers une Twig Extension ou un service
- Template de plus de 200 lignes sans extraction de blocs ou partials
- Duplication de HTML entre templates. Extraire dans un `_partial.html.twig` ou un Twig Component
- Template qui dépend de trop de variables implicites
- Conditions de rôles ou d'états métier dupliquées dans plusieurs templates
- Partial qui modifie fortement son comportement selon trop d'options
- Macro ou component introduit alors que le projet utilise déjà une convention différente
- Layout ou blocks incohérents avec le reste de l'application

### Accessibilité (important)

- Images sans attribut `alt`
- Formulaires sans labels associés aux inputs
- Liens sans texte descriptif
- Hiérarchie de headings cassée (h1 puis h3 sans h2)
- Éléments interactifs non accessibles au clavier
- Tableaux sans `<thead>`, `<th>`, ou `scope`
- Bouton rendu comme lien ou lien rendu comme bouton sans sémantique correcte
- Message d'erreur non associé au champ concerné
- Champ obligatoire indiqué seulement par couleur ou placeholder
- Modale, menu ou dropdown sans gestion focus/clavier si visible dans le template
- Icône seule sans texte accessible (`aria-label`, texte masqué ou title selon convention)
- Contraste ou taille de cible manifestement problématique si visible dans les classes/styles

### Formulaires Symfony (important)

- Utiliser `form_start()` et `form_end()` sauf convention projet différente
- Conserver `form_rest(form)` ou `form_end(form)` pour les champs cachés et CSRF
- Afficher les erreurs globales et erreurs de champs
- Associer labels, help et erreurs aux champs
- Ne pas afficher ni rendre modifiables les champs sensibles ou système
- Suppression protégée par CSRF et confirmation si action destructrice
- Prototypes de collections correctement échappés et documentés côté JS si utilisés
- Upload : enctype correct via `form_start`, aide utilisateur et erreurs visibles

### Bonnes pratiques Twig et Symfony

- Utiliser `{{ path('route_name') }}` au lieu de URLs en dur
- Utiliser `{{ asset('path') }}` pour les fichiers statiques
- Utiliser `|trans` pour les chaînes (i18n-ready)
- Préférer les Twig Components aux macros pour les éléments réutilisables
- `form_rest(form)` à la fin de chaque formulaire
- Nommage des blocs : explicite et cohérent
- Utiliser `url()` quand une URL absolue est nécessaire, notamment email
- Garder les conditions simples et lisibles
- Préférer une variable préparée côté contrôleur à un calcul complexe en Twig
- Respecter les domaines de traduction existants
- Utiliser les filtres de date, nombre et devise selon la locale/convention projet
- Ne pas forcer Twig Components si le projet utilise macros/partials

### Structure des templates

- Héritage : un template de base (`base.html.twig`), des layouts intermédiaires si nécessaire
- Nommage : `entity/action.html.twig` cohérent avec les routes
- Partials préfixés par `_` : `_navbar.html.twig`, `_sidebar.html.twig`
- Un template enfant doit définir seulement les blocks nécessaires
- Les partials doivent avoir une responsabilité claire
- Les includes doivent recevoir explicitement les variables nécessaires si la convention projet le permet
- Les pages CRUD doivent rester cohérentes avec le layout, les flashes, la navigation et les actions existantes

### i18n et contenu (suggestion)

- Chaînes utilisateur hardcodées alors que le projet est traduit
- Domaine de traduction incohérent
- Pluriel non géré (`transchoice` legacy ou `trans` avec ICU selon version)
- Dates, nombres, montants ou devises formatés manuellement
- Texte de bouton ou lien non explicite
- Message d'erreur ou état vide absent

### UX (suggestion)

- État vide absent sur une liste
- Flash messages non affichés ou incohérents avec le layout
- Pagination, tri ou recherche absents alors que la liste peut grossir
- Confirmation manquante sur suppression ou action irréversible
- Navigation active, breadcrumbs ou retour liste incohérents avec la convention projet
- Erreurs de formulaire difficiles à comprendre

### Emails Twig (important)

- Utiliser des URLs absolues pour les liens externes (`url()` plutôt que `path()`)
- Éviter de dépendre de JS ou CSS externe non supporté par clients email
- Prévoir texte alternatif ou version texte si le projet le fait déjà
- Vérifier données sensibles, liens expirables et contexte utilisateur
- Garder styles compatibles email si le projet utilise des styles inline
- Images avec `alt` et dimensions si nécessaires

## Commandes utiles

Adapter au projet :

- `bin/console lint:twig templates/`
- `bin/console debug:twig`
- `bin/console debug:router route_name`
- `bin/console debug:translation locale`
- Tests fonctionnels ciblés de la page
- Symfony Profiler pour requêtes Doctrine et temps de rendu

Ne pas lancer de commande destructive. Ne pas modifier les traductions ou assets sans demande explicite.

## Ne pas faire

- Ne pas ajouter `|raw` pour corriger un affichage sans sanitization prouvée.
- Ne pas mettre de logique métier complexe dans Twig.
- Ne pas masquer un problème d'autorisation avec un simple `is_granted()` côté template.
- Ne pas introduire Twig Components, Stimulus ou un design system si le projet ne les utilise pas.
- Ne pas casser la compatibilité legacy Symfony/Twig en utilisant une syntaxe non supportée.
- Ne pas supprimer `form_rest` ou les champs cachés sans vérifier CSRF et method spoofing.
- Ne pas optimiser un template en déplaçant le problème vers le contrôleur sans preuve.

## Format de sortie

Présenter les findings triés par sévérité décroissante.

Pour chaque problème :

- Fichier et ligne
- Catégorie : sécurité, performance, accessibilité, formulaire, architecture, i18n, UX, email, bonnes pratiques
- Sévérité : bloquant, important ou suggestion
- Preuve vérifiée dans le template ou le code lié
- Impact concret
- Correction proposée
- Test ou vérification conseillé

Si aucun problème n'est trouvé, le dire clairement et mentionner les limites de l'analyse : contrôleur non lu, variables non connues, profiler non disponible, tests non lancés.
