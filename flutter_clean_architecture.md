# Flutter Clean Architecture — Structure Complète

> Guide de référence complet : structure des dossiers, rôle de chaque fichier et règles strictes à respecter.

---

## Structure des dossiers

```
lib/
│
├── core/
│   ├── constants/
│   │   ├── app_colors.dart
│   │   ├── app_strings.dart
│   │   └── app_dimensions.dart
│   │
│   ├── errors/
│   │   ├── failures.dart
│   │   └── exceptions.dart
│   │
│   ├── network/
│   │   ├── api_client.dart
│   │   └── network_info.dart
│   │
│   ├── usecases/
│   │   └── usecase.dart
│   │
│   ├── utils/
│   │   ├── date_utils.dart
│   │   ├── string_utils.dart
│   │   └── validators.dart
│   │
│   └── widgets/
│       ├── app_button.dart
│       ├── app_text_field.dart
│       └── loading_widget.dart
│
├── features/
│   └── auth/
│       ├── data/
│       │   ├── datasources/
│       │   │   ├── auth_remote_datasource.dart
│       │   │   └── auth_local_datasource.dart
│       │   ├── models/
│       │   │   └── user_model.dart
│       │   └── repositories/
│       │       └── auth_repository_impl.dart
│       │
│       ├── domain/
│       │   ├── entities/
│       │   │   └── user.dart
│       │   ├── repositories/
│       │   │   └── auth_repository.dart
│       │   └── usecases/
│       │       ├── login_usecase.dart
│       │       └── logout_usecase.dart
│       │
│       └── presentation/
│           ├── bloc/
│           │   ├── auth_bloc.dart
│           │   ├── auth_event.dart
│           │   └── auth_state.dart
│           ├── pages/
│           │   ├── login_page.dart
│           │   └── register_page.dart
│           └── widgets/
│               └── login_form.dart
│
├── config/
│   ├── routes/
│   │   ├── app_router.dart
│   │   └── route_names.dart
│   └── theme/
│       ├── app_theme.dart
│       └── app_text_styles.dart
│
├── injection/
│   └── injection_container.dart
│
└── main.dart
```

---

## Rôle de chaque dossier et fichier

### `core/` — Le cerveau partagé de l'app

> Tout ce qui est utilisé par **plusieurs features** se met ici. On n'y met jamais de logique spécifique à une feature.

#### `core/constants/`

