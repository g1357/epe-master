# E+E Master

Новый корпоративный и экспертный сайт компании E+E Master (`epe-m.ru`) на
Drupal 11 и Varbase 11.

Репозиторий: <https://github.com/g1357/epe-master>

## Статус

Исследовательская фаза Varbase 11 завершена: выполнены восемь практических
исследований Canvas, structured content, multilingual, search, access/workflow,
SEO/forms, API/AI/Recipes и production readiness.

E+E Master v1 Requirements & Architecture Review завершён на уровне
рекомендаций; значимые owner-dependent решения перечислены отдельно и ещё не
считаются принятыми. Реализация E+E Master v1 **не начата**: production content
types, theme, permissions, search/API и runtime configuration не создавались.

Текущий baseline:

- Varbase project template 11.0.6;
- Varbase 11.0.0-rc1;
- Drupal 11.4.5;
- PHP 8.4;
- Drush 13.7.6;
- DDEV 1.25.3;
- MariaDB 11.8;
- Composer 2;
- Node.js 20 и Yarn 4 (штатный front-end toolchain Varbase).

Точные версии PHP, Composer, Node.js и Yarn следует сверять командами из
[руководства по локальной разработке](docs/development/local-environment.md),
поскольку часть инструментов поставляется DDEV-контейнером.

## Быстрый старт

Требования: WSL2, Docker CE/Docker Desktop и DDEV.

```bash
git clone https://github.com/g1357/epe-master.git
cd epe-master
ddev start
ddev install-varbase full
```

После уже выполненной установки достаточно `ddev start`. Локальный адрес:
<http://ee-master.ddev.site>.

## Принцип разработки

> Platform first, custom last.

Сначала используются штатные возможности Varbase 11, Drupal core и уже
входящие зрелые contrib-модули. Дополнительный contrib или custom-код допустим
только после документированного подтверждения ограничения платформы.

## Документация

- [Архитектура](docs/architecture/README.md)
- [E+E Master v1 architecture](docs/architecture/epe-master-v1-architecture.md)
- [ADR backlog](docs/architecture/adr-backlog.md)
- [ADR-0001: Varbase 11 и platform first](docs/adr/ADR-0001-varbase-11-platform-first.md)
- [Локальная разработка](docs/development/local-environment.md)
- [Принципы и требования](docs/requirements/project-principles.md)
- [E+E Master v1 requirements](docs/requirements/epe-master-v1-requirements.md)
- [Future backlog](docs/requirements/epe-master-future-backlog.md)
- [Карта возможностей Varbase](docs/research/varbase-capability-map.md)
- [План исследования](docs/research/plan.md)

## Структура

```text
.ddev/          Штатная DDEV-конфигурация и команды Varbase
.github/        Шаблоны GitHub
docs/           Архитектура, ADR, требования и исследования
recipes/        Composer-managed Drupal/Varbase recipes
web/            Drupal document root
composer.json   Корневые PHP-зависимости
composer.lock   Зафиксированные PHP-версии
```

`vendor`, contrib-код, runtime-файлы и секреты не хранятся в Git и
восстанавливаются штатными инструментами.
