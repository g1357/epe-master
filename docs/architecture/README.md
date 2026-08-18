# Архитектура платформы

## Текущий контекст

E+E Master создаётся как Composer-managed Drupal 11 проект на Varbase 11.
Фаза 1 предназначена для исследования платформы, а не реализации продуктовых
функций.

## Baseline

```text
GitHub repository
  └── Composer project (drupal/varbase_project)
      ├── Drupal 11 core
      ├── Varbase profile and recipes
      ├── contributed modules and themes
      └── web/ document root

DDEV
  ├── PHP 8.4 + Apache FPM
  ├── MariaDB 11.8
  ├── Composer 2 / Drush 13
  └── Node.js / Yarn / Storybook toolchain from Varbase
```

Varbase 11 использует recipe-first подход. Конфигурация распределена по
Varbase Starter и набору base recipes, а профиль установки остаётся тонким.
Исходный Composer lock-файл и `patches.lock.json` являются частью
воспроизводимого baseline.

## Среды

- Local: DDEV.
- Version control and collaboration: GitHub.
- Future production: Beget; параметры hosting stack ещё не подтверждены и
  должны быть исследованы до архитектурного решения о deployment.

## Ограничения фазы 1

- нет собственных Drupal-модулей;
- нет собственной темы;
- нет бизнес-интеграций;
- нет production-конфигурации;
- штатные full-возможности включены только для исследования и не являются
  утверждённым production-составом.

См. [ADR-0001](../adr/ADR-0001-varbase-11-platform-first.md).
