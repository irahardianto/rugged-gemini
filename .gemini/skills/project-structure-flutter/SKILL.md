---
paths:
  - "**/*.dart"
---

## Flutter/Mobile Layout

Vertical slice — features are self-contained modules.

```
apps/
  mobile/
    lib/
      core/                         # Foundation
        network/
          api_client.dart           # Dio/http with base config
          api_interceptor.dart      # Auth, logging interceptors
        storage/
          local_storage.dart        # SharedPreferences/Hive wrapper
        theme/
          app_theme.dart            # ThemeData, color schemes
          app_typography.dart       # TextStyles, font families
        router/
          app_router.dart           # GoRouter config (@riverpod)
          app_router.g.dart         # Generated
        constants/

      features/                     # Vertical slices
        task/
          screens/
            task_list_screen.dart    # Route target (ConsumerWidget)
            task_detail_screen.dart
          widgets/
            task_card.dart
            task_form.dart
          state/
            task_notifier.dart       # @riverpod class
            task_notifier.g.dart     # Generated
            task_state.dart          # freezed state classes
          models/
            task.dart               # freezed domain model
            task.g.dart             # json_serializable
            task.freezed.dart       # freezed
          logic/
            task_logic.dart         # Pure business rules
          repository/
            task_repository.dart    # Abstract interface
            task_repository_impl.dart
            task_repository_mock.dart
          api/
            task_api.dart           # REST calls (Dio)
        auth/
        settings/

      shared/                       # Cross-feature, NO domain logic
        widgets/
          app_button.dart
          app_text_field.dart
          loading_indicator.dart
        utils/
          date_formatter.dart
          validators.dart
        models/
          api_response.dart
          pagination.dart

    test/                           # Mirrors lib/ layout
      features/
        task/
          state/task_notifier_test.dart
          logic/task_logic_test.dart
          widgets/task_card_test.dart
          api/task_api_test.dart
      integration_test/
        task_flow_test.dart

    pubspec.yaml
    analysis_options.yaml           # Includes riverpod_lint
```

Key differences from web: `screens/` = views, `widgets/` = components, `state/` = Riverpod 3, `repository/` = data + cache. `ProviderScope` at root for DI (no manual locator). `*.g.dart`/`*.freezed.dart` = codegen artifacts. Tests in `test/` mirroring `lib/` (Flutter convention). Naming: `*_test.dart` (unit), `*_integration_test.dart` (integration).

### Related
- Project Structure GEMINI.md § Project Structure
- Flutter Idioms and Patterns @.gemini/skills/flutter-idioms/SKILL.md