| Fichier | Rôle |
|---|---|
| `app_colors.dart` | Toutes les couleurs de l'app (primary, background, error…) |
| `app_strings.dart` | Tous les textes statiques (labels, messages d'erreur, titres) |
| `app_dimensions.dart` | Espacements, tailles, border radius réutilisables |

#### `core/errors/`

| Fichier | Rôle |
|---|---|
| `failures.dart` | Erreurs côté **métier** (ServerFailure, CacheFailure…). C'est ce que le domain retourne en cas de problème via `Either`. |
| `exceptions.dart` | Erreurs côté **technique** (NetworkException, CacheException…). C'est ce que la couche data lance avec `throw`. |

> **Règle :** `exceptions` → lancées dans `data/`, converties en `failures` dans le repository, remontées au domain via `Either`.

#### `core/network/`

| Fichier | Rôle |
|---|---|
| `api_client.dart` | Configuration de Dio : baseUrl, headers par défaut, intercepteurs (token, logs) |
| `network_info.dart` | Vérifie si le téléphone est connecté à internet avant un appel API |

#### `core/usecases/`

| Fichier | Rôle |
|---|---|
| `usecase.dart` | Interface abstraite que **tous** les usecases doivent implémenter. Définit la signature : `call(Params) → Future<Either<Failure, T>>` |

#### `core/utils/`

| Fichier | Rôle |
|---|---|
| `date_utils.dart` | Fonctions utilitaires pour formater et parser les dates |
| `string_utils.dart` | Fonctions utilitaires sur les chaînes (capitalize, truncate, slugify…) |
| `validators.dart` | Validation des formulaires (email valide, mot de passe fort, champ requis…) |

#### `core/widgets/`

| Fichier | Rôle |
|---|---|
| `app_button.dart` | Bouton personnalisé réutilisable dans toute l'app |
| `app_text_field.dart` | Champ de saisie personnalisé réutilisable |
| `loading_widget.dart` | Spinner / indicateur de chargement global |

---

### `features/` — Le cœur de l'application

> Chaque feature est un **module totalement indépendant** avec ses 3 couches. L'exemple ci-dessous utilise `auth/` mais la structure est identique pour toutes les features.

---

#### Couche `data/` — Parle au monde extérieur

##### `data/datasources/`

| Fichier | Rôle |
|---|---|
| `auth_remote_datasource.dart` | Appels HTTP vers l'API (login, register, refresh token…). Lance des `exceptions` en cas d'erreur. |
| `auth_local_datasource.dart` | Lecture/écriture locale : token dans SharedPreferences, cache utilisateur… |

##### `data/models/`

| Fichier | Rôle |
|---|---|
| `user_model.dart` | `UserModel` = `User` (entity) + `fromJson()` + `toJson()`. Sert à convertir le JSON de l'API en objet Dart et inversement. |

##### `data/repositories/`

| Fichier | Rôle |
|---|---|
| `auth_repository_impl.dart` | Implémentation concrète du repository. Décide : prendre les données du cache ou de l'API ? Convertit les `exceptions` en `failures`. Implémente l'interface définie dans le domain. |

---

#### Couche `domain/` — Règles métier pures

> **Aucune dépendance externe.** Cette couche ne connaît ni Flutter, ni Dio, ni SharedPreferences.

##### `domain/entities/`

| Fichier | Rôle |
|---|---|
| `user.dart` | Objet métier pur (id, email, name). Pas de JSON ici — juste les données importantes pour l'app. Étend `Equatable`. |

##### `domain/repositories/`

| Fichier | Rôle |
|---|---|
| `auth_repository.dart` | **Contrat (interface abstraite)** du repository. Définit QUOI faire, pas COMMENT. Ex : `Future<Either<Failure, User>> login(String email, String password)` |

##### `domain/usecases/`

| Fichier | Rôle |
|---|---|
| `login_usecase.dart` | **Une seule responsabilité :** exécuter la connexion. Appelle `authRepository.login()` et retourne le résultat. |
| `logout_usecase.dart` | **Une seule responsabilité :** exécuter la déconnexion. |

> **Règle :** 1 action métier = 1 usecase séparé.

---

#### Couche `presentation/` — Ce que l'utilisateur voit

##### `presentation/bloc/`

| Fichier | Rôle |
|---|---|
| `auth_bloc.dart` | Reçoit les events, appelle les usecases, émet les states. C'est le chef d'orchestre de l'UI. |
| `auth_event.dart` | Les **actions** utilisateur (LoginPressed, LogoutPressed, RegisterPressed…) |
| `auth_state.dart` | Les **états** possibles de l'écran (AuthInitial, AuthLoading, AuthSuccess, AuthError) |

##### `presentation/pages/`

| Fichier | Rôle |
|---|---|
| `login_page.dart` | Écran complet de connexion. S'abonne au BLoC et réagit aux states. |
| `register_page.dart` | Écran complet d'inscription. |

##### `presentation/widgets/`

| Fichier | Rôle |
|---|---|
| `login_form.dart` | Formulaire extrait de `login_page.dart` pour alléger le code. Widget **spécifique** à cette feature uniquement. |

---

### `config/` — Configuration globale de l'app

#### `config/routes/`

| Fichier | Rôle |
|---|---|
| `app_router.dart` | Définition de toutes les routes avec GoRouter. C'est le GPS de ton application. |
| `route_names.dart` | Constantes des noms de routes (`'/login'`, `'/home'`…). Évite les fautes de frappe dans les navigations. |

#### `config/theme/`

| Fichier | Rôle |
|---|---|
| `app_theme.dart` | `ThemeData` complet : couleurs, boutons, inputs, appBar, cards… |
| `app_text_styles.dart` | Tous les styles de texte (titre, sous-titre, body, caption…) |

---

### `injection/`

| Fichier | Rôle |
|---|---|
| `injection_container.dart` | Enregistre **toutes** les dépendances avec GetIt. C'est ici qu'on dit : "quand on demande `AuthRepository`, donne `AuthRepositoryImpl`". Se lit de bas en haut : External → Data → Domain → Presentation. |

---

### `main.dart`

Point d'entrée de l'app. Lance l'initialisation de GetIt, configure les providers BLoC globaux et démarre `MyApp`. C'est le **seul** endroit où on appelle `init()` de l'injection.

---

## Schéma du flux de données

```
[Page]  →  envoie un Event
   ↓
[BLoC]  →  appelle le UseCase
   ↓
[UseCase]  →  appelle le Repository (interface)
   ↓
[Repository Impl]  →  choisit DataSource (remote ou local)
   ↓
[DataSource]  →  appelle l'API ou la base locale
   ↓
   Le résultat remonte en sens inverse jusqu'à la Page
```

---

## Les règles strictes à respecter

| # | Règle | ❌ Interdit |
|---|---|---|
| 1 | **Domain ne dépend de rien** | Jamais importer `data/` ou `presentation/` dans `domain/` |
| 2 | **Toujours passer par un UseCase** | La page ne doit jamais appeler directement le repository |
| 3 | **Model ≠ Entity** | Ne pas utiliser `UserModel` dans le domain — seulement `User` |
| 4 | **Un BLoC par feature** | Pas de BLoC global sauf pour l'auth et le thème |
| 5 | **Repository = interface dans domain** | L'implémentation est toujours dans `data/` |
| 6 | **Injection via GetIt** | Pas de `new MaClasse()` dans les widgets |
| 7 | **Either pour les erreurs** | Jamais de `try/catch` dans le domain |
| 8 | **Feature = module isolé** | Une feature ne doit pas importer une autre feature directement |

---

## Règle d'or — Où mettre un fichier ?

| Question | Réponse |
|---|---|
| C'est utilisé dans 2+ features ? | → `core/` |
| C'est lié à l'affichage ? | → `presentation/` |
| C'est une règle métier ? | → `domain/` |
| Ça parle à une API ou BDD ? | → `data/` |
| C'est un objet sans JSON ? | → `entities/` |
| C'est un objet avec JSON ? | → `models/` |
| C'est une action utilisateur ? | → `usecases/` |

---

## Packages recommandés

```yaml
dependencies:
  flutter_bloc: ^8.1.6           # Gestion d'état
  get_it: ^8.0.0                 # Injection de dépendances
  dartz: ^0.10.1                 # Either (gestion erreurs fonctionnelle)
  equatable: ^2.0.5              # Comparaison d'objets
  dio: ^5.7.0                    # Appels HTTP
  go_router: ^14.0.0             # Navigation
  shared_preferences: ^2.3.0     # Stockage local simple
  flutter_secure_storage: ^9.2.2 # Stockage sécurisé (tokens JWT)

dev_dependencies:
  bloc_test: ^9.1.7              # Tests BLoC
  mocktail: ^1.0.4               # Mocking pour les tests
```

---

*Document généré pour Flutter Clean Architecture — Dernière mise à jour : Mars 2026*
