# BudgetBuddy

A small Flutter app for tracking expenses and income by category. Everything is stored
on the device; there is no account, backend or network access.

> This repository was formerly published under the **Tylefnx** GitHub account.

## Features

- **Expenses and income modes.** The main screen shows the categories for the current
  mode as a circular wheel. Its centre shows the mode's total, and tapping the centre
  switches between expenses and income.
- **Categories.** Use the **+** button on the wheel to add a category: give it a name
  and pick one of a set of Material icons. Long-press a category to edit its name and
  icon, or to delete it along with its log entries.
- **Recording amounts.** Tap a category to enter an amount. You can:
  - **Add** the amount to the category total. This writes a log entry
    ("Spent: …" or "Earned: …").
  - **Change** the category total to the new value. This writes an "Updated: …"
    entry and clears the category's earlier log entries.
- **Balance.** Below the wheel, the app shows your total income minus your total
  expenses: green when positive, red when negative.
- **Activity log.** The bottom half of the screen lists the log entries for the
  current mode.
- **Local persistence.** Categories and logs are saved in two
  [sembast](https://pub.dev/packages/sembast) databases (`categories.db` and `logs.db`)
  in the app's documents directory.
- Dark theme. Amounts are shown with a `$` sign.

## Tech stack

| Area | Choice |
|---|---|
| Framework | Flutter, Dart SDK `^3.5.3` |
| State management | `hooks_riverpod` (`StateNotifier` + `StateNotifierProvider`), `flutter_hooks` |
| Models / state unions | `freezed`, `json_serializable` |
| Persistence | `sembast` (via `sembast_io`), `path_provider` |
| Error handling | `dartz` `Either<Failure, T>` |
| Routing | `auto_route` |
| UI | Material, `circle_list` for the category wheel |
| Lints | `flutter_lints` |

## Architecture

The code is organised by layer inside each model:

```
lib/
├── main.dart              # opens both sembast databases, overrides providers, runs the app
├── core/
│   ├── presentation/      # AppWidget (MaterialApp.router), shared dialog/text field widgets,
│   │                      # database initialisation
│   └── shared/            # database providers (overridden in main.dart)
├── router/                # auto_route configuration (single MainPage route)
├── main_screen/           # main page, category wheel, logs list
├── dialogs/               # create/edit/delete category, add amount, icon picker
└── models/
    ├── categories/
    │   ├── domain/          # Category (freezed + JSON), CategoryFailure
    │   ├── infrastructure/  # CategoriesServices (sembast store) and CategoriesRepository
    │   ├── application/     # CategoriesStateNotifier + freezed CategoriesState
    │   └── shared/          # Riverpod providers
    └── logs/              # same layout for LogWithDate entries
```

How data moves through the app:

- **Services** read and write a sembast store (`categories_store` or `log_store`).
- **Repositories** wrap the service calls. They return `Either<Failure, T>` and turn a
  `FileSystemException` into a `permission` failure.
- **State notifiers** expose a freezed union (`initial` / `reading` / `writing` /
  `done` / `failed`) that the widgets watch through Riverpod.

## Getting started

Prerequisite: a Flutter SDK that supports Dart `^3.5.3`.

```bash
flutter pub get
flutter run
```

The generated files (`*.freezed.dart`, `*.g.dart`, `app_router.gr.dart`) are committed,
so you don't need to run code generation to build the app. If you change an annotated
model or the router, regenerate them with:

```bash
dart run build_runner build --delete-conflicting-outputs
```

**Platforms:** the repo has platform folders for Android, iOS, Linux, macOS, Windows
and web. Storage uses `sembast_io` and `path_provider`, which need `dart:io` file
access, so the web target won't work as written.

## License

GPL-3.0. See [LICENSE](LICENSE).
